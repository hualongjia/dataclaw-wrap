# 实施核对记录

## 任务 1.1

- universal-deployment-wrap 与其 universal-deployment 独立仓库工作区均干净；已读取外层 AGENTS.md，内层未发现 AGENTS.md。
- 已核对三个 DEV runtime 配置与源码契约。后端旧登录回调使用 /api/auth/callback，缺少管理机器客户端、独立会话、Redis、来源与角色配置；前端存在旧 Auth2 KEY。
- 管理用户、Web 用户、管理机器 Client 及 Secret 来源：dataclaw-wrap/docs 两份 Auth3 资料的 DEV 段；不在此记录真实凭据。
- 最新源码通过 Auth3 应用管理员关系核验资格，不读取系统管理员角色 ID 或编码。
- 后端 DEV .env 未提供 DATACLAW_REDIS_URL 和 DATACLAW_SESSION_SECRET；部署 DEV 配置中未发现 DATACLAW_REDIS_URL  的既有来源。不能用 localhost Redis 替代。
- 待明确 DEV Redis 连接来源，以及是否已有需要沿用的 DEV 会话密钥；新会话密钥可在确认无既有值后生成并稳定保存。

尚未修改部署配置或触发任何发布。

## 实施与静态验证结果

维护人明确要求 Redis URL、会话密钥先留空，稍后自行填写。本次按该决定完成占位配置，不将空值作为真实环境验收通过。

- DEV OAuth 三组 Client/Secret 与资料 DEV 段逐项一致；未在日志或说明中输出 Secret。
- 两前端 JSON 无重复 KEY，源码 runtimeArgs 引用覆盖完全，应用编码与后端一致，回调路由正确。
- 后端所有既有非认证值及桌面 OAuth 值保持不变；四个旧数据库工具 KEY 按源码替代关系迁移，公共数据库配置保留。
- 三个 redeploy.sh 的 bash -n 通过；人工核对配置挂载、摘要、release-id 与刷新链路，无需改脚本或 Deployment。未运行镜像构建、CSP 容器检查或 Kubernetes 服务端清单验证。
- 部署内层仓库仅修改三个 runtimes/dev 文件；其他环境未变。三个仓库 git diff --check 均通过。
- 外层新增三个服务说明与 DEV 发布说明，包含 Jenkins 参数、人工回退及在线验收清单。
- 待维护人填写两项空值，核实 Auth3 路由可达性、机器权限、真实角色绑定、数据库迁移及历史数据源。尚未触发 Jenkins、集群发布或在线验收。

## 停用配置清理

已核对 RolePermissionsService.eligibility，管理资格由管理 Client、应用归属和 Auth3 应用管理员关系判断。移除本地 .env、示例和 DEV 部署中的两个停用角色配置项，并修正发布前填写要求。仅 Redis URL 和会话密钥保留为空。
