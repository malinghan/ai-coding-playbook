# Windsurf 使用技巧与最佳实践

> Windsurf 是 Codeium 推出的 AI 原生 IDE，基于 VS Code，主打 "Flow" 概念——让 AI 和开发者在同一上下文中协作，减少切换摩擦。

---

## 目录

1. [核心概念：Cascade](#核心概念cascade)
2. [快捷键](#快捷键)
3. [Cascade 使用技巧](#cascade-使用技巧)
4. [Rules（项目规范）](#rules项目规范)
5. [Context 管理](#context-管理)
6. [模型选择](#模型选择)
7. [MCP 支持](#mcp-支持)
8. [提示词技巧](#提示词技巧)
9. [与其他工具对比](#与其他工具对比)

---

## 核心概念：Cascade

Cascade 是 Windsurf 的核心 AI 功能，分两种模式：

### Write 模式

- AI 直接修改代码，有 diff 预览
- 适合：实现功能、重构、修复 bug
- 会自动读取相关文件，跨文件操作

### Chat 模式

- 纯对话，不直接修改代码
- 适合：理解代码、探索方案、提问

**选择原则：**
```
需要修改代码    → Write 模式
理解/探索       → Chat 模式
```

Cascade 的核心优势是**持续感知上下文**——它会跟踪你的操作历史、终端输出、文件变更，不需要你反复解释背景。

---

## 快捷键

| 快捷键 | 作用 |
|--------|------|
| `Ctrl+L` | 打开 Cascade 面板 |
| `Ctrl+I` | 内联 AI 编辑 |
| `Tab` | 接受代码补全 |
| `Esc` | 拒绝补全 |
| `Ctrl+Shift+L` | 将选中代码发送到 Cascade |
| `Alt+]` / `Alt+[` | 切换补全候选 |

> macOS 用 `Cmd` 替换 `Ctrl`

---

## Cascade 使用技巧

### 让 Cascade 感知终端输出

Cascade 会自动读取终端的错误输出，直接说"修复这个错误"即可：

```
# 终端出现报错后，在 Cascade 中输入：
"修复刚才的错误"

# Cascade 会自动读取终端输出，分析并修复
```

### 利用操作历史

Cascade 记住你做过的事，可以引用：

```
"撤销刚才对 auth 模块的修改"
"继续刚才没完成的重构"
"基于刚才的修改，更新对应的测试"
```

### 多步骤任务

```
"完成以下任务：
1. 在 src/api/ 下创建 teams.ts，实现团队 CRUD
2. 在 src/routes/ 下注册路由
3. 为新接口生成测试
4. 更新 README 中的 API 文档"
```

### 要求先计划再执行

```
"在修改任何代码之前，先列出你的修改计划：
- 需要修改哪些文件
- 每个文件改什么
- 可能的风险
等我确认后再开始"
```

---

## Rules（项目规范）

Windsurf Rules 是告诉 AI 如何在项目中工作的配置文件，类似 Claude Code 的 `CLAUDE.md`。

### 配置位置

- **全局规则**：`Windsurf Settings → Rules → Global Rules`
- **项目规则**：项目根目录的 `.windsurfrules` 文件（可提交到 git）

### `.windsurfrules` 示例

```markdown
# 项目规范

## 技术栈
- Python 3.12 + FastAPI
- SQLAlchemy 2.0（async）+ PostgreSQL
- Pydantic v2 数据验证
- pytest + httpx 测试

## 编码规范
- 使用 uv 管理依赖，不用 pip/poetry
- 所有数据库操作必须 async
- API 响应统一用 Pydantic 模型
- 遵循 PEP 8，black 格式化（行宽 88）
- 类型注解必须完整

## 目录结构
- app/api/ — 路由层（只做参数解析和响应格式化）
- app/services/ — 业务逻辑层
- app/repositories/ — 数据访问层
- tests/ — 测试（镜像 app/ 结构）

## 禁止事项
- 不在 API 层写业务逻辑
- 不直接在 service 层操作数据库
- 不使用同步数据库操作
- 不提交 .env 文件
- 不修改 alembic/versions/ 下的迁移文件

## 常用命令
- `uv run pytest` — 运行测试
- `uv run uvicorn app.main:app --reload` — 启动开发服务器
- `uv run alembic upgrade head` — 执行迁移
```

### 最佳实践

- 把"禁止事项"写得和"要做什么"一样详细
- 包含常用命令，让 AI 知道如何运行和测试
- 定期更新，过时的规则会误导 AI

---

## Context 管理

### `@` 引用文件

在 Cascade 中用 `@` 精确指定上下文：

```
@src/auth/login.py @src/auth/types.py
把登录逻辑重构为使用 JWT，保持现有 API 兼容
```

### 排除不相关文件

配置 `.codeiumignore`（类似 `.gitignore`）：

```
node_modules/
dist/
.next/
coverage/
*.lock
*.log
.env
secrets/
```

### 精准提供上下文

大型项目中，主动指定文件比让 AI 自己搜索更可靠：

```
# 不够好
"修复支付 bug"

# 更好
@src/payments/processor.py @src/payments/models.py @tests/test_payments.py
修复支付回调处理中的幂等性问题，确保重复回调不会重复扣款
```

---

## 模型选择

Windsurf 支持多种模型，在 Cascade 面板中切换：

| 模型 | 特点 | 适用场景 |
|------|------|----------|
| `Claude Sonnet 4.5` | 代码质量高，均衡 | 日常编码（推荐默认） |
| `Claude Opus 4` | 最强推理 | 复杂架构、疑难 bug |
| `GPT-4o` | 速度快 | 简单任务、快速问答 |
| `Gemini 2.0 Pro` | 长上下文 | 大文件分析 |

**选择原则：**
```
日常编码          → Claude Sonnet 4.5
复杂推理/架构     → Claude Opus 4
速度优先          → GPT-4o
```

---

## MCP 支持

Windsurf 支持 MCP（Model Context Protocol），让 AI 连接外部工具。

### 配置方式

编辑 `~/.codeium/windsurf/mcp_config.json`：

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/project"]
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "ghp_xxxx"
      }
    },
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres", "postgresql://localhost/mydb"]
    }
  }
}
```

### 在 Cascade 中使用

配置好 MCP 后，Cascade 可以直接调用这些工具：

```
"查询 orders 表中今天的订单总金额"
→ Cascade 自动调用 postgres MCP 执行 SQL

"在 GitHub 上创建 issue：修复用户登录超时问题"
→ Cascade 自动调用 github MCP
```

---

## 提示词技巧

### 明确任务边界

```
"只修改 @src/auth/login.py：
- 把密码验证提取到独立函数 validate_password()
- 不改变函数签名
- 不添加新功能
- 保持现有测试通过"
```

### 分步骤执行

```
# 第一步：分析
"分析 @src/database/ 目录，找出所有 N+1 查询问题，
只列出问题，不要修改代码"

# 确认后，第二步：修复
"修复你找到的第 1 个 N+1 问题"
```

### 利用 TDD

```
"用 TDD 实现用户注册功能：
1. 先写测试（正常注册、邮箱重复、密码太短）
2. 运行 `uv run pytest tests/test_register.py`，确认失败
3. 实现功能
4. 再次运行测试，确认通过"
```

### 要求解释后再修改

```
"先解释 @src/cache/redis_client.py 的连接池管理逻辑，
特别是重连机制，确认我理解正确后再优化"
```

---

## 与其他工具对比

| | Windsurf | Cursor | Claude Code |
|--|----------|--------|-------------|
| 类型 | AI 原生 IDE | AI 原生 IDE | 终端 CLI |
| 基础 | VS Code | VS Code | 独立 |
| 核心功能 | Cascade（持续上下文） | Composer（Agent） | Agent |
| 项目配置 | `.windsurfrules` | `.cursor/rules/` | `CLAUDE.md` |
| MCP 支持 | ✅ | ✅ | ✅ 最完整 |
| 终端感知 | ✅ 自动读取 | 部分 | ✅ |
| 价格 | 免费起步 | $20/月 | 按 token |

**推荐场景：**
- 想要免费 AI IDE → Windsurf（免费额度较慷慨）
- 需要最强 Agent 能力 → Claude Code
- 习惯 GUI 且预算充足 → Cursor

---

## 推荐工作流总结

```
1. 项目初始化
   └── 创建 .windsurfrules（技术栈、规范、禁止事项）
   └── 配置 .codeiumignore

2. 日常编码
   └── Tab 接受内联补全
   └── Ctrl+I 内联编辑
   └── Cascade Write 模式处理功能开发

3. 复杂任务
   └── 用 @ 精确指定上下文
   └── 要求先计划再执行
   └── 分步骤，每步确认

4. 调试
   └── 终端报错后直接说"修复这个错误"
   └── Cascade 自动读取终端输出
```

---

*最后更新：2026-03 | 基于 Windsurf 最新版*
