# 规格增量

## Purpose

使内置客户端和第三方应用能够按租户独立选择可直接调用的 Agent，让管理员维护默认入口，并让列表展示、会话执行和授权撤销遵守同一份应用绑定。

## ADDED Requirements

### Requirement: 应用与 Agent 的入口类型多选

应用管理及 Agent 编辑页 SHALL 分别支持从 web、desktop、api 中多选并回显已保存值。

#### Scenario: 双端入口

- **WHEN** 管理员为应用选择 web 和 api，为 Agent 选择 web 和 desktop 并保存
- **THEN** 刷新后两组选择分别保留；两端均可按所含渠道筛选 Agent，表单不再提供 both、lite 或 agentos。

#### Scenario: 历史渠道迁移

- **WHEN** 升级旧应用和 Agent 的入口类型
- **THEN** web、desktop、api 单值转换为单项数组，Agent both 转换为 [web, desktop]；历史应用 lite 转换为 [web]，agentos 转换为 [api]，已有 App-Key 保持不变。

### Requirement: 双向维护应用绑定

系统 SHALL 允许具备应用编辑权限的用户在新增和编辑时选择本租户多个 Agent，并从绑定集合中指定至多一个默认项。

#### Scenario: 独立默认

- **WHEN** p3demo 和 sdt6 都绑定 A、B，分别设置 A 和 B 为默认
- **THEN** 重新打开两个应用时，各自的绑定和默认项均与保存值一致。

#### Scenario: 非法提交

- **WHEN** 用户提交其他租户的 Agent、未绑定的默认项，或无应用编辑权限而提交变更
- **THEN** 服务端拒绝请求并保留原绑定和默认项。

### Requirement: Agent 中心反查和解绑

系统 SHALL 在 Agent 列表展示已绑定应用，支持按应用名称或编码检索，在编辑页的绑定应用 Tab 展示应用名称、编码、默认标记并按应用编辑权限解绑。

#### Scenario: 跨页检索

- **WHEN** 管理员用应用名称的一部分或用 p3demo 编码搜索
- **THEN** 匹配绑定关系的 Agent 在所有分页数据中检出，各出现一次，分页总数与结果一致。

#### Scenario: 解除默认项

- **WHEN** 管理员从 Agent 页解除 p3demo 的默认绑定并确认清空提示
- **THEN** 应用页同时反映解绑和默认项清空；其余 Agent 保持排序，sdt6 的配置保留。

### Requirement: 按已验证应用返回并排序

应用 Agent 列表 SHALL 按有效 App-Key 和当前租户返回已发布的已绑定项，可见默认项第一，其余按 sortNo 和创建时间升序排列。

#### Scenario: 应用隔离

- **WHEN** p3demo 绑定 B、C 并指定 C 为默认，A 只绑定 sdt6
- **THEN** p3demo 的列表依次返回 C、B；应用上下文无效时不会返回租户级 Agent 清单。

#### Scenario: 聊天筛选

- **WHEN** Web 和桌面聊天获取各自应用列表，默认项与本端入口不匹配
- **THEN** 各端只展示入口包含本端类型的可选 Agent；不可见默认项被过滤，剩余顺序保留。已有有效选择继续选中，否则选择可见第一项。

### Requirement: 直接执行与撤销

服务 SHALL 在应用直接访问 Agent 详情、启动执行、续聊、重新生成及恢复待审批任务时校验入口 Agent 的有效应用绑定，并保留原会话历史的既有查看权限。

#### Scenario: 跨应用直接启动

- **WHEN** 同一用户从 p3demo 提交仅绑定 sdt6 的 Agent ID
- **THEN** 服务拒绝读取该 Agent 详情和直接执行，既不调用模型也不派发工具。

#### Scenario: 运行期间解除入口绑定

- **WHEN** 用户有权查看历史，但原入口 Agent 已解绑，此时续聊、重新生成或批准待执行操作
- **THEN** 历史仍可查看，服务在新的执行副作用产生前停止并提示选择可用 Agent。

### Requirement: 内置应用的初始绑定

系统 SHALL 在新租户首次初始化时为内置 Web 和桌面应用各建立一个默认绑定，并保留后续管理员调整。

#### Scenario: 首次与重跑

- **WHEN** 租户完成首次初始化，管理员随后解绑 WebAgent 并重新运行初始化
- **THEN** 首次 dataclaw-web 仅绑定并默认选用 WebAgent，dataclaw-desktop 仅绑定并默认选用 DesktopAgent；重跑保留解绑和自定义默认设置。
