# P3Demo 会话产物抽屉

## Why

用户需要从回答中打开 Agent 交付的文件，也需要在重开会话后找回产物。Agent OS 提供的是会话文件树，其中同时包含交付文件、临时文件和上传附件，P3Demo 需要提供面向交付文件的读取入口。

## What Changes

- 在智能问答页提供右侧产物抽屉，按原层级展示当前个人会话 `generated/` 目录下的文件，包括回答中没有引用的产物。
- 将回答里的产物路径定位到抽屉中的具体文件，支持按类型预览以及下载原文件。
- Agent OS 为第三方用户身份精确开放会话文件树、节点预览和节点下载三个 GET 路由，沿用租户、应用、用户及个人会话归属校验；其他文件入口继续关闭。
- 由业务后端复用 Agent OS 的会话文件树、节点预览和节点下载接口，统一校验会话归属和交付目录。

## Capabilities

### New Capabilities

- `conversation-artifacts` — 个人会话交付文件的发现、定位、预览、下载和授权。

### Modified Capabilities

## Impact

P3Demo 的 React 页面、业务后端及共享响应契约需要配套接入。会话身份及页面入口依赖独立变更 `add-third-party-conversation-integration`。DataClaw Web 的 Vue 抽屉作为交互参照。第三方对接说明归入 `dataclaw/docs/third-party-api/`。
