# 规格增量

## ADDED Requirements

### Requirement: Agent OS 目录快照使用统一镜像

Agent OS 的租户及用户目录快照 SQL SHALL 统一读写 auth_tenant_mirror 和 auth_user_mirror，按租户标识及租户内用户标识区分归属；现行建库与升级结果均应移除三张旧目录缓存表。

#### Scenario: 当前基线建库

- **WHEN** 从 Agent OS 当前有效基线创建空库并完成目录同步
- **THEN** 租户资料可从 auth_tenant_mirror 读取，用户及其所属租户可从 auth_user_mirror 读取，库中不存在 cache_auth_tenant、cache_auth_user_info、cache_auth_user_tenant_rela。

#### Scenario: 升级与数据保留

- **WHEN** 对包含三张空旧表和已有镜像、同步目标及历史的 Agent OS schema 应用本次迁移
- **THEN** 仅三张指定旧表被清理，原有镜像与同步记录保持完整；随后同步两个租户的同一用户标识时，快照仍可分别读取。

#### Scenario: 独立库边界

- **WHEN** 部署 Agent OS 迁移时同一数据库存在 KMS schema 或其他 cache_auth_* 表
- **THEN** 本次迁移仅作用于目标 Agent OS schema 的三个表，KMS 和其他表继续可用。
