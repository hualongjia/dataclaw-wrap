# 实施与验收记录

## 当前结论

实现已覆盖 Agent OS 任务归属、个人访问范围、工作空间引用、个人聊天工具、正文依赖删除生命周期、P3Demo 11 个转发接口和智能问答任务入口。服务器可信通知配置按用户选择放在部署配置中。

代码、单元测试和本地数据库回归不替代真实身份与页面验收。用户已正常登录P3Demo与本地管理端，并完成本文末尾列明的真实页面、模型、工具审批和来源生命周期验收。未完成的双用户/跨租户、子Agent部署及完整接口矩阵保持未勾选，不能将单一身份实测扩大为全部验收。

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

## 真实身份验收矩阵（完成范围见末尾实测记录）

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

1.1、1.2、2.4、2.6、3.7、4.1–4.3 已按现有证据勾选（8/20）。其余条目含更广矩阵或联合验收，即使实现代码已经存在也保持待验收。完成上述步骤后按每条任务的完整要求更新 checkbox，不能直接全部勾选。

## 追加真实用量验证（2026-11-27）

用户授权新增运行时/SSE 用量链路，使用供应商原始 total_tokens，不增加金额计费。服务提交 6866b064（采集）、2f80995e（流/故障隔离）、3f1c5f00（迁移/ledger/API）、170fedcb（终态恢复/覆盖缺口），P3Demo 64e7c1a/40d2b01（schema/展示），DataClaw 89410063，postcheck 47d44c99。

独立证据：parser 12 项；真实 PostgreSQL ledger 3 项和派发 10 项通过，包含多调用/幂等/故障恢复/终态重启未知覆盖/已知下限 exceeded。迁移 000002 重跑、000003 重跑、000004 升级及旧零值未知通过；postcheck 完整结构通过、事务内移除 usage_accounted_tokens 被拒并回滚。P3Demo 完整 pnpm verify 通过，DataClaw vue-tsc 通过。真实身份和供应商调用验收尚未完成，以上 fixture 不冒充真实模型实测。

待真实身份补验：登录 P3Demo 后创建允许模型执行的个人任务，触发多轮/子 Agent，保存供应商原始 usage 和 executionId；核对每个物理请求只计一次、总量等于供应商总和、预算同 Token 差额增加；分别检查 P3Demo 和 DataClaw 的真实零/未知/下限显示。按原有 2.x/3.x 的多身份、来源删除及页面全流程清单继续，未完成任务保持未勾。

第 9 轮后补充：服务 098ca546 在发布执行终态之前结账，running 失主回收使用未知覆盖恢复语义；个人范围读取不处理全局内存补偿队列。ledger 新增健康执行与两个服务实例并发读取统计场景，ledger 4 项 + 派发 10 项通过（连同 parser 12 项共 26 项）。postcheck cccc1459 精确核验默认表达式：完整结构通过，移除 usage_accounted_tokens、将默认改为 10、给 usage_finalized_at 加 CURRENT_TIMESTAMP 三种反例均拒绝并事务回滚。服务 tsc 与 DataClaw vue-tsc 均通过。所有真实身份验收状态不变。

第 10 轮提交：605a9097（000005 恢复标记迁移、双 canonical/postcheck），79aef8ba（租约、条件回写、完整观察纠正推断未知、迟到回调补偿、请求ID冲突重分配），cafb6e19（升级/租约文档）。真实 DB 用量 8 项 + 派发 11 项通过：seal 首次短错→异实例恢复→owner 重试可恢复 known；实际缺 usage 保持未知；seal 后迟到写入失败重新补偿；有效心跳长任务不回收、过期回收；查询后完成不被旧回收快照覆盖；跨执行 Snowflake 碰撞不改其他明细；配置变更和删除取消两生产分支确认未发送且为真实零。parser 12 项、启动截止 1 项通过，tsc 通过。升级链增加 000005 后通过，postcheck 完整及此前三反例通过。未再次运行没有变动的 P3/DC 全量验证，沿用上一轮结果；真实身份验收仍未完成。

第 11 轮提交：08f1a201（批准续跑在异步准备/批准后检查取消信号并传给 Agent stream），9ce1ab2b（所有结束和迟到路径共用 revision 复核的全量观察 flush，稳定全量成功才清推断标记；补偿按入队代次确认，软删存在执行可结算，物理删除清理，活跃执行暂缓）。新增三个批准续跑行为测试；chat suite 55 项通过；真实 DB ledger 10 项通过，含 A 未登记且全量重放失败时 B 单条成功仍未知、恢复后完整计数，以及软删/物理删除/并发补偿重入。scheduled 模块 unit 68 项通过，tsc 通过。P3/DC 未变，不重复运行其全量检查；真实身份验收状态不变。

第 12 轮提交 0ab1b941：每个观察共享登记 in-flight Promise，失败释放以供重试；每事务固定候选 ID，核对原执行/租户后只发布一次最终 ID。真实 DB ledger 11 项通过；新增未登记观察的两次 finish 与两次 seal 并发，并强制跨执行雪花 ID 冲突，登记等待时仅一事务，最终一条明细、预算只累计一次、其他执行不变。tsc 通过；未变模块沿用先前验证，真实身份验收仍未完成。

第 13 轮提交 1729b073：每次登记事务前保留当前候选 ID，事务返回不确定时重试同一候选，避免雪花冲突重分配后的已提交行成为孤儿。真实 DB ledger 12 项及 tsc 通过；新增原冲突 A→实际提交 B 后向调用者抛错→重试仍使用 B，最终仅一条明细且真实用量/预算 47。真实身份验收状态不变。

第 14 轮静态复核：由 gpt-6.2-astra high 只读审查服务 1729b073，确认第 13 轮不确定提交问题关闭，未发现新的可行动问题；静态评审在第 14/15 轮结束。该结论不代替真实联调，任务仍为 6/20。真实身份待用户完成 P3Demo 登录并提供可用的应用/用户/Agent 工具授权后，继续本文已列的双用户/双应用、来源删除、实际模型 usage 与完整页面验收；当前没有这些实测证据，相关任务保持未勾选。本轮无实现变更，不重复运行已通过检查。


## 真实本地联调（2026-11-27 晚）

用户已完成真实登录，并授权复用正常管理端已登记的 p3demo-dev、临时本地上游及最小工具授权。管理端真实 hualong 会话可管理应用及 Agent；复用已发布且仅个人会话的 p3demo-dev，默认 P3Demo Agent（383909762327302144）与平台托管 deepseek-v4-flash。原 Agent 仅授权 get_weather；通过正常工具授权页增加 create_scheduled_task，其他权限未改。Key由管理UI复制后按用户明确许可使用pbpaste直接写0600本地env并清剪贴板，未提交秘密。原env备份 /tmp/scheduled-task-p3-env-backup，P3上游临时为127.0.0.1:5000。

P3真实任务384703673812234240已完成11操作页面验收：创建、列表/详情、暂停、暂停后编辑名称与UTC时区仍暂停、恢复后下次运行变为北京时间17:00（UTC09:00）、立即运行确认、历史、单次执行、按需统计、删除。执行384703877978370048为manual/completed，输出“个人任务执行成功”，历史1条；最近/7天/30天均9288 Token、预算已用9288。原始模型明细一条known9288、无恢复标记，执行及任务预算一致。删除后列表0，再按原ID定位显示不存在，DB任务软删。按此完整证据勾选3.7，其余包含更广隔离/多身份/多轮子Agent的任务不因此勾选。

第一次run-now成功响应曾被前端身份验证版本保护丢弃（未自动重放），重新定位安全读取了已完成结果；未削弱租户/身份隔离。真实SSE工具调用发现data-agent-step元数据被P3当未知事件中断，修复6cf7bfb后标准tool输入/审批/结果可用，P3完整verify通过；仅忽略该已知步骤上下文，其他未知类型仍拒绝。


### 来源生命周期实测完成与清理

真实会话384705603829608448中逐次批准create_scheduled_task，A384707278113570816与B384707464688795648均sourceType=conversation、sourceId为该会话、workspace为空，知识范围分别为user及conversationIds=[该会话]。来源强绑定修复c7dbe474/df30ab0b防止模型以custom混淆来源和正文依赖或伪造来源ID，工具6项回归通过。最小PII修复aa2e690e仅保护可信create_scheduled_task结构化scheduledTaskId，其他字段/自然语言数字仍按原规则处理，chat与定向58项/tsc通过；此修复未冒充额外实际模型验收。

A删除前执行384707764698972160 completed/known10821；B删除前384708192652197888 completed/known11337。P3各历史均显示1条手动completed。正常侧栏删除来源后，DB会话ifDelete=1；B详情已暂停/来源会话已删除，手动运行和直接恢复400拒绝，没有新增执行；A保持active和原计划，删除后再手动执行384708974415937536 completed/known10710，页面保留2条成功历史。B取消正文依赖并改完整提醒后保存仍paused，显式恢复才active。旧排队触发不能绕过及删除/派发竞态沿用先前DB回归，本次未伪称真实队列重放或等待次日自动触发。按组合证据完成2.6。

DataClaw Web同一真实用户本应用列表为0，直接访问P3活动任务384705764999933952显示任务不存在；只证明本人跨应用读取拒绝，不替代其他用户或租户矩阵。P3当前Agent子Agent绑定0，运行时generalPurposeAgent=false，实际多轮/子Agent完整用量对账4.4保持未完成。单次实际模型原始usage汇总已验证，不当作完整4.4。

全部4个本次验收任务（384703673812234240、384705764999933952、384707278113570816、384707464688795648）均已通过正常P3删除并只读核对ifDelete=1；刷新页面显示暂无个人任务/总数0，无到期验收任务遗留。没有删除原有用户任务。P3父仓库原有docs/auth3-app-branding-integration.md未改动。

### 临时环境与待补验条件

本地服务5000、管理Web5002、DataClaw Web5001与P3 3000/3001用于本次已授权联调；P3 env临时本地endpoint和Key不提交。原env备份/tmp/scheduled-task-p3-env-backup为0600；如恢复旧上游，应将该备份还原至P3的apps/service/.env实际目标（保持0600）并重启P3服务。剪贴板已清空。仅通过正常管理UI给P3Demo Agent追加create_scheduled_task授权，原get_weather保留；若撤销本次测试授权可在同Agent工具页取消create_scheduled_task。未新增应用、未伪造登录、未更改Auth3全局规则。

剩余真实验收需同租户第二用户与其他租户的合法登录、各应用/空间的授权测试数据，以及已发布并授权的测试子Agent。之后按上方矩阵完成跨用户/跨租户全部读写、管理权限正反例、历史多页与执行父子错配、P3空间引用负例及DataClaw空间正例、实际子Agent多轮usage/预算/两端页面核对及通知配置联合验收。当前不创建或冒用这些身份，不将本次8/20标为全部完成。

第15轮同轮修复：2e208461将可信业务sourceConversationId与推理conversationId/thread分离，仅经过会话授权的消息/审批入口显式注入来源；runtime默认null，定时执行与eval临时线程不冒充业务会话，非聊天执行可创建独立custom后续任务。78607655在两条审批续流中用服务端expectedApprovalActions初始化工具名，标准output没有重放input也保留仅可信scheduledTaskId，其他PII仍脱敏。tsc与86项定向测试通过，覆盖真实会话/定时/eval线程来源、定时首轮及审批上下文、无input重放的真实PII规则输出。这两修复为回归证据，不冒充新增真实模型实测；保留8/20完成状态，待本轮静态复核闭环。

第15轮最终闭环：784a6081补齐两处已授权聊天自动续写的sourceConversationId，并拒绝非聊天显式null上下文中模型伪造conversation/sourceId；定时及eval保持独立来源语义。tsc及79项定向回归通过。已用rg核对全部生产初始/续写/审批/定时/eval入口。gpt-6.2-astra high在本次第15/15轮只读静态复核DataClaw784a6081与P3Demo6cf7bfb，确认两项P2均关闭，未发现其他可行动问题，评审在15轮结束，不开启第16轮。此次仅补记录，不改实现或重复运行测试。

最终跟踪为8/20；剩余未勾项包含真实同租户第二用户、其他租户身份及各应用授权测试数据矩阵、管理员权限正反例、P3空间负例/DataClaw空间正例、多页历史/父子执行错配、通知实际投递以及已发布并授权子Agent的完整多轮usage/预算/两端页面对账。已完成的本人P3页面生命周期、正常工具审批创建、同来源删除/解除恢复与本人跨应用读取拒绝不等同于这些未完成矩阵。当前P3 Agent无子Agent绑定且关闭通用子Agent；没有为补验擅自建立新身份或启用其他停用Agent。

最终代码：DataClaw784a6081、P3Demo6cf7bfb。P3本地临时env原文备份仍为/tmp/scheduled-task-p3-env-backup（0600，无秘密入git）；还原目标为/Users/hualong/jxworkspace/p3demo-wrap/p3demo/apps/service/.env所指向的实际文件/Users/hualong/jxworkspace/p3demo-wrap/p3demo.apps.service.env，保持0600并重启P3 Service才生效。本地授权测试环境保留供后续补验；本次全部4测试任务已软删，无活动验收计划遗留。


## 用户接受未验证后收尾（2026-11-27）

此前8/20反映当时保守的复合场景跟踪，并非其余实现未完成。用户现明确接受另一用户、另一租户身份及可运行子Agent才能完成的真实完整矩阵和用量对账为 **accepted-not-tested**。不冒用身份，不删规范，不把豁免计为测试通过。已据任务原文字重新核对证据：不要求真实联调的场景可使用unit/DB验证；明确真实联调的条目保留实际证据。

新增实测：DataClaw当前真实hualong用户在空间111（383891906466721792）创建任务384714547513491456，详情显示空间111/custom、每天09:00。暂停后管理端同租户列表、汇总与详情均可见相同状态；通过正常管理端删除后列表/汇总0。P3个人真实会话384714989152731136批准create_scheduled_task携带workspaceId="1"，生产validateTaskReferences报“来源会话不可访问”，拒绝个人来源附加空间且没有写任务。首次长数字ID被既有PII掩码后仅报numeric格式错，不算空间权限实测。P3显示output-denied/受控流错误，权限结论来自对应生产工具日志和无写入只读核对，不夸称页面完整友好错误展示；未扩展通用PII或SSE范围。

全部5个验收任务（此前4个及新空间任务）均正常删除，只读核对ifDelete=1；新增负例会话也正常删除且ifDelete=1，无活动验收计划。没有外发测试邮件或Webhook消息。

### 最终逐项证据映射

|任务|已完成证据|接受未验证/限制|
|---|---|---|
|1.1–1.2|迁移/postcheck及应用回填真实DB重跑、有效/冲突/NULL/损坏样本，见前文提交链|无新增豁免|
|2.1|23a865c0生产创建写入分支忽略伪造creator/application/tenant；管理员复制新ID与当前可信归属、跨应用拒绝；原凭据归属/creator回归；Web路由只有11接口，templates/copy不在业务控制器|复制为服务回归，不是管理UI实测（UI无复制入口）|
|2.2|application-scope匹配/冲突/空归属、原凭据校验；真实DB失效候选暂停及领取无越权派发|无须额外真实身份|
|2.3|真实授权聊天A/B来源当前会话、空间空、列表定位；tool unit覆盖缺省/0及可信source；references覆盖来源三维归属和有效空间关系|跨用户/跨租户真实矩阵accepted-not-tested；已有自动化不丢弃|
|2.4|references与事务/撤成员/启动删除串行DB回归，前文已完成|无新增豁免|
|2.5|P3真实个人来源携空间拒绝；P3写schema及服务references覆盖显式/知识/来源/间接空间；DataClaw真实空间任务创建成功|并未声称所有空间组合均真实HTTP实测|
|2.6|真实同来源A/B、历史保留、删源A继续/B暂停、解除后仍paused再显式恢复；旧触发采用DB回归|不声称真实旧队列重放|
|3.1|scope helper固定tenant/app/creator，list测试及过滤where/汇总共用作用域；真实本人跨应用列表0与管理汇总核对|跨用户/跨租户真实完整矩阵accepted-not-tested|
|3.2|四读父授权在同步前拒绝；23a865c0五写拒绝无事务/派发；缺可信应用与Auth3/App-Key边界回归；本人跨应用真实404|跨用户/跨租户真实完整矩阵accepted-not-tested|
|3.3|scope管理仅本租户；PermissionGuard授权/拒绝回归；真实管理列表/详情/删除；后台原creator与凭据的回归/DB派发|其他真实身份/租户矩阵accepted-not-tested；未改真实用户角色来制造无权限身份|
|3.4|P3真实11操作生命周期；Service逐项方法路径和当前用户/App-Key头回归；DC配置/状态及跨应用读取对照|无新增豁免|
|3.5|P3真实暂停编辑仍paused、UTC恢复新计划、manual执行、删除不可见；五写授权回归|同应用他人真实矩阵accepted-not-tested，本人跨应用五写为自动化作用域证据|
|3.6|实际P3历史/单次/按需统计及原始usage预算；新增page2/exclude及父子错配回归；父任务先授权无同步回归|真实跨身份完整矩阵accepted-not-tested；多页为定向回归，不制造大量模型执行|
|3.7|真实P3页面完整生命周期，见既有详细记录|无新增豁免|
|3.8|36eb4030更新11上游/转发文档和README状态；真实P3空间拒绝、DC空间正例、真实两类删源及相应接口链|通知目的地/深链/故障为已有自动化证据，不宣称真实外发投递|
|4.1–4.3|采集/parser、真实DB ledger/恢复/迁移及两端schema/UI检查，见前文；实际单模型usage与预算一致|无新增豁免|
|4.4|真实单模型已记录；多轮/子Agent完整实际对账未运行|accepted-not-tested，用户明确接受跳过，不是passed|

新增23a865c0仅测试：application-scope 14/14与tsc通过，覆盖伪造身份创建、复制和五写拒绝、历史分页/排除/父子错配。第15轮之后没有生产实现变化，仅测试/文档更新，不发起第16轮静态评审。P3未变沿用完整verify，DataClaw未变的生产模块沿用先前验证。

最终20/20追踪项已处理，**包含accepted-not-tested，不等于20项全部实测通过**。无需为了已接受跳过范围再索要身份或授权；将来若补验，可按上文真实矩阵继续。临时本地env/备份和create_scheduled_task工具授权按前述方式可还原，秘密未提交；保留用户原有P3父docs改动。


补充收尾检查：PermissionGuard、scheduled-tasks.references、scheduled-task-delivery-targets及notifications.application-links共4 suites/26项通过，覆盖管理员权限正反例、全部显式/间接空间引用策略、可信深链/无配置无外链及Webhook精确登记限制。没有外发真实通知，未将该自动化结果称为实测投递。用户豁免仅真实跨身份完整矩阵和真实多轮/子Agent对账，没有泛化为通知/空间等所有场景免验。

临时配置已还原：核对当前P3 env与原备份仅P3DEMO_AGENT_OS_BASE_URL、P3DEMO_AGENT_OS_APP_KEY两项不同，没有期间新增env改动；已恢复原备份字节并维持0600，P3 Service按原环境重启（日志/tmp/scheduled-task-p3-service-restored.log）。原备份/tmp/scheduled-task-p3-env-backup保留0600，未输出凭据。正常管理端create_scheduled_task授权保留，原get_weather未撤销。本次没有更改Auth3全局规则。P3父用户docs改动保留。此前“临时本地环境保留”描述为历史状态，以本条已还原为准。


## 用户追加：移除定时任务 Webhook（2026-11-27）

本次是前次15轮评审结束后的新增用户授权范围，**没有追加第16轮独立静态评审**，不宣称原评审覆盖新代码。按P3安全基线核对外部接入与数据/密钥边界：删除定时任务外发通道、默认拒绝废弃输入、保留授权/审计与正常结果查询；不涉及新网络调用、不迁移或删除历史数据。其他Skill/MCP的Webhook未改。

提交：DataClaw fbaf30ac（服务发送/签名/目的地配置移除、旧策略兼容、历史通知敏感字段清理）；0261328e（DataClaw创建/编辑/详情和Agent OS只读展示/类型删除、接入文档）；P3Demo bab9287（写入严格拒绝与读取兼容）。SCHEDULED_TASK_FRONTEND_ROUTES_JSON保留；Webhook目标环境变量已从turbo和service .env.example删除。部署仓库只静态检索，未发现该变量/定时任务Webhook配置，未改部署文件、未启动/构建/发布部署。

兼容行为：新请求选择webhook_notification/external_notification或在任意保留通道携带webhookUrl/webhookSecret（包括null）会拒绝。旧两通道经读模型归一detail_only、输出不含URL/secret，执行提前返回不fetch；编辑读取后的合法策略可用，管理复制将归一策略保存副本。数据库原JSON不破坏性改写。旧通知metadata删URL/secret、通道归一，历史外部投递错误不回显旧正文；执行模型没有deliveryPayload/deliveryResultJson专列，投递快照仅返回状态/时间和受控错误，保留其他渠道错误。结果仍在P3/DataClaw详情/历史，邮件、站内通知及可信前端链接保留。

验证：服务先红后绿，旧两通道读/执行不出网、废弃通道/字段拒绝、归一编辑合法、复制剔除旧字段；scheduled模块76项通过、23项真实DB测试按默认开关跳过（无DDL变化、不冒称重跑）；随后新增历史外部错误反例，service suite20项通过，通知链接/敏感字段回归通过。service tsc通过；DC vue-tsc与Agent OS Web check-types通过；P3 shared109项及完整pnpm verify通过（14/14构建任务成功，11缓存）。提交hook的ESLint/Prettier通过。首轮服务commit hook发现删除旧测试后的未使用import，已清理后正常提交，未绕过检查。

5.1–5.3按上述完成，不归入用户豁免。当前23/23为追踪项已处理，仍含原其他身份真实完整矩阵及实际多轮/子Agent对账的accepted-not-tested；不是23项全部真实联调通过。本次没有新增模型调用/外发测试通知或改动真实数据；本地env及其run timeout、备份路径、P3父用户docs保持原状态。上线需发布新服务/前端产物，本次未部署，既有运行实例不会因源码提交自动更新。
