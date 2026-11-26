# resource-application-ownership Specification

## Purpose

使管理员可以在同一租户内标识 Agent、工具、MCP 服务和 Skill 为哪个业务应用建设，查找该应用的资源，同时独立维护直接调用和资源使用权限。

## Requirements

### Requirement: 四类资源的业务归属

系统 SHALL 为 Agent、工具、MCP 服务和 Skill 提供可空的单选归属应用，持久化归属并限定为当前租户。

#### Scenario: 设置与清空

- **WHEN** 有该资源编辑权限的管理员依次为四类资源选择 p3demo、保存并刷新，再清空
- **THEN** 每个资源详情可见相应归属，清空后显示未指定；已有及公共资源允许为空。

#### Scenario: 跨租户输入

- **WHEN** 管理员提交其他租户的应用标识
- **THEN** 写入被拒绝，原归属保持有效。

### Requirement: 独立查找业务归属与调用绑定

管理列表 SHALL 支持按归属应用查找四类资源，Agent 中心应分别标识归属筛选与绑定应用搜索。

#### Scenario: 归属和授权不同

- **WHEN** Agent A 归属 p3demo 但只绑定 sdt6，Agent B 归属其他应用但绑定 p3demo
- **THEN** 按归属 p3demo 查到 A；按绑定 p3demo 查到 B。四类资源列表按归属筛选时均只返回匹配的本租户记录。

### Requirement: 归属独立于授权和上下文

设置、变更或清空归属应用 SHALL 只修改业务归类，保留当前应用调用绑定、Agent 资源授权以及既有工作空间、租户和记忆边界。

#### Scenario: 共享 MCP

- **WHEN** MCP 归属 p3demo，sdt6 的 Agent 已获该 MCP 授权，p3demo 的另一 Agent 未获授权
- **THEN** 已授权 Agent 仍可在既有权限范围内使用，未授权 Agent 不能仅凭归属调用；调整 MCP 归属后两者权限保持原值。

#### Scenario: 工作空间 Skill

- **WHEN** 工作空间自有 Skill 被标记归属 p3demo
- **THEN** 既有工作空间归属和可见范围保留，业务归属本身不向其他工作空间开放资源。
