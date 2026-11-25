# Tasks

## 1. 后端 DEV 配置

- [x] 1.1 核对部署仓库规则与工作区，逐项比对 DEV .env、源码 .env.example 及 Auth3 DEV 登记，生成不含凭据的配置对应记录；明确 Redis、会话密钥的来源和未核实项，确认管理资格由 Auth3 判断。
- [x] 1.2 更新 agent-os-service DEV 的管理、用户和机器 OAuth 配置及登录退出回调；通过脚本比对两份资料 DEV 段确认 Client 与 URI 一致，输出不含 Secret。
- [x] 1.3 补齐 Redis、独立会话、期限、Cookie、CORS 和应用编码设置，目录同步默认关闭；检查必需 KEY、来源与回调匹配，确认未复制本机密钥，并移除停用的角色身份配置。
- [x] 1.4 按源码证实的替代关系清理旧 KEY，保留非认证业务值及桌面配置；通过差异检查说明每项变化，并补充外层 docs/services/agent-os-service.md 的配置来源与依赖。

## 2. 前端 DEV 运行配置

- [x] 2.1 清理 agent-os-webapp DEV JSON，仅保留源码读取的配置，补齐认证站点地址并设置 dataclaw-admin-dev、/auth/callback；验证 JSON 无重复键、无缺失引用、无敏感凭据。
- [x] 2.2 同步 dataclaw-webapp DEV JSON，保留桌面下载入口与业务地址，设置相同管理应用编码和正确回调；验证源码 KEY 覆盖及 JSON 结构。
- [x] 2.3 在外层 docs/services/agent-os-webapp.md 与 dataclaw-webapp.md 记录服务映射、域名、配置用途及刷新条件；核对与实际 Deployment 和脚本一致。

## 3. 发布准备与一致性验收

- [x] 3.1 核对三个 redeploy.sh 的配置注入与 Pod 刷新流程，仅修复阻止新配置生效的问题；对修改脚本运行 bash -n，对配置与清单运行现有可用静态检查，记录结果。
- [x] 3.2 提供 Jenkins DEV 参数清单、三个服务发布顺序、源码分支与提交核对步骤、前端 REDEPLOY_CONFIGMAP=YES 及人工回退说明；检查文档不含凭据且不暗示已发布。
- [x] 3.3 对三个 runtime 文件执行跨端一致性与差异检查，确认未修改其他环境，输出验证记录，列明环境未核实项及后续登录、退出、租户隔离、管理资格和历史数据源验收步骤。

## Workflow follow-up

- 实施不触发 Jenkins 或 Kubernetes；获后续发布指令后才执行真实环境发布及业务验收。
- 在真实环境核实管理员角色、机器权限及 Redis；未核实时不得声称部署就绪或在线验收通过。
