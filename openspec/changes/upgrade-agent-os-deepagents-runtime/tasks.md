# Tasks

## 1. 服务端依赖与范围隔离

- [x] 1.1 在当前锁文件下从 Desktop 实际导入位置记录 deepagents、langchain、core、LangGraph、模型适配器的解析版本，另记录 KMS 各 workspace 的锁定依赖闭包，形成可比较的基线。
- [x] 1.2 按 design.md 的九项版本更新 agent-os-service 依赖和 pnpm-lock.yaml，用冻结锁文件安装，从服务和 deepagents 两个位置验证实际加载版本及 peer 约束。
- [x] 1.3 比较 Desktop 解析结果和 KMS 依赖闭包与基线，确认两者无版本漂移、无源文件和 manifest 变更。

## 2. PostgreSQL 新库初始化

- [x] 2.1 对照 checkpoint-postgres 1.1.4 同步部署用全量 DDL、检查点迁移登记和 postcheck；在隔离空库按仓库正式初始化顺序执行，核对配置 schema、task_path 列和迁移版本。
- [x] 2.2 校准 DeepagentsPostgresMemoryService 的启动检查与连接生命周期，验证缺少必需列能被识别，在有效新库上用实际 saver/store 完成 checkpoint 写读、增量重建和 store 写读。

## 3. 对话、工具与审批恢复

- [x] 3.1 适配聊天和记忆归并的 createDeepAgent 调用；在主对话显式启用计划工具。验证授权工具、Skills、子 Agent、文件访问规则及记忆归并冒烟流程，运行 agent-os-service 的 check-types 和 build。
- [x] 3.2 验证 runtimeThreadId 对应的增量状态恢复路径，按需适配读取逻辑；以可计数工具验证批准一次、重复或并发提交同一批准仍只执行一次、拒绝执行零次，并运行会话与审批相关 Jest 用例。
- [x] 3.3 核对 stream 到 UI 消息的映射、子图归属及工具失败语义，用包含 write_todos、子 Agent 和工具调用的流验证进度可见、最终回复无重复、同轮同路径的写入冲突可见。

## 4. Web 消费端验证

- [ ] 4.1 在 dataclaw-web 的 ChatPage 与 useConversationRuntime 验证实时文本、计划进度、审批与最终结果，按需适配，运行相关 Vitest 用例及 build。
- [ ] 4.2 在 agent-os-web 的会话任务详情核对同一会话的状态、消息及工具结果与聊天端一致，按需适配并运行 agent-os-web 的 check-types 和 build。

## 5. 空库端到端验收

- [ ] 5.1 按部署顺序在隔离环境使用新版制品与正式 SQL 初始化空库。创建多轮会话，在工具暂停时重启服务，从 Web 批准并续聊；核对原线程、历史顺序、审批计数和两个 Web 的最终记录。
- [ ] 5.2 汇总依赖树前后对比、空库 postcheck、构建和测试结果，并记录端到端会话与工具调用标识，使验收结果可复核。
