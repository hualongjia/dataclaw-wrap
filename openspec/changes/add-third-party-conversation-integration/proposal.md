# 第三方应用会话接入

## Why

P3Demo 的智能问答入口尚为占位页。它的服务端已经调用 Agent OS，但 Agent 入口类型校验、模型选择与会话详情映射使用户无法完整使用已授权的 Agent，也无法在重新进入会话时继续计划问答。

## What Changes

- 在 P3Demo 智能问答页复用应用绑定的 Agent 和模型，按 Web 入口筛选、采用配置的默认项，并在发送前展示所选模型的个人凭证要求。
- 让用户在输入框切换计划模式，在同一会话中确认方案或补充答案，离开后可从历史继续回答。
- 复用最近会话、新建、详情、流式回复、取消、重新生成和工具审批接口，在 P3Demo 侧栏补齐置顶、改名和删除操作。
- 由 Agent OS 按已验证应用的配置执行会话访问校验。新接入的第三方应用默认仅可使用个人会话，已有工作空间能力的应用保留原范围。
- 在 `dataclaw/docs/third-party-api/` 维护可供其他应用及后续接入 skill 使用的上游接口说明和联调状态。

## Capabilities

### New Capabilities

- `third-party-conversation-integration` — 第三方 Web 应用的会话访问与交互契约，涵盖最近会话、正文与侧栏操作、Agent 与模型选择以及计划模式。首个接入方为 P3Demo。

### Modified Capabilities

## Impact

P3Demo 的业务后端、共享响应类型和智能问答页面需要配套调整。Agent OS 在应用登记信息中保存会话访问范围，由现有会话及工作空间会话入口统一执行。DataClaw Web 继续使用原有入口，纳入工作空间权限回归验证。定时任务由独立变更维护。
