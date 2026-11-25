# AGENT

## Git Commit 规范

- `git commit` 提交信息使用中文描述。
- 推荐格式：`type(scope): 中文说明`

示例：

```text
fix(agent-os-web): 调整模型供应商地址占位文案
```

### 常用类型

- `feat`: 新功能、新能力
- `fix`: 修复缺陷、修正异常行为
- `refactor`: 重构实现，不改变对外行为
- `docs`: 文档更新
- `test`: 测试相关（新增/修改/修复测试用例）
- `chore`: 杂项维护、构建配置、脚本调整

### 提交示例

```text
feat(agent-os-web): 新增模型供应商配置页面
fix(agent-os-web): 调整模型供应商地址占位文案
refactor(agent-os-service): 拆分桌面客户端注册服务逻辑
docs(dataclaw): 补充桌面端接口说明
chore(dataclaw): 更新 eslint 与 prettier 配置
```

### 编写要求

- `type` 使用英文小写：`feat`、`fix`、`refactor`、`docs`、`chore`
- `scope` 使用受影响的模块或目录名，例如 `agent-os-web`、`agent-os-service`、`dataclaw`
- 冒号后使用简洁中文，直接说明本次变更内容
- 一次提交只表达一个主要变更，避免把多个无关修改混在同一条提交信息中

### 主键生成约定

- 如果表主键是 `int(8)` / `BIGINT` 语义的自增主键，新增记录时默认使用雪花 ID 生成，不依赖数据库自增。