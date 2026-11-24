# Proposal

## Why

DataClaw 已迁移到 Auth3，但 Jenkins DEV 运行时配置仍保留旧认证参数，缺少浏览器会话及机器客户端配置，无法可靠部署当前代码。需要将三个既有服务对齐 DEV 应用登记和当前配置契约。

## What Changes

- 对齐 agent-os-service 的 DEV Auth3 管理端、用户端凭证、登录与退出回调、会话、Redis、Cookie 和 CORS 配置。
- 清理两个前端未使用的运行时 KEY，补齐管理端认证站点地址，统一使用 dataclaw-admin-dev 作为管理与 Base 归属编码。
- 核对 ConfigMap 更新方式、源码分支和配置生效检查，提供可执行的 Jenkins 发布说明。
- 保留服务名、现有部署链路及非认证业务配置；不修改 demo、fujian，不触发 Jenkins 或集群发布，不迁移历史数据。

## Capabilities

### New Capabilities

- `dev-auth3-deployment`: 三个 DEV 服务的认证配置一致性、配置发布与验收契约。

### Modified Capabilities

无。既有 client-authentication 行为保持不变，本次补齐其 DEV 部署配置。

## Impact

规划保存在 dataclaw-wrap；实施目标为 /Users/hualong/jxworkspace/universal-deployment-wrap/universal-deployment 独立仓库中的 runtimes/dev/agent-os-service/configmap/.env、runtimes/dev/agent-os-webapp/runtime-args.json、runtimes/dev/dataclaw-webapp/runtime-args.json，以及确有必要的对应发布脚本。外层 docs/services 可补充服务部署说明，不包含凭据。

Auth3 参数依据 docs/dataclaw-admin-auth3-info.md 和 docs/dataclaw-web-auth3-info.md 的 DEV 段落。Redis 和会话密钥须使用部署环境配置，不复制本机密钥。桌面认证保持现状，不在本次迁移范围内。
