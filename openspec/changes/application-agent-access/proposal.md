# 应用绑定与直接访问

## Why

第三方应用需要独立管理多个 Agent。当前内置客户端按入口类型列出 Agent，第三方只返回一个默认项，列表与实际会话调用也缺少统一的应用绑定校验。

## What Changes

- **BREAKING** 应用类型、Agent 入口类型分别改为 web、desktop、api 的多选集合。按已验证应用返回已发布绑定项，默认项置顶。
- 在应用新增和编辑页维护绑定和默认项，在 Agent 中心查看、搜索及解绑。新租户首次建立两对内置绑定。
- 在直接启动、续聊、重新生成及恢复审批时校验有效绑定，保留既有会话历史访问边界。

## Capabilities

### New Capabilities

- `application-agent-access` 覆盖应用接入、入口选择和调用授权。

### Modified Capabilities

## Impact

涉及应用注册、Agent 查询、会话及任务入口、初始化、两个管理页面和 DataClaw 两端。先实施本变更，再实施 agent-delegation。
