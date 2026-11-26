# Skill 与 MCP 授权设计

## Context

Agent 中心已有工具授权面板，以及可独立使用的 AgentSkillBinding.vue 和 AgentMcpBinding.vue。服务端已有 AgentSkillBinding、AgentMcpBinding 和对应管理接口。工作空间插件页同时负责资源绑定和工作区自有 Skill 的管理。聊天页会合并会话及工作空间的资源选择，运行时还会读取 Skill 快照和 MCP 偏好。

## Goals / Non-Goals

授权管理、会话首次执行与恢复执行共享同一份 Agent 绑定。沿用资源的租户、所有者、状态和运行条件。知识库和数据源的页面调整限于工作空间插件菜单及其附属交互，其底层资源服务不在此变更范围。

## Decisions

1. **按 Agent 维护绑定。** 在 AgentCenterView.vue 中直接挂载两个面板，随 selectedAgentId 切换加载。复用管理端现有 GET、PUT 绑定接口，查询沿用 `agentos:agent:view`，保存沿用 `agentos:agent:profile:update`。授权数据以服务端为准。切换 Agent 时隔离未完成的读写响应，避免 A 的结果显示在 B 的面板。
2. **将绑定作为可执行资源的上限。** 在会话创建运行上下文时，按已验证的租户和实际运行的 Agent 查询有效绑定。Skill 保留已有绑定类别、匹配逻辑及状态过滤，MCP 仅从有效绑定构建工具。工作空间、用户专属资源仍须满足调用者的可见范围。有效集合为空时该类资源为空。仅隐藏按钮会保留直接提交资源 ID 的通路，故服务端同时收回工作空间和客户端选择对授权的影响。
3. **复用正常执行和恢复路径的授权判断。** 在 conversations.service.ts 和 conversation-deepagents-chat.service.ts 中梳理发送、重新生成、快照挂载和审批恢复入口。恢复前根据当前有效绑定过滤 Skill 快照及 MCP 工具；若待审批动作对应的 MCP 已撤销，则结束该动作并向调用方返回原因。保留对话记录用于展示。
4. **拆开插件绑定与资源管理。** WorkspaceDetailPage.vue 移除 Skill/MCP 勾选、保存、知识库/数据源切换项及专属加载逻辑。为有权限用户保留创建、编辑、删除工作空间自有 Skill 的入口。首页草稿和聊天输入移除可自行追加 Skill/MCP 的选择及提交逻辑。其余插件配置字段继续按原有职责处理。

## Risks / Trade-offs

- **恢复路径重用快照。** 按服务端当前绑定筛选可装载的资源并检查待派发调用，用撤销后恢复的用例验证。
- **共享 Agent 可关联范围较窄的资源。** 绑定查询与运行时资源访问分别执行权限检查，覆盖跨租户和跨工作空间用例。
- **入口移除影响资源维护。** 单独验证工作空间自有 Skill 的管理流程，清除仅为已移除菜单服务的网络请求。

## Migration Plan

目标环境为空库。沿用已有绑定表，无需历史回填。先部署服务端授权逻辑和 Agent 管理入口，配置用于验收的 Agent，再验证 DataClaw 界面与运行时；如需回退，协调回退服务端和两个 Web 版本。
