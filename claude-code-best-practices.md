# Claude Code 使用技巧与最佳实践

> Claude Code 是 Anthropic 官方 CLI 工具，让你在终端里直接与 Claude 协作完成编程任务。

---

## 目录

1. [快捷键](#快捷键)
2. [Slash 命令](#slash-命令)
3. [CLAUDE.md — 项目记忆](#claudemd--项目记忆)
4. [Git Worktree 并行开发](#git-worktree-并行开发)
5. [MCP 服务器](#mcp-服务器)
6. [Hooks 自动化](#hooks-自动化)
7. [Skills（自定义命令）](#skills自定义命令)
8. [提示词技巧](#提示词技巧)
9. [多 Agent 工作流](#多-agent-工作流)
10. [其他实用技巧](#其他实用技巧)

---

## 快捷键

| 快捷键 | 作用 |
|--------|------|
| `Esc` | 中断当前响应 |
| `Esc Esc`（双击）| 编辑上一条消息 |
| `Ctrl+C` | 强制退出当前操作 |
| `Ctrl+L` | 清屏（保留对话上下文） |
| `Shift+Tab` | 切换 Auto-accept 模式（自动批准所有工具调用） |
| `Ctrl+R` | 搜索历史命令（shell 级别） |
| 方向键 `↑` / `↓` | 浏览历史输入 |

> **Auto-accept 模式**：`Shift+Tab` 开启后，Claude 的所有文件读写、命令执行都会自动批准，适合你完全信任任务时提速。关闭前记得手动关掉。

---

## Slash 命令

在对话中输入 `/` 触发内置命令：

| 命令 | 说明 |
|------|------|
| `/help` | 查看所有可用命令 |
| `/clear` | 清空当前对话上下文 |
| `/compact` | 压缩对话历史，节省 token（保留关键信息） |
| `/memory` | 查看和编辑持久记忆（`MEMORY.md`） |
| `/init` | 在当前项目生成 `CLAUDE.md` |
| `/model` | 切换模型（Sonnet / Opus / Haiku） |
| `/vim` | 切换 Vim 键位模式 |
| `/doctor` | 检查 Claude Code 环境配置是否正常 |
| `/cost` | 查看本次会话的 token 用量和费用 |
| `/status` | 查看当前状态（模型、权限模式等） |
| `/permissions` | 管理工具调用权限 |
| `/mcp` | 查看已连接的 MCP 服务器 |
| `/bug` | 报告 Claude Code 的 bug |
| `/commit` | 触发 commit skill（生成规范的 git commit） |
| `/review` | 触发代码审查 skill |
| `/pr` | 创建 Pull Request |

---

## CLAUDE.md — 项目记忆

`CLAUDE.md` 是 Claude Code 的"项目说明书"，每次启动会自动加载。

### 放在哪里

- `./CLAUDE.md` — 当前项目（会提交到 git，团队共享）
- `~/.claude/CLAUDE.md` — 全局个人偏好
- `./subdir/CLAUDE.md` — 子目录级别的说明

### 写什么

```markdown
# 项目说明

## 技术栈
- Node.js 20 + TypeScript
- PostgreSQL（通过 Prisma ORM）
- 测试框架：Vitest

## 开发规范
- 使用 pnpm，不用 npm/yarn
- 提交信息遵循 Conventional Commits
- 所有新函数必须有单元测试

## 常用命令
- `pnpm dev` — 启动开发服务器
- `pnpm test --run` — 运行测试（单次，非 watch 模式）
- `pnpm db:migrate` — 执行数据库迁移

## 注意事项
- 不要修改 `src/legacy/` 目录下的文件
- 环境变量在 `.env.local`，不要提交
```

### 最佳实践

- 用 `/init` 让 Claude 自动分析项目生成初稿
- 把"不要做什么"也写进去，比"只写要做什么"更有效
- 定期更新，过时的说明会误导 Claude

---

## Git Worktree 并行开发

Git Worktree 允许你同时在多个分支上工作，配合 Claude Code 可以实现**并行多任务**。

### 基本用法

```bash
# 创建新 worktree（基于 main 分支新建 feature 分支）
git worktree add ../my-project-feature feature/new-api

# 在新 worktree 里启动 Claude Code
cd ../my-project-feature
claude

# 查看所有 worktree
git worktree list

# 删除 worktree
git worktree remove ../my-project-feature
```

### 为什么配合 Claude Code 特别好用

```
主目录 my-project/          ← Claude 实例 A：处理 bug fix
worktree my-project-feat/   ← Claude 实例 B：开发新功能
worktree my-project-docs/   ← Claude 实例 C：更新文档
```

- 每个 worktree 是独立的工作目录，互不干扰
- 可以同时开多个终端，每个跑一个 Claude Code 实例
- 避免频繁切换分支导致的上下文丢失

### 推荐工作流

```bash
# 1. 接到新任务，创建 worktree
git worktree add ../proj-feat-auth feature/auth-refactor

# 2. 进入 worktree，启动 Claude
cd ../proj-feat-auth && claude

# 3. 告诉 Claude 任务目标，让它自主完成
# "重构 src/auth/ 模块，使用 JWT，保持现有 API 兼容"

# 4. 完成后合并，清理 worktree
git worktree remove ../proj-feat-auth
```

---

## MCP 服务器

MCP（Model Context Protocol）是 Anthropic 开放的协议，让 Claude 能连接外部工具和数据源。

### 配置方式

编辑 `~/.claude/settings.json`（全局）或项目根目录的 `.claude/settings.json`：

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/allowed/dir"]
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "your_token"
      }
    },
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres", "postgresql://localhost/mydb"]
    }
  }
}
```

### 常用 MCP 服务器

| 服务器 | 用途 |
|--------|------|
| `@modelcontextprotocol/server-filesystem` | 读写本地文件系统 |
| `@modelcontextprotocol/server-github` | 操作 GitHub Issues、PR、代码 |
| `@modelcontextprotocol/server-postgres` | 查询 PostgreSQL 数据库 |
| `@modelcontextprotocol/server-puppeteer` | 控制浏览器，截图、爬取网页 |
| `@modelcontextprotocol/server-brave-search` | 网络搜索（需要 Brave API key） |
| `@modelcontextprotocol/server-slack` | 读写 Slack 消息 |
| `@modelcontextprotocol/server-memory` | 持久化知识图谱记忆 |

### 查看已连接的 MCP

```bash
/mcp   # 在 Claude Code 对话中查看状态
```

---

## Hooks 自动化

Hooks 让你在特定事件发生时自动执行 shell 命令，实现工作流自动化。

### 配置位置

`~/.claude/settings.json` 或项目 `.claude/settings.json`：

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "prettier --write \"$CLAUDE_FILE_PATH\" 2>/dev/null || true"
          }
        ]
      }
    ],
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "echo \"Running: $CLAUDE_TOOL_INPUT\""
          }
        ]
      }
    ]
  }
}
```

### 常用 Hook 场景

- **自动格式化**：每次 Claude 写文件后自动跑 prettier/eslint
- **安全审计**：在执行 bash 命令前记录日志
- **通知**：任务完成后发送桌面通知
- **测试**：文件修改后自动跑相关测试

---

## Skills（自定义命令）

Skills 是可复用的提示词模板，用 `/skill-name` 触发。

### 内置常用 Skills

| Skill | 触发方式 | 说明 |
|-------|----------|------|
| commit | `/commit` | 生成规范的 git commit message |
| review-code | `/review` | 全面代码审查 |
| write-tests | `/write-tests` | 为当前代码生成测试 |
| explain-code | `/explain` | 用图表和类比解释代码 |
| simplify | `/simplify` | 重构优化代码质量 |

### 创建自定义 Skill

在 `~/.claude/commands/` 目录下创建 `.md` 文件：

```bash
mkdir -p ~/.claude/commands
```

`~/.claude/commands/deploy-check.md`:
```markdown
在部署前执行以下检查：
1. 运行 `pnpm test --run` 确保所有测试通过
2. 运行 `pnpm build` 确保构建成功
3. 检查是否有未提交的更改
4. 检查环境变量配置是否完整
5. 输出检查报告，列出所有问题
```

使用时输入 `/deploy-check` 即可触发。

---

## 提示词技巧

### 给 Claude 明确的角色和约束

```
# 好的提示
"你是一个严格遵循 SOLID 原则的 TypeScript 工程师。
重构 src/user-service.ts，要求：
- 不改变公共 API
- 每个方法不超过 20 行
- 添加完整的错误处理"

# 不够好的提示
"帮我优化这个文件"
```

### 分解复杂任务

```
# 先规划，再执行
"先分析 src/auth/ 目录的结构，列出需要修改的文件和修改原因，
等我确认后再开始修改"
```

### 使用 ultrathink 处理复杂问题

在提示词中加入 `think hard`、`think step by step` 或 `ultrathink` 触发深度推理：

```
"ultrathink: 设计一个支持百万并发的消息队列系统架构"
```

### 引用具体文件和行号

```
"查看 src/api/routes.ts:45-80，解释这段中间件的执行顺序"
```

### 让 Claude 先读后改

```
"先读取并理解 src/database/ 下的所有文件，
然后告诉我你的修改方案，我确认后再执行"
```

---

## 多 Agent 工作流

Claude Code 支持启动子 Agent 并行处理任务。

### 典型场景

```
# 在一个 Claude 实例中，让它协调多个子任务
"同时完成以下三件事：
1. 为 src/api/ 下所有 endpoint 生成测试
2. 更新 README 中的 API 文档
3. 检查所有 TODO 注释并创建 GitHub Issues"
```

### 配合 Git Worktree 的并行工作流

```bash
# 终端 1：主分支，处理 hotfix
cd ~/project && claude
> "修复 issue #234，用户登录失败的 bug"

# 终端 2：feature 分支，开发新功能
cd ~/project-feat && claude
> "实现用户头像上传功能，参考 CLAUDE.md 中的规范"

# 终端 3：文档分支
cd ~/project-docs && claude
> "根据最新代码更新 API 文档"
```

---

## 其他实用技巧

### 权限模式

```bash
# 启动时指定权限模式
claude --dangerously-skip-permissions  # 跳过所有权限确认（CI/自动化场景）

# 或在对话中用 Shift+Tab 切换 auto-accept
```

### 继续上次对话

```bash
claude --continue          # 继续最近一次对话
claude --resume <id>       # 继续指定 ID 的对话
```

### 非交互模式（脚本集成）

```bash
# 直接执行任务，不进入交互模式
claude -p "分析 src/ 目录，输出所有 TODO 注释的列表"

# 管道输入
cat error.log | claude -p "分析这个错误日志，找出根本原因"
```

### 环境变量配置

```bash
# ~/.zshrc 或 ~/.bashrc
export ANTHROPIC_API_KEY="sk-ant-..."
export CLAUDE_MODEL="claude-opus-4-6"  # 默认使用 Opus
```

### 查看费用

```
/cost   # 查看当前会话的 token 消耗和费用估算
```

### 调试模式

```bash
claude --debug   # 输出详细的调试信息，排查问题时有用
```

---

## 推荐工作流总结

```
1. 项目初始化
   └── /init → 生成 CLAUDE.md → 补充项目规范

2. 日常开发
   └── git worktree 创建隔离环境
   └── 启动 Claude Code，给出明确任务
   └── Shift+Tab 开启 auto-accept（信任任务时）
   └── /compact 定期压缩上下文

3. 代码质量
   └── /review 代码审查
   └── /write-tests 生成测试
   └── hooks 自动格式化

4. 提交发布
   └── /commit 生成规范提交信息
   └── /pr 创建 Pull Request
```

---

*最后更新：2025-03 | 基于 Claude Code CLI*
