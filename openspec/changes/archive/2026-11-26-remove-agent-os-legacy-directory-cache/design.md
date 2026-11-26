# Agent OS 目录缓存清理

## Context

DirectorySyncService 按租户写 auth_tenant_mirror，并按 (tenant_id, user_id) 写 auth_user_mirror。前期只读核查中，Agent OS 配置指向的 samatrix_auth3_new 同时存在两套表，三张旧表均为空。KMS 的 auth-sync、审批和邮件模块在独立 schema 使用同名表。

## Goals / Non-Goals

让现行 Agent OS 基线建库与增量升级得到相同的可运行目录结构。本次限定三张表；组织、角色缓存及 KMS 的同步和查询按各自模块管理。

## Decisions

1. 列举清理目标 cache_auth_tenant、cache_auth_user_info、cache_auth_user_tenant_rela，迁移显式限定部署选定的 Agent OS schema，避免依赖连接的宽泛 search_path。使用仓库事务迁移及版本登记约定，逐表删除，无 CASCADE。
2. Agent OS 的目录资料读取以镜像 profile 为来源，租户成员关系使用 (tenant_id, user_id) 和 active 条件。代码核查中目录同步已经写新镜像；仅修改仍依赖旧表的现行 SQL 和构建检查，保留分页完整后再提交以及缺失成员置为不活跃的原语义。
3. database/README.md 区分现行 ddl/server/samatrix、ddl/server/samatrix_auth3 和 schema-versions 冻结备份。更新现行安装基线及前后置检查；旧表名在本次删除语句、缺表断言及历史备份中具有明确意义，禁止跨库批量替换。

## Risks / Trade-offs

- 目标环境实际数据可能不同于先前抽样 → 迁移前核对已解析的 schema、目标表数据及依赖；若出现非空旧数据或未知引用，事务中止并保留原数据供核对。
- 同库不同 schema 存在同名 KMS 表 → 用隔离测试库的哨兵表验证其他 schema 不受影响。

## Migration Plan

先确认现行运行 SQL 能在无旧表的 schema 上完成目录同步与查询，保留目标三表结构备份，再应用事务迁移。另从两个现行建库入口验证结果。失败回滚事务；如部署后需回退，仅恢复目标 schema 的必要旧结构及其备份，保留新镜像和同步历史。
