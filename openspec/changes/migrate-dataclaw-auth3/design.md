# Design

## Context

AuthController 已有三条换票路由，共用 AuthService。两个浏览器前端各有运行时回调路由。ModelApiService 自行验票并维护本地应用配额。现有 access-matrix 提供只读展示，权限声明与守卫可复用。参考 p3demo 的应用角色管理、滑动会话和目录同步行为，适配现有 DataClaw 调用方。

## Goals / Non-Goals

最小化业务变化。用户和租户 ID 在 Auth2、Auth3 中一致，跨 JSON 边界保留十进制字符串精度。工作区和模型服务沿用现有本地标识。完整组织树、外部 Client 白名单、Auth3 Client 到模型应用的映射、KMS 独立应用迁移均在本期范围之外。

## Decisions

1. 每个登录事务记录目标端、一次性 state、verifier 和返回地址。管理端建议使用 /auth/admin/callback，用户端使用 /auth/web/callback，桌面沿用 dataclaw://auth/callback。桌面统一通过系统浏览器进行 OAuth 登录，移除账号密码表单、preload/IPC 密码入口和旧密码协议调用，不增加密码代登机器凭证。分别调用现有 /api/v1/{admin,web,desktop}/auth/token，服务端按目标选择 Client，以完全匹配的 redirect_uri 调用 Auth3 /oauth/token。隔离两端浏览器会话，处理失败、登出及切租户后缓存失效；采用滑动策略时允许省略 refresh_token，以 Auth3 租约验证为准，参照 p3demo 区分用户活动与后台轮询。
2. 封装服务端机器凭证获取及过期更新。模型 API 用机器令牌调用 /api/v3/oauth/introspect 并验证用户主体；其余 AppKey、供应商、用户密钥和 quotaContext 逻辑沿用。对其他用户接口按调用端校验身份和应用权限，避免把模型 API 的外部令牌规则扩展到后台管理。
3. 保留前端已有用户查询契约，在适配层转换 Auth3 的本人资料、授权租户、用户分页与批量人员查询结果。记忆管理用机器令牌和来自验证上下文的 X-Tenant-Id 调用 GET /api/v3/orgs/tree（org.tree），直接取 data 顶层 orgId，保留孤立父节点语义。定时消息对照现有 AuthMessageService 的收件人、渠道、附件、失败反馈逐项适配。
4. 管理写操作调用 /api/v3/app/*，令牌归属固定为 dataclaw-admin，成员受其当前租户约束。复用现有 permission guard、权限码和 usePermission，读取有效权限时按管理应用过滤。角色权限矩阵保存完整绑定，成员跨页增删提交差量；应用管理员资格与本地同步权限分别检查。
5. 参考 p3demo 全局目标配置、执行历史及逐租户结果模型。通过数据库唯一约束控制有效目标和目录业务键（租户、租户内用户）。每次执行快照化目标；分页完整后在单目标事务内幂等更新镜像并调和失效记录。机器授权收窄或分页失败时保留旧快照。调度采用可续租锁与数据库执行代次，过期持有者无法提交；定时任务按时间槽去重，接管时记录中断。默认调度间隔建议沿用参考项目的 30 分钟。

## Risks / Trade-offs

- Auth3 授权配置错误会影响多个入口。发布前分别验证三套回调、角色自管用户身份，以及机器 Client 的自省、目录、租户与消息接口授权。
- 组织 ID 的历史兼容性单独用现存记忆样本验证；异常时维持原访问边界。附件能力以真实交付结果验收。
- 同域 Cookie 或旧缓存可能串端；用同一浏览器同时登录管理端和用户端测试身份隔离。

## Migration Plan

先部署增量目录表和 Auth3 配置，登记管理端权限基线和初始管理员。联调环境验证三端登录、历史数据、AI 配额、角色成员、目录并发及通知，再协调切换服务与前端。回退时先停同步调度，再恢复配套的应用版本、认证配置与回调；保留同步数据及历史，用户重新登录。
