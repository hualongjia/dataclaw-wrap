# Spec Delta

## Purpose

模型调用方继续以用户令牌、业务应用密钥和供应商编码发起请求，使外部应用的用户可以使用已有模型服务，且用量归属仍可按原有应用核对。

## ADDED Requirements

### Requirement: 模型 API 身份
系统 SHALL 接受有效 Auth3 用户令牌并以 sub 和 tid 确定用户及租户。

#### Scenario: 外部应用
- **WHEN** 外部 Client 的有效用户令牌同时携带本租户有效 SeaboxAppKey 和有效 SeaboxProviderCode
- **THEN** 模型列表及推理沿用原流程，无额外来源白名单或 DataClaw 应用授权要求

#### Scenario: 无有效身份
- **WHEN** 自省失败、active 为 false、token_use 非 access 或缺少 sub/tid
- **THEN** 拒绝访问且不转发请求

### Requirement: 本地配额归属
系统 SHALL 继续根据租户内的有效 AppKey 解析本地应用与密钥记录。

#### Scenario: 推理用量
- **WHEN** 模型请求通过原有配额检查
- **THEN** 按原 applicationId、applicationApiKeyId 和用户记录用量

#### Scenario: 拒绝路径
- **WHEN** 密钥缺失、属于另一租户或配额耗尽
- **THEN** 沿用原有错误语义，配额耗尽返回 429
