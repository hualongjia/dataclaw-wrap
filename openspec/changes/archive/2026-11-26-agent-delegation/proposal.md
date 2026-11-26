# 多层 Agent 委派

## Why

Agent 既可能作为一个应用的入口，也可能在另一条任务链上受委派。固定类型和单层装配限制同一 Agent 在这两种场景中复用。

## What Changes

- **BREAKING** 取消主／子固定类型及其候选过滤；任一同租户有效 Agent 可作为委派目标并继续委派。
- 允许多个父 Agent 复用同一目标；拒绝自绑和间接环路。
- 逐层装配执行、解析模型、校验委派边和资源授权；应用直接绑定只校验入口。

## Capabilities

### New Capabilities

- `agent-delegation` 覆盖显式委派关系、多层执行及模型选择。

### Modified Capabilities

## Impact

依赖 application-agent-access 的入口绑定。涉及 Agent 中心、客户端选择器、模型配置、委派存储及 Deep Agents 会话执行。
