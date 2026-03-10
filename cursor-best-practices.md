# Cursor 使用技巧与最佳实践

> Cursor 是基于 VS Code 的 AI 编辑器，深度集成 AI 能力，适合日常编码工作流。

---

## 目录

1. [核心交互模式](#核心交互模式)
2. [快捷键](#快捷键)
3. [Context 引用（@ 符号）](#context-引用-符号)
4. [Cursor Rules](#cursor-rules)
5. [模型选择策略](#模型选择策略)
6. [MCP 支持](#mcp-支持)
7. [Notepads](#notepads)
8. [大型代码库技巧](#大型代码库技巧)
9. [提示词技巧](#提示词技巧)
10. [隐私与安全](#隐私与安全)

---

## 核心交互模式

Cursor 有三种主要 AI 交互模式，理解它们的区别是用好 Cursor 的关键。

### Ask 模式（`Ctrl+L`）

- 纯对话，不直接修改代码
- 适合：理解代码、提问、探索方案
- Claude/GPT 会引用你的代码库回答问题
- 回答中的代码可以一键 Apply 到编辑器

### Edit 模式（`Ctrl+K`）

- 直接在当前文件内联编辑
- 适合：修改选中代码、快速重构、生成代码片段
- 有 diff 预览，可以 Accept / Reject
- 速度快，适合小范围精准修改

### Agent 模式（`Ctrl+Shift+I` 或 Composer）

- 自主完成多步骤任务，可跨文件操作
- 适合：实现新功能、重构模块、修复复杂 bug
- 会自动读取文件、执行终端命令、创建新文件
- 类似 Claude Code，但在 GUI 环境中

**选择原则：**
```
理解代码 → Ask
修改当前文件 → Edit (Ctrl+K)
跨文件任务 → Agent
```

---

## 快捷键

### 核心快捷键

| 快捷键 | 作用 |
|--------|------|
| `Tab` | 接受 AI 自动补全 |
| `Esc` | 拒绝自动补全 |
| `Ctrl+K` | 内联编辑（Edit 模式） |
| `Ctrl+L` | 打开 Chat（Ask 模式） |
| `Ctrl+Shift+I` | 打开 Composer（Agent 模式） |
| `Ctrl+Shift+L` | 将选中代码添加到 Chat |
| `Ctrl+Enter` | 在 Composer 中提交（Agent 模式执行） |
| `Ctrl+/` | 切换行注释 |

> macOS 用 `Cmd` 替换 `Ctrl`

### Chat 内快捷键

| 快捷键 | 作用 |
|--------|------|
| `@` | 触发 context 引用菜单 |
| `↑` | 编辑上一条消息 |
| `Ctrl+Enter` | 使用 Agent 模式发送 |
| `Shift+Enter` | 换行（不发送） |

### 代码接受快捷键

| 快捷键 | 作用 |
|--------|------|
| `Ctrl+Y` | 接受所有 diff 更改 |
| `Ctrl+N` | 拒绝所有 diff 更改 |
| `Alt+]` / `Alt+[` | 逐个接受/拒绝 diff 块 |

---

## Context 引用（@ 符号）

在 Chat 或 Composer 中输入 `@` 可以精确控制 AI 的上下文。

### 文件和代码

| 引用 | 说明 |
|------|------|
| `@文件名` | 引用特定文件 |
| `@文件夹` | 引用整个目录 |
| `@代码符号` | 引用函数、类、变量 |
| `#文件名` | 快速引用文件（简写） |

### 代码库级别

| 引用 | 说明 |
|------|------|
| `@Codebase` | 让 AI 搜索整个代码库（语义搜索） |
| `@Git` | 引用 git 历史、diff、commit |
| `@Recent Changes` | 最近修改的文件 |

### 外部资源

| 引用 | 说明 |
|------|------|
| `@Web` | 实时网络搜索 |
| `@Docs` | 引用已索引的文档（可添加自定义文档） |
| `@Notepads` | 引用 Notepad 内容 |

### 实用示例

```
# 跨文件重构
"@src/auth/login.ts @src/auth/types.ts
把 login 函数改为 async/await，更新相关类型定义"

# 参考文档实现
"@Docs/React Query 用 React Query 重写 @src/hooks/useFetch.ts"

# 基于 git diff 做 code review
"@Git 审查最近 3 个 commit 的改动，找出潜在问题"

# 搜索代码库
"@Codebase 找出所有直接操作数据库的地方，不经过 repository 层的"
```

---

## Cursor Rules

Cursor Rules 是告诉 AI 如何在你的项目中工作的配置文件，类似 Claude Code 的 `CLAUDE.md`。

### 两种配置方式

**1. 项目级 `.cursor/rules/`（推荐）**

在项目根目录创建 `.cursor/rules/` 文件夹，每个 `.mdc` 文件是一条规则：

```
.cursor/
  rules/
    general.mdc        # 通用规范
    typescript.mdc     # TypeScript 规范
    testing.mdc        # 测试规范
    api-design.mdc     # API 设计规范
```

**2. 全局规则**

`Cursor Settings → Rules → User Rules`，适用于所有项目。

### `.mdc` 文件格式

```markdown
---
description: TypeScript 编码规范
globs: ["**/*.ts", "**/*.tsx"]
alwaysApply: false
---

# TypeScript 规范

- 所有函数必须有明确的返回类型注解
- 使用 `interface` 而不是 `type` 定义对象结构
- 禁止使用 `any`，用 `unknown` 替代
- 异步函数统一使用 async/await，不用 .then() 链
- 错误处理使用自定义 Error 类，不直接 throw string
```

### 规则触发方式

| `alwaysApply` | `globs` | 触发时机 |
|---------------|---------|----------|
| `true` | - | 每次对话都加载 |
| `false` | 有 | 匹配文件时自动加载 |
| `false` | 无 | 手动 `@Rules` 引用时加载 |

### 实用规则示例

**`general.mdc`**
```markdown
---
alwaysApply: true
---

## 项目信息
- 技术栈：Next.js 14 + TypeScript + Tailwind CSS + Prisma
- 包管理：pnpm
- 测试：Vitest + Testing Library

## 编码原则
- 优先修改现有文件，不随意创建新文件
- 函数保持单一职责，超过 50 行考虑拆分
- 所有用户输入必须验证（使用 Zod）

## 禁止事项
- 不使用 `console.log`（用 logger 模块）
- 不直接操作 DOM（用 React refs）
- 不在组件中写业务逻辑（放到 hooks 或 services）
```

**`testing.mdc`**
```markdown
---
globs: ["**/*.test.ts", "**/*.spec.ts"]
---

## 测试规范
- 测试文件与源文件同目录
- 使用 describe/it 结构，描述用中文
- 每个测试只验证一件事
- Mock 外部依赖，不发真实网络请求
- 运行命令：`pnpm test --run`（非 watch 模式）
```

---

## 模型选择策略

Cursor 支持多种模型，根据任务选择合适的模型可以平衡质量和速度。

| 模型 | 适用场景 |
|------|----------|
| `claude-sonnet-4-5` | 日常编码，性价比最高（推荐默认） |
| `claude-opus-4` | 复杂架构设计、难以复现的 bug |
| `gpt-4o` | 需要视觉输入（截图、设计稿）时 |
| `cursor-small` | 简单补全、快速问答，速度最快 |
| `o1 / o3` | 算法题、数学推导、逻辑密集型任务 |

**切换方式：** Chat 输入框右侧的模型选择器，或 `Ctrl+/` 快速切换。

---

## MCP 支持

Cursor 从 0.45 版本开始支持 MCP（Model Context Protocol）。

### 配置方式

`Cursor Settings → MCP` 或编辑 `~/.cursor/mcp.json`：

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/Users/yourname/projects"]
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

### 在 Agent 模式中使用

配置好 MCP 后，在 Composer（Agent 模式）中 AI 可以直接调用这些工具：

```
"查询 users 表中最近 7 天注册的用户数量"
→ AI 自动调用 postgres MCP 执行 SQL

"在 GitHub 上创建一个 issue，标题是 '修复登录超时问题'"
→ AI 自动调用 github MCP 创建 issue
```

---

## Notepads

Notepads 是 Cursor 的持久化笔记功能，可以存储常用的上下文、模板、规范。

### 创建 Notepad

`Cursor Settings → Notepads → New Notepad`

### 使用场景

```markdown
# Notepad: API 设计模板
所有 REST API 遵循以下规范：
- URL 使用 kebab-case：/user-profiles
- 响应格式：{ data: T, error: string | null, meta: {...} }
- 错误码：400 参数错误，401 未认证，403 无权限，404 不存在，500 服务器错误
- 分页参数：?page=1&pageSize=20
```

在 Chat 中用 `@Notepads` 引用：

```
"@Notepads/API设计模板 按照规范实现用户列表接口"
```

---

## 大型代码库技巧

### 开启 Codebase Indexing

`Cursor Settings → Features → Codebase Indexing` 开启后，`@Codebase` 引用会更准确。

配置 `.cursorignore` 排除不需要索引的目录：

```
node_modules/
dist/
.next/
coverage/
*.lock
*.log
```

### 精准提供上下文

大型项目中，不要依赖 AI 自己找文件，主动用 `@` 指定：

```
# 不够好
"修复用户登录的 bug"

# 更好
"@src/features/auth/LoginForm.tsx @src/api/auth.ts @src/store/authSlice.ts
修复登录后 token 没有正确存储到 Redux store 的问题"
```

### 分而治之

```
# 不要一次性要求太多
"重构整个 auth 模块"  ← 容易出错

# 分步骤
"第一步：只重构 @src/auth/login.ts，把回调改为 async/await"
"第二步：更新 @src/auth/register.ts 保持风格一致"
"第三步：更新相关测试文件"
```

---

## 提示词技巧

### 给出明确的约束

```
"重构 @src/components/UserCard.tsx：
- 不改变组件的 props 接口
- 把内联样式改为 Tailwind 类
- 提取重复逻辑到自定义 hook
- 不添加新功能"
```

### 先解释再修改

```
"先解释 @src/utils/cache.ts 的缓存失效逻辑，
确认我理解正确后再修改"
```

### 要求输出格式

```
"分析 @src/ 目录，列出所有性能问题，
用表格格式输出：问题描述 | 文件位置 | 严重程度 | 修复建议"
```

### 利用 git 上下文

```
"@Git diff HEAD~3 审查这三个 commit 的改动，
重点检查安全漏洞和边界条件处理"
```

---

## 隐私与安全

### Privacy Mode

`Cursor Settings → General → Privacy Mode` 开启后：
- 代码不会被用于训练模型
- 请求不会被 Cursor 服务器记录
- 适合处理敏感代码库

### `.cursorignore`

类似 `.gitignore`，阻止特定文件被发送给 AI：

```
# .cursorignore
.env
.env.*
secrets/
credentials/
*.pem
*.key
config/production.yml
```

### 团队配置共享

把 `.cursor/rules/` 提交到 git，团队共享 AI 行为规范：

```bash
git add .cursor/rules/
git commit -m "add cursor rules for team consistency"
```

---

## 推荐工作流总结

```
1. 项目初始化
   └── 创建 .cursor/rules/ 目录
   └── 写 general.mdc（技术栈、规范、禁止事项）
   └── 配置 .cursorignore

2. 日常编码
   └── Tab 接受内联补全
   └── Ctrl+K 快速修改当前代码
   └── Ctrl+L 提问、理解代码

3. 复杂任务
   └── Ctrl+Shift+I 打开 Composer（Agent 模式）
   └── 用 @ 精确指定上下文
   └── 分步骤执行，每步确认后再继续

4. 代码审查
   └── @Git 引用 diff
   └── Ask 模式分析潜在问题
   └── 用 Notepads 存储审查 checklist
```

---

*最后更新：2025-03 | 基于 Cursor 0.45+*
