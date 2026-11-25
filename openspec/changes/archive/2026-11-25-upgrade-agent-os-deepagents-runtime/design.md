# Design

## Context

对话入口在 `apps/agent-os/service/src/workspaces/conversations.service.ts`。聊天与记忆归并各自调用 `createDeepAgent`，前者还需处理工具审批、文件权限、Skills 和子 Agent。数据库初始化由仓库 DDL 管理；`DeepagentsPostgresMemoryService` 使用配置指定的 schema 并在启动时核对运行对象。DataClaw Web 负责对话交互，Agent OS Web 的会话任务详情通过 `getAdminConversation` 读取同一会话的记录。

## Goals / Non-Goals

以已验证的 2026-11-25 稳定版组成可复现的服务端依赖树，沿用现有 HTTP、消息流和管理端查询契约。此次只覆盖新版生成的数据。异步子 Agent、会话分叉及新增文件工具等可选功能不属于交付内容。

## Decisions

### 1. 在服务端解析并锁定运行时依赖

| npm 包 | 当前 | 目标 |
|---|---:|---:|
| deepagents | 1.10.2 | 1.17.1 |
| langchain | 1.4.0 | 1.6.8 |
| @langchain/core | 1.1.48 | 1.5.1 |
| @langchain/openai | 1.4.7 | 1.8.5 |
| @langchain/langgraph | 1.3.0 | 1.7.5 |
| @langchain/langgraph-checkpoint | 1.0.2 | 1.2.3 |
| @langchain/langgraph-checkpoint-postgres | 1.0.1 | 1.1.4 |
| @langchain/langgraph-sdk | 1.9.2 | 1.17.0 |
| langsmith | 0.6.3 | 0.14.0 |

单升 LangGraph 无法满足目标版本 Deep Agents 对 LangChain、core 和模型适配器的 peer 约束。更新服务直接依赖，并在服务 workspace 满足所需 peers；从实际调用方核对 SDK、LangSmith 和运行时的解析版本。保留兼容的现有 UI 流适配器，在接口回归中验证。全局 override 会影响 Desktop 的隐式导入，因此使用按服务限定的安装和共享锁文件差异检查。[目标包依赖声明](https://registry.npmjs.org/deepagents/1.17.1)

### 2. 以目标 saver/store 的完整结构初始化新库

按照部署使用的 `database/ddl/server/samatrix_auth3/` 全量 SQL、postcheck 及其来源同步更新 checkpoint/store 定义。以适配器 1.1.4 的 migrations 为准，核对列默认值、索引与库内迁移版本记录，尤其是 `checkpoint_writes.task_path`。现有仅检查表存在的路径可能在首次写入时才报错；将关键列与实际写读纳入验收。保持数据库初始化的显式入口，复核现有对 pool、isSetup 等内部成员的访问。生产进程自动建表会引入 DDL 权限和多实例竞争。[适配器变更记录](https://github.com/langchain-ai/langgraphjs/blob/main/libs/checkpoint-postgres/CHANGELOG.md)

### 3. 使用原线程恢复，按重建后的消息解释增量状态

主对话采用 Deep Agents 默认的增量消息通道。沿用已持久化的 runtimeThreadId、工具调用 ID 与审批记录；当前代码只在 scoped checkpoint 缺失时才尝试依据原始 pendingWrites 和 channel_values 匹配旧线程。新库正常路径须用同一线程恢复。需要读取完整历史时使用图运行时重建的状态；避免把缺少全量消息的增量检查点误判为缺失历史，或把另一线程当作恢复目标。批准、拒绝和重复请求通过既有业务入口验收。[增量检查点说明](https://github.com/langchain-ai/deepagentsjs/pull/1058)

### 4. 在服务边界适配新版工具和消息流

主 Agent 显式启用新版待办中间件，保留现有授权工具集合，核对同轮同路径写入冲突能作为工具失败呈现。保留 `agent.stream()` 和现有 UI 消息转换。按既有展示需求订阅子图事件，核对命名空间、工具 ID 及重复消息；无条件转发所有子图消息可能泄漏仅供内部使用的输出。DataClaw Web 在聊天页验证过程，Agent OS Web 在任务详情验证持久化后的消息与工具记录。

## Risks / Trade-offs

- 工作区配置了 `shamefully-hoist=true`。升级前后从 Desktop 的真实模块位置解析其 LangChain 导入，比较 KMS 各 workspace 的依赖闭包。出现偏移时调整服务端依赖放置，避免把适配工作扩到范围外。
- 仅凭首页或业务消息表的历史无法证明检查点可恢复。在隔离的新库中用跨多步的上下文追问、暂停审批和可计数工具验证执行状态。
- 流事件顺序和错误状态可能改变。在两端核对同一会话 ID、工具调用结果、计划进度及最终消息次数。

## Migration Plan

准备新版应用与配套新库脚本。在隔离空库预演后，按已确认的部署安排发布新版、重建目标数据库并执行正式初始化和 postcheck。用正常业务入口完成首轮会话、重启续聊与审批验收后开放流量。

## 实施确认

2026-11-25 用户确认：隔离验证可使用本地 PostgreSQL，保留生产 SeaboxSQL precheck，并明确未验证 SeaboxSQL。因根级 shameful hoist 会改变 Desktop 隐式导入，允许在 pnpm-workspace.yaml 用 packageExtensions 为 Desktop 补充 langchain 1.4.0、core 1.1.48、LangGraph 1.3.0，保持 Desktop 源文件及 manifest 与 KMS 锁定闭包不变。
