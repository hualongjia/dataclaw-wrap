# 实施与验收记录

## 当前结论

实现已覆盖 Agent OS 任务归属、个人访问范围、工作空间引用、个人聊天工具、正文依赖删除生命周期、P3Demo 11 个转发接口和智能问答任务入口。服务器可信通知配置按用户选择放在部署配置中。

代码、单元测试和本地数据库回归不替代真实身份与页面验收。当前没有已登录的第三方会话，浏览器打开 P3Demo 后停在统一认证登录页；未输入、提取或猜测任何凭据。涉及真实身份、授权 Agent 和页面全流程的复合任务保持未勾选。

## 已有证据

截至当前：dataclaw 代码与接口文档 HEAD `ab4a0de2`，P3Demo HEAD `6402567`。第 5 轮后 Service 类型检查、55 项任务/通知定向单测、正文截止单测通过；显式启用的 fixture 数据库回归 10 项通过。DataClaw Web 通知目标单测及 `vue-tsc` 通过，P3Demo 完整 `pnpm verify` 通过。后续静态评审修复以新增提交补充本记录。

- Prisma Client 生成、Prisma validate、Service `tsc --noEmit` 通过。
- 新库：从 canonical DDL 抽取 `scheduled_task` 的 CREATE TABLE 与个人范围索引，在临时表执行后检查可空 BIGINT 列与索引。
- 升级：`database/tests/scheduled_task_application_migration.sql` 在临时表连续执行迁移两次，检查类型、可空性、索引、原计划时间。
- 开发库应用新增列迁移后，首次回填命令连续两次均 `updated=0/unresolved=0`。当时没有任务；后续只读核对为 0 任务、0 执行，未发现非 fixture 数据受影响。
- 服务定向测试覆盖应用范围、父任务先授权、原凭据身份、空间及有效绑定关系、无空间聊天工具、正文依赖、删除/编辑锁顺序、启动握手与迟到取消。
- 数据库回归显式启用并限定雪花 fixture ID，校验两实例仅一次领取、旧触发不能执行未来计划、延后/暂停使旧候选失效、失效凭据退出到期队列、结果回写保留暂停、回填已删依赖与终态、SQL NULL 和损坏依赖隔离，以及启动/删除串行、推理期间释放锁、领取后配置变更取消旧触发。测试后清理 fixture。
- 测试回填通过内部可选 fixture ID 集合限制查询，并在写前再次确认集合成员；凭据验证桩仅识别 fixture 租户和每条 fixture 的唯一合成 Key。此筛选参数没有 Controller/HTTP 暴露。
- P3Demo 完整 `pnpm verify` 通过，包含格式、Lint、类型、Service/Web/shared 测试和构建。定向回归覆盖 11 个上游方法路径及当前用户/App-Key 注入、长操作等待、`"0"` 归一、长输出读取、统计按需加载、暂停后编辑、custom 来源解除依赖和执行父任务状态清理。
- `SCHEDULED_TASK_FRONTEND_ROUTES_JSON` 无映射返回空链接，不从请求头回退。Webhook 仅允许当前租户/应用登记的完整 URL，禁止重定向，错误不回显外部正文。

## 可重复的本地验证命令

在 `dataclaw/apps/agent-os/service`：

```sh
pnpm exec prisma validate
pnpm exec tsc --noEmit
pnpm exec jest scheduled-tasks scheduled-task-delivery-targets create-scheduled-task scheduled-task-locking scheduled-task-start-boundary scheduled-task-conversation-deletion conversation-application-scope --runInBand
RUN_SCHEDULED_TASK_DATABASE_TESTS=1 node --env-file=.env node_modules/jest/bin/jest.js scheduled-tasks.dispatch.integration --runInBand
```

数据库回归使用本地开发连接及自动生成的 fixture。普通 Jest 运行默认跳过数据库用例；显式命令启用它。生产全量回填命令是 `pnpm scheduled-tasks:backfill-applications`，部署时先执行迁移并核对 `updated/unresolved`。

在 `p3demo-wrap/p3demo`：

```sh
pnpm verify
```

## 真实身份验收步骤（未执行）

使用合法登录与各应用自己的 App-Key，准备同租户用户甲、乙，应用 A（DataClaw Web）、B（P3Demo），以及另一租户用户。不得用数据库伪造登录或绕过 Guard。

1. **写入身份与复制（2.1、2.2）**：甲从 A 创建任务，在请求中加入伪造 creator/application 字段，核对数据库实际写入可信身份；缺 App-Key 请求失败且无新增。管理端用匹配原应用身份复制，副本独立 ID；跨应用复制失败。确认 Web templates/copy 路由不能取得数据或产生副本。编辑/更新凭据不改变原归属。对匹配、冲突和空归属旧任务核对派发与 Agent 调用。
2. **真实聊天工具（2.3、2.5）**：使用已授权 `create_scheduled_task` 的 Agent，在个人会话创建未来时间提醒，分别省略空间与使用运行时 `"0"`，核对返回 ID 可在本人当前应用列表定位、空间为空、来源为当前会话。交叉引用乙、另一应用及另一租户会话均拒绝；DataClaw 合法空间会话可创建。
3. **空间权限（2.4、2.5）**：在 P3Demo 的页面、直接 API、聊天工具中测试显式空间、空间来源、workspace 知识范围及会话间接绑定空间，均须在写入前拒绝。DataClaw 测试同应用所有者/成员成功，非成员、同租户其他应用空间（含本人所有）、跨租户失败。撤销成员后，手动与到期执行在正文读取和 Agent 启动前停止。
4. **来源生命周期（2.6）**：同一真实会话创建完整独立提醒和显式正文依赖周期任务，先取得执行历史，再删除来源。两任务及历史保留；独立提醒继续，依赖任务暂停并显示原因。重放旧到期触发、手动运行及恢复不能绕过；编辑为 `mode=user` 后仍暂停，显式恢复才按新计划调度。
5. **三维隔离（3.1、3.2、3.3）**：甲在 A/B 各建任务、乙在 A 建任务；甲在 A 的列表、分页、搜索、状态汇总及空间筛选只统计甲在 A 的记录。逐一用他人/本人其他应用/其他租户 ID 请求全部读写入口，核对无任务变化、执行、通知或越权同步。管理端有 `agentos:job:view` 时可查本租户全部用户和应用，无权限及跨租户拒绝。
6. **P3Demo 全流程（3.4、3.5、3.7）**：从智能问答“定时任务”入口创建个人周期任务，按返回 ID 定位并查看配置、状态和最近结果；暂停后编辑仍暂停，恢复按新时间与时区计算；立即运行产生 manual 记录且保留自动计划。再核对五种写操作的跨用户/跨应用拒绝，最终删除测试任务，列表、详情与历史均不可见。
7. **执行与统计（3.6）**：准备超过一页历史，验证翻页、excludeExecutionId、指定执行定位与父子 ID 不匹配拒绝。打开统计区域才发统计请求，核对运行次数、失败次数、Token 与预算。无权限请求在状态同步前被拒绝。
8. **第三方接口和通知（3.8）**：逐项保存 11 个接口的实际请求/响应及错误（脱敏），确认 P3Demo 转发方法、路径、用户及应用头。登记本应用可信前端 origin/路由后点击通知定位任务及执行；未登记不附外链。登记测试 Webhook，核对成功、重定向拒绝和错误脱敏。按实际结果更新主题和 README，不将源码/Mock 测试标为真实联调。

## 任务跟踪说明

1.1、1.2、2.4 已按现有证据勾选。其余条目含更广矩阵或真实身份联合验收，即使实现代码已经存在也保持待验收。完成上述步骤后按每条任务的完整要求更新 checkbox，不能直接全部勾选。
