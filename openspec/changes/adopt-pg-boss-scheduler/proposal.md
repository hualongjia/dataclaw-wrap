# PostgreSQL 多实例定时调度

## Why

agent-os-service 每个副本都扫描到期任务，进程内的执行标记无法协调多个副本。同一计划时间可能产生两次 Agent 调用和两条通知。

## What Changes

- 使用 pg-boss 和共享 PostgreSQL 持久化到期触发，允许任一可用副本领取作业。
- 保留现有五类计划、任务管理接口及执行历史，支持时区明确限定为 `Asia/Shanghai`，为同一次自动触发确定稳定的业务执行身份。拒绝非上海时区进入调度；历史异时区计划须先核对，不自动转换原计划时刻。
- 规定崩溃重试、外部调用结果未知时的处理，以及从旧轮询切换和回退的步骤。

## Capabilities

### New Capabilities

- `distributed-task-scheduling` 多副本部署中的计划触发、执行恢复和历史计划接管。

### Modified Capabilities

## Impact

涉及 agent-os-service 的 scheduled-tasks 模块、Prisma 迁移、PostgreSQL 队列对象和发布流程。与 `bind-scheduled-tasks-to-application` 配套，任务及执行历史继续使用原业务表，调度状态只依赖 PostgreSQL。
