# 实施与验证记录

## 范围与提交

用户明确授权本变更修改、提交独立 P3Demo 项目，并批准 Agent OS 第三方身份策略精确开放三个文件 GET 路由。没有部署、修改存量应用配置、提取真实登录令牌或执行真实文件生成/删除/移动。

- 外层规划：`1a87a61`，同步三条只读路由、安全边界及实施目录授权。
- DataClaw：`21f97b5e`，精确路由接入与三入口资源范围回归；`96401418`，第三方接口文档和索引。
- P3Demo：`b0cd859`，共享契约及产物树/预览/下载；`5187f69`，右侧抽屉、回答定位与预览下载及回归。

任务 1.1–2.3 已实现并本地验证；3.1、3.2 保持未勾选。后两项需要已部署新代码、P3Demo 真实登录会话及允许生成、覆盖、移动、删除验收文件的环境，不能用测试替身代替。

## 已执行验证

P3Demo 根目录 `pnpm verify` 通过：格式、全工作区 ESLint、TypeScript、测试和生产构建。Service 119 套/1167 项通过，1 项独立环境测试跳过；Web 115 套/688 项通过。产物组件 15 项行为测试与既有问答页 28 项通过。测试覆盖混合树、空树、嵌套/空目录、节点归属拒绝、截断和二进制文件名、完整路径定位、相对及带前导斜杠引用、显式 Markdown 链接、完成状态、单次刷新、旧预览清除、覆盖内容重读、迟到树/预览隔离、HTML sandbox 和图片 Blob 地址释放。

DataClaw 受影响路由策略及会话范围测试 82 项通过，服务 `tsc --noEmit` 和三个修改文件 ESLint 通过。全服务 Jest 运行结果为 153 套/1558 项通过、12 套/131 项跳过，另有 3 项失败，位于本次未修改的文件：

- `workspaces/scheduled-task-locking.spec.ts`：解除依赖测试传入的时区被现有 Asia/Shanghai 校验拒绝。
- `tools/database-operation-plan-migration.spec.ts`、`tools/dataset-discovery-migration.spec.ts`：基线表数断言 161，迁移后置检查记录为 160。

未为通过本变更检查而删除、跳过或改动这些测试及相关业务逻辑。全服务不能声明全部通过。

OpenSpec `validate --strict` 通过，任务完成后逐项刷新 apply 状态为 6/8。

## 数据流与安全控制

```text
登录浏览器 → P3Demo 统一 HTTP 客户端 → AuthGuard/参数 Zod → 业务产物服务
  → 当前用户令牌 + 后端 App-Key → Agent OS 精确 GET 策略
  → 租户/应用/用户/个人会话归属 → 会话文件索引 → 预览或原字节
```

浏览器到业务服务沿用会话、CSRF、预期租户与身份批次机制，服务间沿用配置地址、受控 HTTP 客户端和请求超时。未修改认证令牌生命周期或部署传输配置。

| 资产/威胁 | 控制与验证 |
| --- | --- |
| 文件目录、内容可能含业务敏感数据，跨用户/租户/应用枚举 | 服务端会话授权先于索引读取；三个入口真实服务方法越权矩阵拒绝（AZ-04、AZ-06、COD-30） |
| 通过节点 ID 读取其他会话或 temp/uploads 文件 | 每次从受信当前会话树筛选根 generated 子树，仅文件可预览/下载；测试拒绝目录外、其他节点与目录节点（COD-21、COD-50） |
| 文件路径穿越或同名误定位 | 不折叠 `..`、拒绝空段/反斜杠/控制字符，按完整路径精确匹配，不按 basename 回退（COD-21） |
| HTML/Markdown 内容注入 | HTML 空 sandbox、禁脚本和外部资源 CSP；Markdown 通过现有渲染器，自定义本地链接，不执行文件（COD-10、COD-11） |
| 会话切换后旧内容串用 | 按会话 key 卸载、请求代次检查、Blob URL 回收，不持久化预览（DAT-23、DAT-24） |
| 下载变成公开链接或正文进日志 | 用户显式选择单文件后经统一鉴权客户端请求，后端 no-store/nosniff，凭据不进 URL；没有新增批量导出。审计目录沿用项目已记录的用户“不记录 AI 操作（含文件）”策略，不修改策略配置 |

范围为个人会话产物读取，不新增工具执行、知识记忆、模型 Prompt、数据库写入或文件修改能力。新代码不增加依赖、不改变锁文件。独立静态评审由主代理另行安排，本记录不代替评审或上线验收。

## 扫描结果与限制

- 对两仓本次提交新增行执行本地模式检查：私钥块、AWS access ID、GitHub token、JWT、含口令数据库/Redis URL，均 0 命中。该检查范围有限，未安装/运行 Gitleaks，不等于专用秘密扫描器的完整检查。
- 静态分析执行项目 ESLint 和 TypeScript，结果见上述验证；未运行 Semgrep/CodeQL。
- 默认镜像的依赖 audit endpoint 不存在；改用 `pnpm audit --prod --json --registry=https://registry.npmjs.org` 完成只读扫描，报告现有依赖 44 high、1 critical、45 moderate、11 low。critical 为 `proxy-addr` 的 GHSA-jqcg-44mw-7w3h，其他高危涉及 multer、undici、zod、MCP SDK 等。原始报告保存在执行机器 `/tmp/p3-artifact-dependency-audit-official.json`。

本次没有改变 package 清单或锁文件，未擅自升级现存依赖。项目安全基线 `references/security-review-checklist.md` REV-18 规定未缓解高危/严重漏洞不得上线，例外需双人审批与到期时间，因此不能声明发布安全门禁已通过。本变更尚未上线，依赖处置或正式例外由用户和责任角色决定。
