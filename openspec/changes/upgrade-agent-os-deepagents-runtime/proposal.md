# Proposal

## Why

DataClaw 的服务端对话由 Deep Agents 创建，当前锁定的 1.10.2 在每一步保存完整消息历史。本次部署后将重建数据库，适合在首条正式会话前采用 1.17.1 的增量消息检查点，并验证工具审批和 Web 进度展示。

## What Changes

- 将服务端 Deep Agents 升至 1.17.1，联动更新 LangChain、LangGraph、模型适配器、持久化和追踪依赖。
- 为新库提供匹配新版 PostgreSQL saver/store 的初始化结构和检查，支持增量会话跨重启续聊及审批。
- 在对话 Agent 中保留任务计划工具、子 Agent、流式输出和既有工具权限；通过两个 Web 的既有入口验收。
- 共享锁文件更新限于此次服务端依赖闭包，核对 Desktop 与 KMS 的实际解析结果。

## Capabilities

### New Capabilities

- `agent-conversation-runtime` 涵盖增量对话存储、重启后继续执行、工具审批与 Web 端任务展示。

### Modified Capabilities

## Impact

应用范围为 `apps/agent-os/service`、`apps/agent-os/web`、`apps/dataclaw/web`。配套文件包括 DataClaw 工作区锁文件、服务端新库初始化 SQL 及 postcheck。Desktop 与 KMS 仅做依赖隔离核查。部署按“新版就绪 → 重建并初始化数据库 → 验收 → 开放流量”进行；旧数据迁移及回滚方案在本次范围之外。
