# Proposal

## Why

DataClaw 两个 Web 端和共享服务依赖 Auth2，桌面端也经过该服务换票。迁移到 Auth3 时需要保留业务调用契约，并为管理端补齐应用角色维护和租户目录同步。

## What Changes

- **BREAKING** WebAdmin 和用户 Web 改用服务端独立会话。两个浏览器端分别持有签名会话 Cookie，多实例共享 Redis；旧浏览器登录态在切换时要求重新登录。桌面端和外部模型 API 沿用 Bearer 契约。
- **BREAKING** 三个登录客户端切换到 Auth3，分别登记回调；切换后旧 Auth2 会话需要重新登录。
- 桌面统一使用系统浏览器 OAuth 登录，移除账号密码表单和密码登录 IPC。
- 迁移用户资料、工作区目录、记忆根组织查询和定时消息调用；用户和租户沿用原 ID。
- 模型 API 使用 Auth3 自省，继续按租户和本地 AppKey 验证应用、检查配额并记录用量。
- 管理端新增自身应用的角色权限管理、全局租户同步管理；用户端仅迁移认证和已有目录调用。
- 两个 Web 端核对页面预期租户，保护并发切租户，限制本地会话最长寿命，防止退出回调重放，并在部署入口启用 CSP。管理授权变更、同步配置变更和手动同步采用前置操作审计。

## Capabilities

### New Capabilities

- `client-authentication` 三端登录、身份与既有 Auth 业务适配。
- `model-api-access` 用户令牌和应用密钥共同确定 AI 调用身份及用量归属。
- `admin-role-permissions` 管理端应用角色、权限码及成员维护。
- `tenant-directory-sync` 全局配置、执行和查看租户及用户目录同步。

### Modified Capabilities

无既有主规格。

## Impact

涉及 dataclaw/apps/agent-os/{service,web}、apps/dataclaw/web 和桌面认证入口，目录镜像及执行记录需要数据库迁移。实施参考同级 p3demo-wrap 的认证、role-permission-management 和 auth-directory-sync 模块，以及本项目 Auth3 技能中的 Open API 契约。
