# Design

## Context

三个服务共享 dataclaw 源码；部署实现位于 universal-deployment-wrap/universal-deployment 独立仓库。当前后端 DEV .env 缺少多项 Auth3 配置；前端存在旧认证 KEY 和旧回调路径。前端 redeploy.sh 在 REDEPLOY_CONFIGMAP=NO 时复用集群配置，后端现有脚本直接调用 configmap-deploy.sh。现有前端 Deployment 已有配置摘要和发布标识。

## Goals / Non-Goals

**Goals:** 在保持服务名及部署结构的前提下，使 DEV 配置符合当前源码；把可离线完成的检查与需要真实环境的验收区分开。

**Non-Goals:** 不重构部署平台，不调整其他环境，不发布集群，不修改业务鉴权语义或历史数据。

## Decisions

1. 以服务端 .env.example 的 KEY、分组与注释作为配置契约，以现有 DEV 值保留数据库、工具、模型及存储配置。只在源码证实替代关系后迁移旧 KEY，不把缺少的可选项一律设为必填。不会整份复制本机 .env，以免引入本机凭据和路径。
2. OAuth 管理用户、普通 Web 用户、管理机器客户端分别从两份资料 DEV 段读取；Secret 仅留在服务端配置链路，不复制到规划或服务说明。沿用现有部署注入模式，不扩展为凭据管理平台改造。会话密钥在 DEV 多实例间稳定一致；不得复制本机密钥。
3. 两前端仅保留当前源码引用的 KEY，以最近清理后的源码配置为清单，替换环境值。两端应用编码均为 dataclaw-admin-dev，这是当前 Base 归属契约，并非用户端取得管理权限。保留前端实际生效的 DEV 服务地址。
4. HTTP DEV 来源配套 COOKIE_SECURE=false、同站策略及受信任域设置；精确列出两前端来源。Auth3 Issuer 按实际服务配置与 Auth3 路由核对，不因路径中含 auth2 就推断为旧协议。令牌策略按登记的跟随会话语义及当前实现核对。
5. 管理资格由 Auth3 应用管理员关系核验，不配置本地角色 ID 或编码；目录同步首次保持关闭，完成机器权限与目标范围核验后另行启用。桌面 OAuth 保留原配置。
6. 优先只改三个 runtime 文件；若发布脚本无法装载当前配置才做最小修改。补充外层三个 docs/services 服务说明，遵守该仓库 AGENTS.md；规划根的 allowedEditRoots 不覆盖部署仓库，实施时应按用户明确目标确认并记录跨仓库范围，不假定同一 Git 仓库。

## Risks / Trade-offs

- Redis 可达性或机器接口授权未核实 → 在配置交付中明确阻塞项，不伪造值、不宣称上线可用。
- 切换 Base appCode 使历史数据源不可见 → 验收现有工作区；记录差异，任何数据迁移另行确认。
- subPath 配置挂载不自动刷新 → 沿用现有 Pod 刷新方式并核对实际配置摘要。
- 旧客户端会话与新认证不兼容 → 发布说明要求重新登录；失败由人工回退整套匹配版本及配置。

## Migration Plan

实施阶段先核对部署仓库状态及规则，修改 DEV 配置并进行 JSON、KEY、回调一致性和 Shell 静态检查。分别记录两个仓库的变更及验证结果，提供 Jenkins 参数清单：三个服务名、DEPLOY_ENV=dev、前端 REDEPLOY_CONFIGMAP=YES、包含迁移提交的 BRANCH_ENV（当前候选 dev-auth3，发布前确认提交）。本变更不触发发布。

获另行发布指令后，准备依赖及已核实身份，协调更新后端与两个前端，检查配置生效、双端登录、切租户、退出、管理资格与数据源可见性。回退由人工恢复配套源码及配置并刷新 Pod，不自动重放写入。
