# 实施与验证记录（2026-11-25）

## 范围与环境

当前 8/12 项完成；4.1、4.2、5.1、5.2 保持未完成，等待真实 Web 认证与业务验收。

用户确认允许 PostgreSQL 替代本地验证、允许 packageExtensions 隔离 Desktop。生产
`00_precheck.sql` 的 SeaboxSQL 21.3.1.329 门槛未改。此次使用本机 PostgreSQL 17.12，
独立集群 `/tmp/dataclaw-runtime-upgrade/pgdata`、端口 55432、可丢弃数据库 `runtime_upgrade`。
**未执行 SeaboxSQL 验证，结果不代表生产验收。**

应用代码位于独立 Git 仓库 `dataclaw/`；本文与规划在外层仓库。原有部署 change 归档及
`dev-auth3-deployment` spec 均未改动或提交。

## 依赖与隔离（1.1–1.3）

基线代码：`763e51f2c466426bbef22952d9795875d923df78`。Desktop 从实际
`apps/dataclaw/desktop/src/main/chat/deepagentsChatLoop.ts` 解析，升级前后版本一致：

| 包 | 升级前 | 升级后 |
| --- | --- | --- |
| deepagents | 1.10.2 | 1.10.2 |
| langchain | 1.4.0 | 1.4.0 |
| @langchain/core | 1.1.48 | 1.1.48 |
| @langchain/langgraph | 1.3.0 | 1.3.0 |
| @langchain/openai | 1.4.7 | 1.4.7 |

根级 shameful hoist 在直接升级后会改变 Desktop 隐式导入，故按用户确认在
`pnpm-workspace.yaml` 为 Desktop 补充三项旧版本依赖。Desktop 源文件与 manifest 未改。
workspace YAML packageExtensions 需要新版 pnpm；`packageManager` 从 10.0.0 更新为
本机验证的 10.34.9，防止旧版静默忽略配置。

KMS 三个 workspace 的 importer、递归 link workspace（包括 @repo/fetch、@ej/ai）、
包元数据与 snapshots 全闭包比较完全相等。两份规范 JSON SHA256 均为：
`1a49af52ea51fa88739b9de0ff3612b4f1f76a21594144f5a9608b34ce23da7e`。
KMS 源文件及 manifest 未改。共享锁保留旧闭包并加入新服务闭包，未采用全局 override。

`pnpm install --frozen-lockfile` 成功；从服务和 deepagents 两个实际解析位置均验证 design
九项精确版本，并核对这些运行时之间的 peer 约束全部满足。安装器原有 SWC/chokidar
peer 警告不属于运行时九包。可复核脚本位于 `scripts/verification/runtime-upgrade/`：
`baseline.cjs`、`kms-lock.cjs`、`service-resolution.cjs`。

## PostgreSQL（2.1–2.2）

对照 checkpoint-postgres 1.1.4 发布包 migrations：checkpoint 0–5，store 0–3；新增
`checkpoint_writes.task_path text NOT NULL DEFAULT ''`。同步全量 DDL、迁移登记、postcheck
及初始化说明，历史 migration 保持历史内容；本次不提供旧数据迁移。

`init-runtime-local-postgres.sh` 只针对明确的本机 URL 替换版本门槛，保留正式 precheck
权限/空 schema 检查，再依序执行原全量 SQL 与 postcheck；三步均通过。

`postgres-smoke.cjs` 使用真实服务、PostgresSaver、PostgresStore 和 createDeepAgent：
两轮 4 条历史后关闭并重新建立连接，getState 重建 4 条历史，第三轮共 6 条且可引用首轮
信息；store 写读通过；重复销毁不重复关闭 pool。缺列用随机独立 schema 模拟并 finally
清理，启动被拒绝，不改动有效主 schema。线程随机生成，允许同库复跑。

## 对话、审批、流与权限（3.1–3.3）

主 Agent 显式添加 todoListMiddleware。新版文件权限默认未匹配时允许访问，因此在既有
允许路径后加兜底 deny，保持 Skills/上传目录只读、generated/temp 可写及启用记忆挂载。

`stream-smoke.cjs` 使用实际 DeepAgent 与既有 toUIMessageStream、服务收集器：

- write_todos `plan-1`、子 Agent task `child-1` 正确呈现工具状态；子 Agent 内部输出只在
  task 工具结果中，不进入主回复；主回复恰为一次 `FINAL-unique-answer`。
- 同轮同路径 `write-1` 成功，`write-2` 为 output-error：
  `Error: parallel file mutations to the same path are not allowed.`
- 真实 FilesystemBackend 加载 upgrade-skill 元数据，使用服务的权限规则验证上传路径写入
  被拒绝且文件不存在。保持根图订阅，避免不加区分转发子图内部输出。

审批新增生产事务内的 JSON 快照 claim 与会话状态 claim。`approval-smoke.cjs` 使用真实
Prisma/数据库 claim 函数与 DeepAgent/PostgresSaver，在中断后重建服务连接：

| 决定 | conversationId | messageId | 结果 |
| --- | --- | --- | --- |
| 批准 | 1795580478880001 | 1795580478880101 | 两个并发 claim 仅一个成功；count-call 执行一次；重复 claim 拒绝 |
| 拒绝 | 1795580478942002 | 1795580478942102 | 两个并发 claim 仅一个成功；count-call 执行零次 |

线程采用 `conversation:<conversationId>:assistant:<messageId>`，恢复的中断仍属于原线程。
脚本完成后清理测试业务行，checkpoint 保留。此为运行时/事务组件回归，**未代替真实 Web
审批入口验收**。已有 scoped checkpoint 正常路径无需读取原始增量消息；旧线程兼容逻辑保持。

`memory-smoke.cjs` 调用真实 ConversationMemoryConsolidationService.runConsolidationAgent，
以确定性测试模型调用 write_file，实际持久化并读回一条用户长期记忆后清理。测试模型用于
可重复组件回归，不作为真实模型/Web 端到端验收。

## 构建与测试

| 检查 | 结果 |
| --- | --- |
| agent-os-service check-types、build | 通过 |
| 会话/线程/心跳 Jest 三套 | 57/57 通过 |
| dataclaw-web 全量 Vitest | 229/229，52 套通过 |
| dataclaw-web build | 通过 |
| agent-os-web check-types、build | 通过 |
| PostgreSQL、审批、stream/Skills/权限、记忆归并 smoke | 通过 |

首次 Vitest 227/229：既有认证测试 fixture 未提供 pathname，且未适配异步 session probe。
仅补 fixture 并 mock/等待该测试的异步认证行为；生产认证逻辑未改。后续 229/229 通过。
两个 Web build 有既有大 chunk 提示。构建生成的 public/runtime-args.json 已还原，不提交
本地配置或密钥。commit hooks 的 ESLint、Prettier 均通过。

## 待完成的真实 Web 验收与具体阻塞

隔离实际服务已在 `http://127.0.0.1:5500` 成功启动，连接上述本地库；真实请求：

- `/api/v1/tenant-initialization` 返回 401。
- `/api/v1/web/auth/me` 携带 `Origin: http://localhost:5001` 返回 401
  `Browser session expired`。

需要提供或确认：可用于本地 Web/Admin 回调的 Auth3 测试登录、隔离租户初始化/管理权限、
可用于该租户的测试模型/网关授权与测试 Agent/工具配置。Redis 浏览器会话后端也需确认
隔离可用配置；未使用现有环境凭据写入外部业务系统。

拿到配置后继续通过 ChatPage/useConversationRuntime 发起同一会话、实际服务进程重启、
Web 批准及续聊，再通过 Agent OS SessionTaskDetailView 对照同会话状态、消息与工具结果。
记录真实 Web 会话与调用标识后才能完成 4.1、4.2、5.1、5.2。当前不声称已完成这些验收。

本机详细日志暂存 `/tmp/dataclaw-runtime-upgrade/`；可复现脚本已提交到代码仓库。

## 第二轮静态评审修复

发现原生产普通续聊为每个新 assistant 创建独立 thread，并只注入业务表文本历史，首轮工具
独有结果会丢失。现保留每轮线程及审批绑定，在新轮从所选历史的上一 assistant thread 调用
图 getState 重建完整增量消息（含 AI 工具调用与 ToolMessage），再传入新 thread；重生成
只选择截断后的历史。若上一轮仍有工具审批中断，拒绝新轮并提示先处理原审批，避免复制
悬空工具调用。checkpoint 缺失时保留既有业务历史回退。

新增 `cross-turn-smoke.cjs` 走生产 streamChatResponse、实际 DeepAgent/PostgresSaver：
conversation `1795580998971`，首轮 `conversation:1795580998971:assistant:1` 工具
`unique-tool-result` 返回 `tool-only-b054d3a8-f994-4c31-be82-e97775fa29e8`，首轮最终
仅答 `done`；关闭并重建服务数据库连接后，新轮 `assistant:2` 的模型可读到该工具独有
结果并回答同一标识，图状态 6 条消息且仅一个 ToolMessage。此补充覆盖了此前同线程
smoke 没有覆盖的生产跨轮线程策略；仍不替代待完成的真实 Web/进程重启验收。

同时强化流回归断言：plan-1 必须成功，write-1/write-2 恰一成功一失败，失败必须为同路径
并发冲突。KMS 闭包解析加入 pnpm 包别名支持，任何未解析非 link 依赖立即失败；重新
比较基线与现锁仍完全一致，SHA256 与上文相同。

第二轮修复后 service build/check-types、57 项会话 Jest、强化stream smoke、跨轮PG smoke
及原审批并发/拒绝 smoke 均通过。
