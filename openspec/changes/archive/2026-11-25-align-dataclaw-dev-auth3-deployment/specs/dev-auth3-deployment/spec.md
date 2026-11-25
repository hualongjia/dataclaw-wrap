# Spec Delta

## Purpose

确保 DataClaw 管理端后端、管理端前端和用户端前端在 DEV 环境采用一致的 Auth3 登记信息与浏览器会话配置，并能明确验证部署配置是否生效，避免使用遗留认证参数或其他环境的应用身份。

## ADDED Requirements

### Requirement: DEV 认证身份一致性
DEV 部署 SHALL 为管理端和用户端采用各自 DEV 用户客户端及完整登录、退出回调，管理机器调用使用 DEV 管理机器客户端，管理应用与 Base 数据源归属统一使用 dataclaw-admin-dev。

#### Scenario: 两端认证
- **WHEN** 用户分别从 agent-os.dev.jinxin.cloud 与 dataclaw.dev.jinxin.cloud 登录并退出
- **THEN** 各自使用已登记的 DEV Client 和对应域名的 /auth/callback、/auth/logout/callback，不使用 localhost 身份

#### Scenario: 管理与数据源归属
- **WHEN** DEV 服务查询管理资格或创建、筛选 Base 数据源
- **THEN** 使用 dataclaw-admin-dev 并保留现有租户及所有者限制，历史归属不一致时报告差异，不自动迁移

### Requirement: DEV 浏览器会话配置完整
DEV 部署 SHALL 提供共享 Redis、独立浏览器会话标识、稳定会话密钥、固定期限及与浏览器来源匹配的 Cookie 和 CORS 配置。

#### Scenario: 跨实例与双端访问
- **WHEN** 浏览器访问两个 DEV 前端并由不同后端实例处理请求
- **THEN** 会话能够被一致识别，两端身份隔离，合法来源请求可携带 Cookie，非法来源不能绕过校验

### Requirement: 前端运行配置与当前代码一致
两个 DEV 前端 SHALL 仅提供当前代码使用的运行配置，保留所需业务地址及应用编码，不向浏览器提供 OAuth Secret 或服务端会话密钥。

#### Scenario: 配置加载
- **WHEN** 浏览器加载任一 DEV 前端的 runtime-args.json
- **THEN** JSON 无重复 KEY，包含该端所需配置，回调路由为 /auth/callback，且不包含遗留 Auth2 认证入口和未使用的客户端参数

### Requirement: 配置发布可验证
发布说明 SHALL 明确源码版本、ConfigMap 更新条件、配置刷新方式以及构建成功与认证验收的区别。

#### Scenario: 发布新配置
- **WHEN** 操作人员按照说明发布三个 DEV 服务
- **THEN** 使用包含 Auth3 迁移的源码版本，更新运行配置并刷新挂载，核对实际返回配置及两端登录退出结果，不以 Jenkins SUCCESS 代替业务验收
