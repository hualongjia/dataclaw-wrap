# Proposal

## Why

管理端的权限矩阵、角色管理和权限码管理分散在三个菜单及路由，页面内联表单使授权维护缺少清晰层次。统一入口并参考 p3demo 的三 Tab 布局，让管理人员在一个页面完成查看和维护。

## What Changes

- 系统配置中的“权限配置(Access)”更名为“角色权限”，主地址改为 `/role-permissions`。
- 移除独立角色管理、权限码管理菜单，将权限矩阵、角色管理、权限码管理整合为三个 Tab，默认权限矩阵。
- 参照 p3demo 实现权限树矩阵、搜索工具栏、角色分页表格、权限码层级表格及编辑弹窗。
- 旧地址重定向至新页面对应 Tab；切换保留搜索与分页，未保存修改受保护。

## Capabilities

### New Capabilities

无。

### Modified Capabilities

- `admin-role-permissions`: 增加统一导航路由、三 Tab 布局及授权编辑交互要求。

## Impact

主要影响 `dataclaw/apps/agent-os/web/src/router/routes.ts`、`views/RolePermissionsView.vue`、相关子组件和前端测试。复用已有角色权限 API、应用边界和成员差量提交；必要的权限树数据由现有 parentId 构造，不增加依赖或修改 Auth3 配置。现行主规格仍描述系统管理员角色资格，而运行代码已改为应用管理员资格，这是既有差异，本次仅规划页面重组，不调整授权策略。
