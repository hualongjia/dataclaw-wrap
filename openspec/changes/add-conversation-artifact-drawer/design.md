# 会话产物抽屉设计

## Context

Agent OS 的 `ConversationsController` 已提供会话文件树、节点内容和节点下载。`ToolFileIndexService` 返回会话根节点下的完整索引，列树时会安排后台同步。P3Demo 的 `AgentOsClient` 已封装列表和预览，供附件解析使用。面向抽屉的三个读取入口及下载转发仍需接入。已有消息模型的 `generatedArtifacts` 是可选的上游信息，抽屉还需列出从未被消息引用的文件。

## Goals / Non-Goals

在 P3Demo 的 React 智能问答页组合右侧抽屉，覆盖文件树、回答定位、预览与下载。页面、模型选择和个人会话访问规则沿用 `add-third-party-conversation-integration`。上传文件、工作空间文件浏览、知识库和数据源选择均在本期范围之外。

## Decisions

### 三个读取入口

建议在 P3Demo 业务后端采用以下只读路由。浏览器只调用本应用；业务后端以当前用户和已验证应用身份读取 Agent OS。

| 用途 | P3Demo（相对 /api/v1/ai/conversations/{id}） | Agent OS（相对 /api/v1/web/conversations/{id}） |
| --- | --- | --- |
| 产物树 | GET /artifacts | GET /files |
| 节点预览 | GET /artifacts/{fileNodeId}/content | GET /files/{fileNodeId}/content |
| 原文件下载 | GET /artifacts/{fileNodeId}/download | GET /files/{fileNodeId}/download |

第三方身份策略仅增加 `GET /api/v1/web/conversations/{id}/files`、`GET /api/v1/web/conversations/{id}/files/{fileNodeId}/content`、`GET /api/v1/web/conversations/{id}/files/{fileNodeId}/download` 三条精确路由，ID 限数字。仍要求有效的同租户用户令牌与登记应用 App-Key，由现有会话服务检查应用、租户、用户和个人会话范围。上传、按路径下载、工作空间文件及其他未登记路由继续拒绝，不按前缀放开。

业务后端从当前会话的受信文件树提取根目录下的 `generated/` 子树并保留子目录。预览与下载使用同一归属判断，确认会话访问权限、目标节点属于该会话、节点类型为文件及其规范路径包含完整的根目录段 `generated` 后，才按节点 ID 调用上游。调用方传入的路径或消息中的链接不能作为授权依据。目录过滤用于产物读接口，现有附件处理可以继续使用会话文件树。

预览保留上游的 `previewType`、`content`、`mimeType` 与 `truncated`。图片预览返回的内容字段可为空，由页面通过已鉴权的下载入口获得 Blob。HTML 在隔离的预览上下文中展示。下载端转发文件字节，保留适用的内容类型与文件名。

### React 页面组合

在 P3Demo 的 `apps/web/src` 组合业务组件。从 `@ej/ai/ai-elements/markdown` 使用 `MarkdownRenderer`，以 `finalized` 识别完成的回答，使用 `workspaceFileLinkResolver` 和 `onWorkspaceFileClick` 将可点击的产物路径交给页面定位；显式 Markdown 文件链接使用相同的目录和路径规则。已存在的 Markdown 链接在节点增强时会被跳过，需在链接渲染或点击处理处一并覆盖，确保本地临时文件和上传路径呈现为文字。

从 `@ej/ui` 使用右侧 `Drawer`、`EjTree` 和 `EjImage`。会话 ID、已选节点、打开状态及 P3Demo 的接口调用放在业务页面。`ej-tooling` 提供工程配置。DataClaw Web 的 `ChatWorkspaceDrawer.vue` 用于核对交互，React 实现通过上述公共组件入口组合。

### 会话和完整路径共同定位

将相对路径和带前导斜杠的同一产物路径映射到当前会话的文件树。选择目录外路径时保持为文字。用户点开交付路径时，精确匹配节点，缺失则刷新一次并重试。按会话 ID 与节点 ID 关联异步结果，切换会话时清理旧树、选中项和图片对象 URL。

历史回答引用的是会话内的完整路径。同一路径被覆盖后，文件节点 ID 可能保持不变。用户再次打开时重新读取当前预览内容，图片也重新获取当前文件；下载时以服务端返回的当前文件为准。需要分别交付初稿和定稿时，使用不同的完整路径即可。

文件已移走或删除时，已有列表可能仍包含旧节点。若按原路径找不到节点，或读取该节点时服务端提示文件不存在，本次打开操作合计刷新列表一次，再按原路径重试。仍不可用时清除旧预览并提示“文件暂不可用”。

## Risks / Trade-offs

- 刚生成的文件可能尚未入索引。一次刷新仍缺失时保留回答并提示文件暂不可用，避免打开同名文件。
- 完整树过滤和节点校验会增加上游读取。首期复用同一套受权限约束的查询规则，避免客户端持有的历史树决定下载权限。

## Migration Plan

依赖会话接入变更提供的个人会话授权和页面入口。先接通 P3Demo 后端的三个读取操作，再在智能问答页启用右侧抽屉，与 DataClaw 的既有会话文件接口联合验证。

## 实施范围确认

用户已明确授权本变更修改并提交独立项目 `/Users/hualong/jxworkspace/p3demo-wrap/p3demo`，以及上述三个精确只读上游路由的第三方身份接入与授权回归。
