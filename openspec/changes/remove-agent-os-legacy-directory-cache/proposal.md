# 清理 Agent OS 旧目录缓存

## Why

Agent OS 新库建库脚本仍创建三张旧目录表，而 Auth3 目录同步已经写入租户镜像和租户内用户镜像。并存使 SQL 使用方容易选错数据源。

## What Changes

- 清理 Agent OS 的 cache_auth_tenant、cache_auth_user_info、cache_auth_user_tenant_rela，统一目录 SQL 到现有两张镜像表。
- 同时覆盖存量 schema 的增量迁移和从当前基线建库，两种路径应得到相同的目录结构。
- KMS 独立库、其他 cache_auth_* 表、同步历史与目标均在本次范围之外。

## Capabilities

### New Capabilities

### Modified Capabilities

- `tenant-directory-sync` 约束 Agent OS 目录快照的唯一存储及建库升级结果。

## Impact

涉及 Agent OS 的数据库迁移、生产建库 DDL 和目录同步集成验证，可独立发布。历史冻结备份按仓库约定保留。
