# 记忆范围实施验收记录

日期：2026-11-26。变更：`agent-memory-scopes`，schema：`spec-driven`。

## 验收环境与边界

- 服务端测试使用已配置 PostgreSQL，在每次运行创建独立的 `memory_scope_test_*` / `memory_runtime_test_*` schema，仅复制表定义并写入测试数据，结束后删除 schema。业务 schema 的现有画像和记忆内容未被修改。
- 联合测试使用真实 DeepAgent 文件工具、LangGraph HITL 中断恢复、PostgresStore、PostgresSaver、Prisma，以及会话、记忆管理和后台沉淀服务。身份上下文和模型工具调用由确定性 fixture 提供；隔离结果直接核对实际存储与管理服务回显。
- 联合 fixture 包含应用 301/302、Agent 601/602、用户 201/202，以及两个租户和后台专用用户。每个 `it` 清理可变表并建立自身 fixture，可用 Jest `-t` 独立执行。
- 页面验收在本机 Chrome 和运行中的 Web 开发服务执行，HTTP 响应使用隔离 fixture，覆盖真实 Vue 组件与保存失败处理。

## 已验证行为

| 任务 | 验收依据 |
| --- | --- |
| 1.1–1.2 | Agent 管理、模板和两段运行时默认值一致；单开关更新保留其他配置。真实数据库重载及浏览器刷新、切换 Agent 恢复各自配置。 |
| 1.3 | 跨租户自定义与空配置重置；真实无画像 Agent 读取默认值；其他画像字段及旧用户记忆保留；注入版本写入失败后事务回滚并可重试；并发执行只记录一次；5.2 秒锁等待可完成；成功后管理员改值不被再次重置。 |
| 2.1–2.2 | 六类范围经真实工具持久化；两用户、两应用、两租户和两 Agent 核对共享与隔离；通用称呼跨应用、应用个人快照及优先规则；关闭六范围后独有新文件均未持久化，工具及快照无法取回内容；缺应用时两应用范围的新文件均不存在。 |
| 2.3 | 真实 checkpoint 含独有私有标记、工具写入参数和待办；变更范围后清理这些状态，拒绝旧动作并实际恢复完成。会话入口拒绝其他应用/租户、跨作者重生成以及原人失去成员资格的执行；允许安全拒绝解锁。原轮工作区改绑不改写新范围；并发拒绝 CAS 仅一人成功；迟到拒绝不覆盖新轮 running 状态。 |
| 3.1–3.2 | 请求结束并切换环境应用/用户后真实入队，后台仍写原范围；原许可与最新配置取交集；只整理当前轮，不回填此前关闭沉淀的轮次；归属变化、成员移除和关闭沉淀均跳过；关闭沉淀仍可在前台写记忆。工作区已存在用户文件的 edit_file 和全新非受管文件写入均受阻，受管摘要可更新；结构化事实确认既有测试通过。 |
| 4.1–4.2 | 两组六行及独立沉淀说明；1440/390 两宽度、明暗主题共四组合；宽屏列对齐、窄屏纵排且无溢出；六控件名称和描述关联、键盘空格切换及可见焦点；关闭时说明仍可见；500 保存失败保留输入，成功保存后刷新和切 Agent 回显。 |
| 4.3 | 实际管理列表、过滤、详情和删除识别新范围；两应用同名文件以完整 namespace/key 区分；跨租户详情和删除拒绝；删除 A 不影响 B。前端 owner 类型与 ID 选择不互相覆盖。 |
| 5.1 | 上述真实存储、管理回显、个人/工作区、共享、隔离、恢复和后台场景联合通过。 |

## 复跑命令

在 `dataclaw/apps/agent-os/service`，预先通过安全环境配置设置 `DATABASE_URL`：

```sh
AGENT_MEMORY_TEST_DATABASE_URL="$DATABASE_URL" pnpm exec jest \
  agents/agents.service.spec.ts \
  agents/agent-memory-config-upgrade.integration.spec.ts \
  tenant-initialization/agent-templates.spec.ts \
  workspaces/agent-memory-scopes.spec.ts \
  workspaces/agent-memory-runtime.integration.spec.ts \
  workspaces/conversation-deepagents-chat.service.spec.ts \
  workspaces/workspace-memory-fact.tools.spec.ts \
  memory/memory.service.spec.ts --runInBand
pnpm exec tsc --noEmit --pretty false
node ../../../scripts/verification/agent-memory-scopes-ui.cjs
```

在 `dataclaw/apps/agent-os/web`：

```sh
pnpm exec vue-tsc --noEmit --pretty false
pnpm exec vitest run src/views/memory-view-filters.test.ts
```

运行时集成文件的七个 `it` 已逐一以 `--runInBand -t '<完整用例名称>'` 独立执行，以验证 fixture 独立性。浏览器截图输出到 `/tmp/memory-scope-visual/`。

## 最终结果

- 服务端：8 套、133 个测试通过，包含真实数据库升级与 7 项运行时联合用例。
- Web：3 个过滤测试通过；服务端和 Web 类型检查通过。
- Chrome：四种宽度/主题组合及键盘、描述关联、失败保存保留、刷新和 Agent 切换通过。
- OpenSpec：`openspec validate agent-memory-scopes --strict --json` 通过；apply 为 `all_done`，12/12 完成。

升级配置重置在新版服务接受请求前执行，版本记录与重置在同一事务提交。旧 `users` 命名空间内容保留，不推测租户归属或回写旧路径。本次尚未归档变更。
