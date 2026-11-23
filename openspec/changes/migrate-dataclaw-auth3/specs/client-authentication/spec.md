# Spec Delta

## Purpose

为管理端、用户端和桌面端提供独立的 Auth3 登录入口，使用户在当前租户下继续使用原工作区、记忆管理与已有消息通知功能。

## ADDED Requirements

### Requirement: 三端登录
系统 SHALL 绑定 Client、回调、state 和 PKCE S256。

#### Scenario: 成功回调
- **WHEN** 任一端完成授权
- **THEN** 从原端换票路由登录，用户及租户 ID 对应已有业务记录

#### Scenario: 事务错配
- **WHEN** state、端或 verifier 不匹配
- **THEN** 拒绝建立会话

#### Scenario: 桌面登录入口
- **WHEN** 未登录用户打开桌面端
- **THEN** 通过系统浏览器完成 OAuth 登录并以深链返回；桌面不提供密码登录表单或密码登录 IPC

### Requirement: 会话生命周期
系统 SHALL 支持当前租户、登出及已配置的令牌策略。

#### Scenario: 滑动令牌
- **WHEN** 有效滑动令牌未带刷新令牌
- **THEN** 用户可登录，后续验证遵循 Auth3 租约

#### Scenario: 三端用户活动续租
- **WHEN** 管理端、用户端或桌面端由用户交互触发本端活动上报，且滑动令牌有效
- **THEN** 系统 SHALL 使用该用户令牌向 Auth3 续租一次，包括桌面端以 Bearer 令牌上报的情形

#### Scenario: 普通请求、后台轮询与恢复探测
- **WHEN** 任一端发出普通业务请求、后台轮询或页面恢复探测
- **THEN** 系统 SHALL 在认证流程中仅验证令牌，这些请求不会触发活动续租

#### Scenario: 滑动模式关闭自动刷新
- **WHEN** 任一端在滑动模式下发出请求，无论是否携带刷新令牌
- **THEN** 系统 SHALL 禁止自动刷新用户令牌，租约失效时拒绝请求并要求重新登录

#### Scenario: 切换与登出
- **WHEN** 用户切租户再退出
- **THEN** 权限按新租户加载，退出后受保护页面要求认证

### Requirement: 既有目录与通知
系统 SHALL 通过 Auth3 保留工作区用户解析、记忆范围及定时消息。

#### Scenario: 根组织
- **WHEN** 管理员读写部门记忆
- **THEN** 只允许当前租户根组织范围；目录失败时不会扩大范围

#### Scenario: 用户与附件
- **WHEN** 工作区查询成员或任务发送带附件通知
- **THEN** 返回原用户资料或向原收件人交付附件
