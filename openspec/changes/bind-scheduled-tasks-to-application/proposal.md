## Why

P3Demo 与 DataClaw Web 共用定时任务服务。现有业务端按租户读取任务，用户能看到同事创建的任务及自己在其他应用创建的任务。

## What Changes

- 业务端创建任务、管理端复制任务时持久化经过验证的应用归属。
- **BREAKING** 业务端的列表、统计、详情、执行记录及操作限定为当前用户在本租户、当前应用创建的任务。Agent OS Web 的定时任务管理页供具备查看权限的用户查看租户全量任务。
- 两个应用共用 Agent OS 的创建、分页列表和详情接口，P3Demo 首期通过自己的后端向页面提供这三项能力。
- 下线业务端模板列表与复制任务路由。管理端沿用独立的复制接口。
- 根据可信的既有执行上下文回填历史任务；来源待确认的记录保留空归属，后台沿用原有执行校验。

## Capabilities

### New Capabilities

- `scheduled-task-application-scope` 定时任务的应用归属、访问和历史兼容。

### Modified Capabilities

## Impact

涉及 agent-os-service 的 Prisma 模型、数据库迁移、Web 定时任务 API 和后台执行身份校验。DataClaw Web 和 P3Demo 使用各自的 App-Key 调用现有接口。
