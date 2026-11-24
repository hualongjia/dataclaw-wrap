# Proposal

## Why

现有租户目录同步页把每次执行的逐租户结果直接铺在页面上，历史增多后难以浏览，操作样式也与管理端其他页面不一致。参考 p3demo 的同步交互和发布顺序，统一操作体验，并在启用自动调度前验证手动链路。

## What Changes

- 用 DataClaw 管理端现有面板、表格、状态标签和弹窗重排同步目标与执行历史。
- 历史按执行分页，每行提供逐租户详情入口。新增和移除目标复用管理端表单及确认交互。
- 补充目录同步发布与回退说明，采用关闭调度部署、完成手动联调、统一开启调度的顺序。

## Capabilities

### New Capabilities

无。

### Modified Capabilities

- `tenant-directory-sync`：在 `migrate-dataclaw-auth3` 已定义的能力上补充管理页面要求和验收场景。本变更依赖该迁移，归档时先归档迁移变更。

## Impact

涉及 AgentOS Web 同步页面和 DataClaw 部署文档。沿用现有同步接口、权限、审计、数据库迁移与调度语义。发布验证使用目标联调环境。
