# 将 Skill 与 MCP 授权归属到 Agent

## Why

工作空间插件和聊天输入都能影响 Skill 与 MCP 的选择，同一 Agent 在不同入口可能获得不同的执行能力。管理员需要在 Agent 配置页统一授权，并让服务端在实际执行时落实授权范围。

## What Changes

- 在单个 Agent 配置页增加与「工具授权」同级的「Skill」「MCP」两个 Tab，分别维护绑定。
- **BREAKING** 移除 DataClaw 工作空间的 Skill、MCP 绑定操作。会话使用当前 Agent 的有效授权，工作空间配置、会话配置和请求参数均不能追加授权。
- 从工作空间插件移除「知识库」和「数据源」菜单及其绑定交互。本期不为这两类资源建立 Agent 授权。
- 部署目标为空库，无历史绑定迁移。第三方应用 API 接入另行规划。

## Capabilities

### New Capabilities

- `agent-skill-mcp-authorization`：按 Agent 管理 Skill 与 MCP 授权，限定会话执行范围，并收回 DataClaw 的工作空间绑定入口。

### Modified Capabilities

## Impact

涉及 agent-os-web 的 Agent 配置页、agent-os-service 的授权读取与会话执行、dataclaw-web 的工作空间插件和会话能力选择。复用既有 Agent 绑定表、管理接口及权限码。
