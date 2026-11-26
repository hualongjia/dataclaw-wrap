# 资源归属应用

## Why

运营人员需要区分为 p3demo 建设的资源与 p3demo 获准调用的 Agent。现有租户、工作空间与授权字段无法单独说明业务归属。

## What Changes

- 在 Agent、工具、MCP 服务、Skill 上增加同租户、单选、可空的归属应用。
- 管理界面支持设置、显示和按归属查找；Agent 中心分别支持归属筛选和绑定应用搜索。
- 保留应用调用绑定、Agent 资源授权及 Skill 既有归属范围的独立语义。

## Capabilities

### New Capabilities

- `resource-application-ownership` 覆盖四类资源的业务归属与查找。

### Modified Capabilities

## Impact

涉及四类资源注册模型、管理服务、DTO 与页面，可独立于委派上线。
