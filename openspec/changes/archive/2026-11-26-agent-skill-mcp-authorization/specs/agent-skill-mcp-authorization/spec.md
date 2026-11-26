# Agent 的 Skill 与 MCP 授权

## Purpose

管理员可以按 Agent 配置可用技能和 MCP 服务，让用户在个人或工作空间会话中获得一致的授权结果，并使解除绑定后的续聊和工具审批遵守最新授权。

## ADDED Requirements

### Requirement: 在单个 Agent 上维护两类授权

系统 SHALL 在 Agent 配置页提供与工具授权同级的 Skill 和 MCP 两个独立 Tab，按当前租户与目标 Agent 读写绑定。

#### Scenario: 分别设置两个 Agent

- **WHEN** 管理员为 Agent A 绑定一个可用 Skill 和一个 MCP 服务，刷新页面并切换到未配置的 Agent B
- **THEN** A 的两个 Tab 分别显示各自绑定，B 的两个 Tab 均显示未绑定

#### Scenario: 单独解除授权

- **WHEN** 管理员在 A 的 MCP Tab 解除一项绑定并重新打开该 Agent
- **THEN** 该 MCP 显示未绑定，A 的 Skill 及其他 MCP 绑定保持原值

#### Scenario: 越权更新

- **WHEN** 仅有查看权限的用户直接提交绑定，或管理员提交其他租户专属资源的 ID
- **THEN** 服务端拒绝变更，原有绑定保持原值

### Requirement: 会话仅使用当前 Agent 的有效授权

系统 SHALL 按当前 Agent 的有效绑定和既有资源访问范围加载 Skill 与 MCP，客户端输入不能扩大授权。

#### Scenario: 绑定后发起会话

- **WHEN** 用户选用已绑定可用 Skill 和 MCP 的 Agent，在个人或工作空间会话中发送消息
- **THEN** 用户可使用其中符合现有访问条件的能力，无需再次选择或绑定

#### Scenario: 无授权的 Agent 收到资源 ID

- **WHEN** 用户改用未绑定这些资源的 Agent，工作空间配置、会话配置或消息请求携带此前的资源 ID
- **THEN** 本轮不会加载相应 Skill，也不会向该 MCP 服务发出工具调用

#### Scenario: 解除绑定后恢复执行

- **WHEN** 管理员解除绑定，用户随后续聊、重新生成回答或批准此前暂停的工具调用
- **THEN** 服务端按最新绑定确定可用资源，已解除的 Skill 不再装载，已解除的 MCP 调用不会派发

### Requirement: 收回工作空间插件的绑定入口

DataClaw Web SHALL 移除工作空间插件中的 Skill、MCP 绑定操作，以及知识库、数据源两个菜单。

#### Scenario: 编辑工作空间插件

- **WHEN** 有工作空间编辑权限的用户打开插件区域
- **THEN** 页面没有 Skill、MCP 绑定、解绑或保存绑定操作，也没有知识库、数据源菜单及关联选择弹窗

#### Scenario: 创建工作空间自有 Skill

- **WHEN** 有权限的用户创建或编辑工作空间自有 Skill
- **THEN** 该资源仍可管理，此操作不会给 Agent 新增 Skill 绑定
