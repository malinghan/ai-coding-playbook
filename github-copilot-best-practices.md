# GitHub Copilot 使用技巧与最佳实践

> GitHub Copilot 是 GitHub 官方 AI 编程助手，深度集成在 VS Code、JetBrains 等主流 IDE 中，提供代码补全、Chat 对话和 Agent 模式。

---

## 目录

1. [核心功能模式](#核心功能模式)
2. [快捷键](#快捷键)
3. [Copilot Chat](#copilot-chat)
4. [Agent 模式（Copilot Workspace）](#agent-模式copilot-workspace)
5. [自定义指令（Custom Instructions）](#自定义指令custom-instructions)
6. [Context 控制](#context-控制)
7. [模型选择](#模型选择)
8. [CLI 集成](#cli-集成)
9. [提示词技巧](#提示词技巧)
10. [隐私与安全](#隐私与安全)

---

## 核心功能模式

### 内联补全（Inline Completion）

- 在编辑器中实时预测并补全代码
- 基于当前文件上下文和光标位置生成建议
- 适合：快速写样板代码、补全函数体、生成重复模式

### Copilot Chat（`Ctrl+Shift+I`）

- 对话式 AI 助手，可以提问、解释代码、生成代码
- 支持引用当前文件、选中代码、整个工作区
- 适合：理解代码、调试、重构建议

### Agent 模式（`#` 指令）

- 在 Chat 中使用 `#` 引用工具，让 Copilot 执行多步骤任务
- 可以读取文件、运行终端命令、创建/修改文件
- 适合：跨文件重构、自动化任务、复杂功能实现

**选择原则：**
```
写代码时自动补全    → 内联补全（Tab 接受）
提问/理解代码       → Copilot Chat
跨文件复杂任务      → Agent 模式
```

---

## 快捷键

### VS Code 快捷键

| 快捷键 | 作用 |
|--------|------|
| `Tab` | 接受当前补全建议 |
| `Esc` | 拒绝补全建议 |
| `Alt+]` / `Alt+[` | 切换到下一个/上一个补全建议 |
| `Ctrl+Enter` | 打开补全建议面板（查看多个候选） |
| `Ctrl+Shift+I` | 打开 Copilot Chat 面板 |
| `Ctrl+I` | 内联 Chat（在编辑器中直接对话） |
| `Ctrl+Shift+Alt+I` | 在终端中打开 Copilot Chat |

> macOS 用 `Cmd` 替换 `Ctrl`

### Chat 内快捷键

| 快捷键 | 作用 |
|--------|------|
| `@` | 引用 Chat 参与者（workspace、vscode 等） |
| `#` | 引用工具或文件 |
| `/` | 触发斜杠命令 |
| `↑` | 编辑上一条消息 |

### 内联 Chat 快捷键

| 快捷键 | 作用 |
|--------|------|
| `Ctrl+I` | 在光标处打开内联 Chat |
| `Alt+Enter` | 接受内联 Chat 的修改 |
| `Esc` | 关闭内联 Chat |

---

## Copilot Chat

### Chat 参与者（`@` 引用）

| 参与者 | 说明 |
|--------|------|
| `@workspace` | 让 Copilot 搜索整个工作区 |
| `@vscode` | 询问 VS Code 功能和设置 |
| `@terminal` | 询问终端命令和 shell 脚本 |
| `@github` | 搜索 GitHub 仓库、Issues、PR |

### 斜杠命令（`/` 命令）

| 命令 | 说明 |
|------|------|
| `/explain` | 解释选中代码的工作原理 |
| `/fix` | 修复选中代码中的问题 |
| `/tests` | 为选中代码生成测试 |
| `/doc` | 为选中代码生成文档注释 |
| `/simplify` | 简化选中代码 |
| `/new` | 创建新文件或项目脚手架 |
| `/newNotebook` | 创建新的 Jupyter Notebook |

### 实用示例

```
# 理解复杂代码
选中代码 → /explain

# 修复 bug
选中有问题的代码 → /fix 这里的 token 刷新逻辑有问题

# 搜索整个项目
@workspace 找出所有没有错误处理的 async 函数

# 生成测试
选中函数 → /tests 使用 Jest，覆盖边界条件

# 询问 VS Code 操作
@vscode 如何配置 ESLint 自动修复保存时运行
```

---

## Agent 模式（Copilot Workspace）

Agent 模式让 Copilot 自主完成多步骤任务，类似 Claude Code 的 Agent 工具。

### 启用 Agent 模式

在 Chat 中切换到 Agent 模式（Chat 输入框左侧的模式选择器）。

### 工具引用（`#` 命令）

| 工具 | 说明 |
|------|------|
| `#file` | 引用特定文件 |
| `#editor` | 引用当前编辑器内容 |
| `#selection` | 引用当前选中内容 |
| `#terminalLastCommand` | 引用上一条终端命令及输出 |
| `#terminalSelection` | 引用终端中选中的内容 |
| `#codebase` | 搜索整个代码库 |

### 实用示例

```
# 基于错误输出修复问题
#terminalLastCommand 分析这个错误，找出根本原因并修复

# 跨文件重构
#file:src/auth/login.ts #file:src/auth/types.ts
把登录逻辑重构为 async/await，更新相关类型

# 基于当前代码生成测试
#editor 为这个文件生成完整的测试套件

# 实现新功能
@workspace 参考现有的 user API，实现 team 管理的 CRUD 接口
```

---

## 自定义指令（Custom Instructions）

Custom Instructions 是告诉 Copilot 如何在你的项目中工作的配置，类似 Claude Code 的 `CLAUDE.md`。

### 配置方式

**方式 1：VS Code 设置**

`Settings → GitHub Copilot → Chat: Code Generation Instructions`

```json
{
  "github.copilot.chat.codeGeneration.instructions": [
    {
      "text": "使用 TypeScript，所有函数必须有返回类型注解。使用 pnpm 管理依赖。测试框架使用 Vitest。"
    }
  ]
}
```

**方式 2：`.github/copilot-instructions.md`（推荐，可提交到 git）**

```markdown
# 项目编码规范

## 技术栈
- Node.js 20 + TypeScript 5
- Fastify（不用 Express）
- Prisma ORM + PostgreSQL
- Vitest 测试框架

## 编码规范
- 所有函数必须有明确的返回类型
- 使用 async/await，不用 .then() 链
- 错误处理使用自定义 AppError 类
- 日志使用 pino，不用 console.log

## 禁止事项
- 不使用 any 类型
- 不在 controller 层写业务逻辑
- 不直接操作数据库（必须通过 repository 层）
- 不提交 .env 文件

## 常用命令
- `pnpm test --run` — 运行测试（单次）
- `pnpm dev` — 启动开发服务器
- `pnpm db:migrate` — 执行数据库迁移
```

### 指令优先级

```
VS Code 用户设置 > .github/copilot-instructions.md > 默认行为
```

---

## Context 控制

### 主动引用文件

不要依赖 Copilot 自动猜测上下文，主动用 `#file` 指定：

```
# 不够好
"修复登录 bug"

# 更好
#file:src/auth/login.ts #file:src/middleware/auth.ts
修复登录后 JWT token 没有正确设置到 cookie 的问题
```

### 使用 `.copilotignore`

类似 `.gitignore`，阻止特定文件被发送给 Copilot：

```
# .copilotignore
.env
.env.*
secrets/
*.pem
*.key
config/production.yml
node_modules/
dist/
```

### 控制补全范围

在 VS Code 设置中配置哪些语言启用/禁用补全：

```json
{
  "github.copilot.enable": {
    "*": true,
    "markdown": false,
    "plaintext": false,
    "yaml": true
  }
}
```

---

## 模型选择

Copilot 支持多种底层模型，可在 Chat 界面切换：

| 模型 | 特点 | 适用场景 |
|------|------|----------|
| `GPT-4o` | 均衡，速度快 | 日常编码（默认推荐） |
| `Claude Sonnet 4.5` | 代码质量高，理解力强 | 复杂重构、架构设计 |
| `Claude Opus 4` | 最强推理 | 疑难 bug、算法设计 |
| `o3` | 强推理，较慢 | 数学、算法、逻辑密集型 |
| `Gemini 2.0 Flash` | 速度极快 | 简单问答、快速补全 |

**切换方式：** Chat 输入框右侧的模型选择器。

---

## CLI 集成

### 安装 GitHub Copilot CLI

```bash
# 安装（需要 GitHub CLI）
gh extension install github/gh-copilot

# 认证
gh auth login
```

### 常用命令

```bash
# 解释 shell 命令
gh copilot explain "git rebase -i HEAD~3"

# 建议命令（描述你想做什么）
gh copilot suggest "找出占用 8080 端口的进程并杀掉它"

# 在终端中直接使用（别名）
ghcs "压缩 images/ 目录下所有 PNG 文件"  # suggest
ghce "awk '{print $1}' file.txt"          # explain
```

### 配置别名

```bash
# ~/.zshrc 或 ~/.bashrc
alias ghcs='gh copilot suggest'
alias ghce='gh copilot explain'
```

---

## 提示词技巧

### 给出明确的约束

```
# 好
"重构 #file:src/utils/cache.ts：
- 不改变公共 API（函数签名保持不变）
- 把 Map 替换为 LRU Cache（使用 lru-cache 库）
- 添加 TTL 支持
- 不添加其他功能"

# 不够好
"优化缓存模块"
```

### 先解释再修改

```
"先解释 #file:src/queue/processor.ts 的消息处理流程，
特别是错误重试逻辑，确认我理解正确后再修改"
```

### 利用测试驱动

```
"用 TDD 方式为 #file:src/api/payments.ts 添加退款功能：
1. 先写测试用例（正常退款、部分退款、超额退款的边界情况）
2. 运行测试确认失败
3. 实现功能
4. 确认测试通过"
```

### 分步骤处理复杂任务

```
# 第一步：分析
"@workspace 找出所有直接使用 fetch() 的地方，
不经过统一的 http client 封装，列出文件和行号"

# 确认后，第二步：修复
"把 #file:src/api/users.ts 中的 fetch 调用改为使用 httpClient"
```

### 要求输出格式

```
"分析 @workspace 中所有 API 路由，
用表格输出：路由 | HTTP 方法 | 是否有认证 | 是否有输入验证 | 备注"
```

---

## 隐私与安全

### 内容排除（Content Exclusions）

在 GitHub 仓库或组织设置中配置，阻止特定文件被发送给 Copilot：

```yaml
# .github/copilot-exclusions.yml（组织级配置）
paths:
  - "**/.env"
  - "**/secrets/**"
  - "**/credentials/**"
  - "**/*.pem"
  - "**/config/production.*"
```

### 代码引用过滤

在设置中开启 `Suggestions matching public code: Block`，避免 Copilot 建议与公开代码完全相同的片段（降低版权风险）。

### 企业版注意事项

- 企业版（Copilot Business/Enterprise）默认不用代码训练模型
- 可以在组织级别统一配置 Custom Instructions
- 支持私有模型部署（Copilot Enterprise）

---

## 与其他工具对比

| | GitHub Copilot | Claude Code | Cursor |
|--|----------------|-------------|--------|
| 集成方式 | IDE 插件 | 终端 CLI | 独立 IDE |
| 内联补全 | ✅ 最成熟 | ❌ | ✅ |
| Chat 对话 | ✅ | ✅ | ✅ |
| Agent 模式 | ✅ | ✅ 最强 | ✅ |
| 项目配置文件 | `copilot-instructions.md` | `CLAUDE.md` | `.cursor/rules/` |
| MCP 支持 | 有限 | ✅ 完整 | ✅ |
| CLI 工具 | `gh copilot` | `claude` | ❌ |
| 价格 | $10-19/月 | 按 token 计费 | $20/月 |

**推荐组合：**
- 日常编码补全 → GitHub Copilot（IDE 集成最顺滑）
- 复杂多文件任务 → Claude Code（Agent 能力最强）
- GUI 环境跨文件重构 → Cursor

---

## 推荐工作流总结

```
1. 项目初始化
   └── 创建 .github/copilot-instructions.md
   └── 配置 .copilotignore
   └── 设置内容排除规则

2. 日常编码
   └── Tab 接受内联补全
   └── Ctrl+I 内联 Chat 快速修改
   └── /fix /explain /tests 斜杠命令

3. 复杂任务
   └── 切换 Agent 模式
   └── 用 #file 精确指定上下文
   └── 分步骤执行，每步确认

4. 终端工作
   └── gh copilot suggest 生成命令
   └── gh copilot explain 理解命令
```

---

*最后更新：2026-03 | 基于 GitHub Copilot + VS Code*
