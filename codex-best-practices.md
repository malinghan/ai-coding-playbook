# OpenAI Codex CLI 使用技巧与最佳实践

> Codex CLI 是 OpenAI 推出的终端编程 Agent，基于 o3/o4-mini 模型，支持沙箱执行、多模态输入，适合自动化复杂编程任务。

---

## 目录

1. [安装与配置](#安装与配置)
2. [三种审批模式](#三种审批模式)
3. [快捷键](#快捷键)
4. [AGENTS.md — 项目说明](#agentsmd--项目说明)
5. [Context 管理](#context-管理)
6. [多模态输入](#多模态输入)
7. [沙箱机制](#沙箱机制)
8. [模型选择](#模型选择)
9. [Git 工作流集成](#git-工作流集成)
10. [提示词技巧](#提示词技巧)
11. [实用配置](#实用配置)

---

## 安装与配置

```bash
# 安装
npm install -g @openai/codex

# 设置 API Key
export OPENAI_API_KEY="sk-..."

# 启动（交互模式）
codex

# 直接执行任务（非交互）
codex "为 src/utils.ts 写单元测试"

# 指定工作目录
codex --cwd ~/project "重构 auth 模块"
```

### 配置文件

`~/.codex/config.json`：

```json
{
  "model": "o4-mini",
  "approvalMode": "suggest",
  "notify": true
}
```

---

## 三种审批模式

理解审批模式是用好 Codex 的核心。

### `suggest`（默认）

- 每一步操作都需要手动确认
- 最安全，适合不熟悉任务或敏感代码库
- 可以在执行前修改 AI 的计划

```bash
codex --approval-mode suggest "重构数据库连接层"
```

### `auto-edit`

- 文件读写自动批准，shell 命令仍需确认
- 适合日常编码任务，平衡效率和安全
- 推荐大多数场景使用

```bash
codex --approval-mode auto-edit "添加输入验证到所有 API 路由"
```

### `full-auto`

- 所有操作自动批准，包括 shell 命令
- 适合完全信任的任务、CI 环境、脚本集成
- 配合沙箱使用更安全

```bash
codex --approval-mode full-auto "运行测试并修复所有失败的用例"

# 等价的危险标志（不推荐，语义不清晰）
codex --dangerously-auto-approve-everything "..."
```

**选择原则：**
```
不熟悉任务        → suggest
日常编码          → auto-edit
CI / 自动化脚本   → full-auto（配合沙箱）
```

---

## 快捷键

| 快捷键 | 作用 |
|--------|------|
| `Enter` | 确认/执行当前步骤 |
| `n` / `N` | 拒绝当前步骤 |
| `e` | 编辑 AI 的计划后再执行 |
| `a` | 批准本次会话所有后续步骤 |
| `Ctrl+C` | 中断当前操作 |
| `Ctrl+D` | 退出 Codex |
| `↑` / `↓` | 浏览历史输入 |

---

## AGENTS.md — 项目说明

`AGENTS.md` 是 Codex 的项目配置文件，启动时自动加载，告诉 AI 项目规范和注意事项。

### 放在哪里

- `./AGENTS.md` — 项目根目录（推荐提交到 git）
- `~/.codex/AGENTS.md` — 全局个人配置
- `./subdir/AGENTS.md` — 子目录级别说明

### 写什么

```markdown
# 项目说明

## 技术栈
- Python 3.11 + FastAPI
- PostgreSQL + SQLAlchemy（async）
- 测试：pytest + httpx

## 开发规范
- 使用 uv 管理依赖，不用 pip
- 所有数据库操作必须在 repository 层
- API 响应统一用 Pydantic 模型
- 遵循 PEP 8，行宽 88（black 格式化）

## 常用命令
- `uv run pytest` — 运行测试
- `uv run uvicorn app.main:app --reload` — 启动开发服务器
- `uv run alembic upgrade head` — 执行数据库迁移

## 禁止事项
- 不直接修改 alembic/versions/ 下的迁移文件
- 不在 API 层写业务逻辑
- 不提交 .env 文件
- 不使用同步数据库操作（必须 async）

## 测试规范
- 测试文件放在 tests/ 目录，镜像 app/ 结构
- 使用 pytest fixtures，不在测试中直接连接真实数据库
- 每个测试函数只测一个行为
```

### 与 Claude Code 的 CLAUDE.md 对比

| | AGENTS.md (Codex) | CLAUDE.md (Claude Code) |
|--|-------------------|------------------------|
| 格式 | Markdown | Markdown |
| 自动加载 | ✅ | ✅ |
| 子目录支持 | ✅ | ✅ |
| 全局配置 | `~/.codex/AGENTS.md` | `~/.claude/CLAUDE.md` |

---

## Context 管理

### 传入文件和目录

```bash
# 传入单个文件
codex --context src/auth.py "重构这个文件的错误处理"

# 传入多个文件
codex --context src/models.py --context src/schemas.py "保持两个文件的类型定义一致"

# 传入整个目录
codex --context src/api/ "为所有 endpoint 添加请求日志"
```

### 在交互模式中引用

```
# 直接在提示词中提到文件路径，Codex 会自动读取
"查看 src/auth/login.py 第 45 行附近的逻辑，修复 token 过期的处理"
```

### 控制 context 大小

大型项目中，过多 context 会降低质量：

```bash
# 用 .codexignore 排除不相关文件
echo "node_modules/\ndist/\n*.lock\ncoverage/" > .codexignore

# 精确指定相关文件，而不是整个项目
codex --context src/payments/ "修复支付回调的幂等性问题"
```

---

## 多模态输入

Codex 支持传入图片，适合根据设计稿写代码或分析截图。

```bash
# 传入截图，根据 UI 写代码
codex --image screenshot.png "根据这个设计稿实现 React 组件"

# 传入错误截图
codex --image error-screenshot.png "分析这个错误，找出原因并修复"

# 传入架构图
codex --image architecture.png "根据这个架构图，搭建对应的目录结构和基础代码"
```

### 实用场景

- 把 Figma 截图转成组件代码
- 把报错截图直接丢给 Codex 分析
- 根据数据库 ER 图生成 ORM 模型
- 把手绘流程图转成代码逻辑

---

## 沙箱机制

Codex 在 `full-auto` 模式下默认启用沙箱，隔离文件系统和网络访问。

### 沙箱的作用

- 防止 AI 意外修改项目范围外的文件
- 阻止未授权的网络请求
- 提供可重现的执行环境

### 配置沙箱

```json
// ~/.codex/config.json
{
  "sandbox": {
    "enabled": true,
    "allowNetwork": false,
    "allowedPaths": ["/home/user/project"],
    "blockedPaths": ["/home/user/.ssh", "/etc"]
  }
}
```

### 关闭沙箱（谨慎）

```bash
# 需要网络访问时（如安装依赖）
codex --no-sandbox "安装项目依赖并运行测试"
```

---

## 模型选择

| 模型 | 特点 | 适用场景 |
|------|------|----------|
| `o4-mini` | 速度快，性价比高 | 日常编码任务（推荐默认） |
| `o3` | 推理能力强，较慢 | 复杂算法、架构设计、难 bug |
| `o3-mini` | 平衡速度和推理 | 中等复杂度任务 |

```bash
# 指定模型
codex --model o3 "设计一个分布式锁的实现方案"

# 在配置文件中设置默认模型
# ~/.codex/config.json: { "model": "o4-mini" }
```

**选择原则：**
```
日常编码、写测试、小重构   → o4-mini
复杂业务逻辑、性能优化     → o3-mini
架构设计、算法、疑难 bug   → o3
```

---

## Git 工作流集成

### 配合 Git Worktree 并行开发

```bash
# 创建 worktree
git worktree add ../proj-feat-payment feature/payment-v2

# 在 worktree 中运行 Codex
cd ../proj-feat-payment
codex --approval-mode auto-edit "实现支付宝支付接入，参考 AGENTS.md 中的规范"

# 主目录同时处理 hotfix
cd ~/proj
codex "修复 issue #89，订单状态更新失败"
```

### 任务完成后自动提交

```bash
# 让 Codex 完成任务后自动提交
codex --approval-mode full-auto "
1. 为 src/api/users.py 添加分页支持
2. 更新对应的测试
3. 完成后用 conventional commits 格式提交
"
```

### 代码审查辅助

```bash
# 审查当前 diff
git diff | codex "审查这些改动，找出潜在问题和改进建议"

# 审查特定 commit
git show HEAD | codex "这个 commit 有没有安全漏洞？"
```

---

## 提示词技巧

### 明确任务边界

```
# 好
"只修改 src/auth/login.py，
把密码验证逻辑提取到单独的函数，
不改变函数签名和返回值"

# 不够好
"优化登录模块"
```

### 分步骤执行复杂任务

```
# 第一步：分析
"分析 src/database/ 目录，列出所有 N+1 查询问题，不要修改代码"

# 确认分析结果后，第二步：修复
"修复你找到的第 1 个和第 2 个 N+1 问题"
```

### 要求先计划再执行

```
"在修改任何代码之前，先列出你的修改计划：
- 需要修改哪些文件
- 每个文件改什么
- 可能的风险点
等我确认后再开始"
```

### 利用测试驱动

```
"用 TDD 方式实现用户注册功能：
1. 先写测试用例（覆盖正常流程和边界情况）
2. 运行测试，确认全部失败
3. 实现功能代码
4. 运行测试，确认全部通过"
```

### 非交互模式脚本集成

```bash
#!/bin/bash
# ci-fix.sh — CI 中自动修复 lint 错误

LINT_OUTPUT=$(pnpm lint 2>&1)

if [ -n "$LINT_OUTPUT" ]; then
  echo "$LINT_OUTPUT" | codex \
    --approval-mode full-auto \
    --model o4-mini \
    "修复以下 lint 错误，不改变代码逻辑"
fi
```

---

## 实用配置

### 完整配置示例

`~/.codex/config.json`：

```json
{
  "model": "o4-mini",
  "approvalMode": "auto-edit",
  "notify": true,
  "sandbox": {
    "enabled": true,
    "allowNetwork": false
  },
  "history": {
    "maxEntries": 100,
    "persist": true
  }
}
```

### 环境变量

```bash
# ~/.zshrc
export OPENAI_API_KEY="sk-..."
export CODEX_MODEL="o4-mini"
export CODEX_APPROVAL_MODE="auto-edit"
```

### 查看历史会话

```bash
codex --list-sessions      # 列出历史会话
codex --resume <session-id> # 继续历史会话
```

---

## 与 Claude Code 对比

| | Codex CLI | Claude Code |
|--|-----------|-------------|
| 背后模型 | o3 / o4-mini | Claude Sonnet / Opus |
| 项目配置文件 | `AGENTS.md` | `CLAUDE.md` |
| 沙箱支持 | ✅ 内置 | ❌ 无 |
| 多模态输入 | ✅ 图片 | ❌ 无 |
| MCP 支持 | 有限 | ✅ 完整 |
| Skills / 自定义命令 | ❌ | ✅ |
| 推理能力 | ✅ o3 强推理 | 一般 |
| 速度 | o4-mini 较快 | Sonnet 较快 |

**推荐组合：**
- 日常编码、需要 MCP/Skills → Claude Code
- 复杂推理、算法、需要沙箱隔离 → Codex CLI
- 根据截图/设计稿写代码 → Codex CLI（多模态）

---

## 推荐工作流总结

```
1. 项目初始化
   └── 创建 AGENTS.md（技术栈、规范、禁止事项）
   └── 配置 .codexignore

2. 日常任务
   └── auto-edit 模式（平衡效率和安全）
   └── 精确指定 context 文件
   └── 分步骤执行复杂任务

3. 自动化场景
   └── full-auto + 沙箱
   └── 非交互模式集成到 CI/脚本

4. 复杂推理任务
   └── 切换到 o3 模型
   └── suggest 模式，逐步确认
```

---

*最后更新：2025-03 | 基于 OpenAI Codex CLI*
