# Aider 使用技巧与最佳实践

> Aider 是开源的终端 AI 编程工具，直接在你的 git 仓库中工作，每次修改自动提交，支持几乎所有主流 LLM。

---

## 目录

1. [安装与配置](#安装与配置)
2. [核心工作模式](#核心工作模式)
3. [文件管理（/add、/drop）](#文件管理adddrop)
4. [模型选择](#模型选择)
5. [Git 集成](#git-集成)
6. [配置文件](#配置文件)
7. [提示词技巧](#提示词技巧)
8. [高级功能](#高级功能)
9. [与其他工具对比](#与其他工具对比)

---

## 安装与配置

```bash
# 安装
pip install aider-chat
# 或用 uv（推荐）
uv tool install aider-chat

# 设置 API Key
export ANTHROPIC_API_KEY="sk-ant-..."
export OPENAI_API_KEY="sk-..."

# 启动（使用 Claude Sonnet，推荐）
aider --model claude-sonnet-4-5

# 启动（使用 GPT-4o）
aider --model gpt-4o

# 在已有项目中启动
cd my-project
aider
```

### 首次使用

```bash
# Aider 会自动检测 git 仓库
# 如果不在 git 仓库中，会提示初始化
git init
aider
```

---

## 核心工作模式

Aider 有三种对话模式，用 `/chat-mode` 切换：

### `code` 模式（默认）

- AI 直接修改代码文件
- 每次修改自动创建 git commit
- 适合：实现功能、修复 bug、重构

### `ask` 模式

- 纯对话，不修改文件
- 适合：理解代码、探索方案、提问

### `architect` 模式

- 先用强模型规划，再用弱模型执行
- 适合：复杂功能设计，平衡质量和成本

```bash
# 切换模式
/chat-mode ask
/chat-mode code
/chat-mode architect
```

---

## 文件管理（/add、/drop）

Aider 只会修改你明确添加到上下文的文件，这是它与其他工具的重要区别。

### 添加文件

```bash
# 添加单个文件
/add src/auth/login.py

# 添加多个文件
/add src/auth/login.py src/auth/types.py

# 添加整个目录
/add src/api/

# 添加 glob 匹配的文件
/add src/**/*.test.ts

# 只读模式添加（AI 可以读取但不修改）
/read-only docs/api-spec.md
/read-only CLAUDE.md
```

### 移除文件

```bash
# 移除文件（减少 token 消耗）
/drop src/auth/login.py

# 移除所有文件
/drop
```

### 查看当前上下文

```bash
/files    # 查看已添加的文件列表
/tokens   # 查看当前 token 使用量
```

### 最佳实践

```
# 只添加真正需要修改的文件
# 用 /read-only 添加参考文件（规范、类型定义等）
# 任务完成后 /drop 清理，开始新任务时重新 /add
```

---

## 模型选择

Aider 支持几乎所有主流 LLM，通过 `--model` 参数指定：

| 模型 | 命令 | 适用场景 |
|------|------|----------|
| Claude Sonnet 4.5 | `--model claude-sonnet-4-5` | 日常编码（推荐） |
| Claude Opus 4 | `--model claude-opus-4-5` | 复杂架构、疑难 bug |
| GPT-4o | `--model gpt-4o` | 均衡，速度快 |
| o3-mini | `--model o3-mini` | 算法、推理密集型 |
| DeepSeek Coder | `--model deepseek/deepseek-coder` | 成本低，代码能力强 |
| 本地模型（Ollama） | `--model ollama/codellama` | 离线、隐私敏感场景 |

```bash
# 启动时指定模型
aider --model claude-sonnet-4-5

# architect 模式：规划用强模型，执行用弱模型
aider --model claude-opus-4-5 --editor-model claude-sonnet-4-5

# 在对话中切换模型
/model claude-opus-4-5
```

**选择原则：**
```
日常编码          → claude-sonnet-4-5（性价比最高）
复杂推理/架构     → claude-opus-4-5
成本敏感          → deepseek/deepseek-coder
离线/隐私         → ollama/本地模型
```

---

## Git 集成

Aider 与 git 深度集成，每次代码修改都会自动提交。

### 自动提交

```bash
# 默认行为：每次修改后自动 git commit
# commit message 由 AI 自动生成

# 禁用自动提交（手动管理）
aider --no-auto-commits

# 查看 Aider 创建的提交
git log --oneline
```

### 撤销修改

```bash
# 撤销最近一次 Aider 的修改
/undo

# 查看 diff
/diff
```

### 配合 Git Worktree 并行开发

```bash
# 创建 worktree
git worktree add ../proj-feat-auth feature/auth-refactor

# 在 worktree 中运行 Aider
cd ../proj-feat-auth
aider --model claude-sonnet-4-5

# 主目录同时处理 hotfix
cd ~/proj
aider
```

---

## 配置文件

### `.aider.conf.yml`（项目级配置）

在项目根目录创建，提交到 git 供团队共享：

```yaml
# .aider.conf.yml
model: claude-sonnet-4-5
auto-commits: true
dirty-commits: false
show-diffs: true

# 启动时自动添加的文件
read: CLAUDE.md  # 只读，作为项目规范参考
```

### `.aiderignore`（排除文件）

类似 `.gitignore`，阻止 Aider 修改特定文件：

```
node_modules/
dist/
.next/
coverage/
*.lock
.env
secrets/
migrations/  # 不让 AI 直接修改迁移文件
```

### 全局配置

`~/.aider.conf.yml`：

```yaml
model: claude-sonnet-4-5
dark-mode: true
show-diffs: true
```

---

## 提示词技巧

### 明确指定文件后再提任务

```bash
# 先添加文件，再描述任务
/add src/auth/login.py src/auth/middleware.py
"把 JWT 验证逻辑从 login.py 提取到 middleware.py，
保持现有 API 兼容，不改变函数签名"
```

### 先用 ask 模式理解，再用 code 模式修改

```bash
/chat-mode ask
"解释 src/queue/processor.py 的消息重试逻辑"

# 理解后切换到 code 模式
/chat-mode code
"优化重试逻辑，添加指数退避"
```

### 要求先计划再执行

```
"在修改任何代码之前，先列出：
1. 需要修改哪些文件
2. 每个文件的具体改动
3. 可能的风险点
等我确认后再开始"
```

### 利用 /read-only 提供参考

```bash
# 把规范文档作为只读参考
/read-only docs/api-design-guide.md
/read-only src/types/index.ts  # 类型定义参考

# 然后让 AI 按规范实现
"按照 api-design-guide.md 中的规范，实现用户管理的 CRUD 接口"
```

### 分步骤处理复杂任务

```bash
# 第一步：只分析
/chat-mode ask
/add src/
"找出所有 N+1 查询问题，列出文件和行号"

# 第二步：逐个修复
/chat-mode code
/drop
/add src/users/repository.py
"修复 get_users_with_posts 函数中的 N+1 问题"
```

---

## 高级功能

### `/run` — 执行命令并反馈

```bash
# 运行测试，让 AI 根据结果修复
/run pytest tests/test_auth.py

# 运行 lint，让 AI 修复问题
/run ruff check src/

# 运行构建，让 AI 修复错误
/run npm run build
```

Aider 会自动读取命令输出，并根据错误信息修复代码。

### `/paste` — 粘贴外部内容

```bash
# 粘贴错误日志
/paste
# 然后粘贴内容，Ctrl+D 结束
"根据这个错误日志，找出并修复问题"
```

### `/web` — 抓取网页内容

```bash
# 抓取文档页面作为参考
/web https://docs.example.com/api-reference
"按照这个 API 文档，实现对应的客户端"
```

### `--watch` 模式

监听文件变化，自动触发 AI 处理带有 `AI!` 注释的代码：

```bash
aider --watch-files
```

在代码中添加注释触发 AI：

```python
def process_payment(amount: float, currency: str):
    # AI! 添加输入验证：amount 必须大于 0，currency 必须是有效的 ISO 4217 代码
    pass
```

保存文件后，Aider 自动检测并实现注释描述的功能。

### 非交互模式（脚本集成）

```bash
# 直接执行任务，不进入交互模式
aider --message "为 src/utils.py 中的所有函数添加类型注解" src/utils.py

# 管道输入
echo "修复所有 lint 错误" | aider --model claude-sonnet-4-5 src/

# CI 中自动修复
aider --yes --no-auto-commits \
  --message "修复 mypy 类型错误" \
  $(mypy src/ 2>&1 | grep "error:" | awk -F: '{print $1}' | sort -u)
```

---

## 与其他工具对比

| | Aider | Claude Code | Cursor |
|--|-------|-------------|--------|
| 类型 | 终端 CLI | 终端 CLI | IDE |
| 开源 | ✅ | ❌ | ❌ |
| 自动 git commit | ✅ 每次修改 | 手动 | 手动 |
| 多模型支持 | ✅ 几乎所有 | Claude only | 多模型 |
| 本地模型 | ✅ Ollama | ❌ | 有限 |
| 项目配置 | `.aider.conf.yml` | `CLAUDE.md` | `.cursor/rules/` |
| 文件控制 | 手动 /add | 自动感知 | 自动感知 |
| 价格 | 按 token（自带 key） | 按 token | $20/月 |

**推荐场景：**
- 想用自己的 API key，控制成本 → Aider
- 需要本地模型（隐私/离线） → Aider + Ollama
- 开源项目，想要透明可审计的工具 → Aider
- 需要最强 Agent 能力 → Claude Code

---

## 推荐工作流总结

```
1. 项目初始化
   └── 创建 .aider.conf.yml（指定默认模型）
   └── 创建 .aiderignore
   └── /read-only CLAUDE.md（加载项目规范）

2. 日常任务
   └── /add 只添加需要修改的文件
   └── 先 ask 模式理解，再 code 模式修改
   └── /run 运行测试，让 AI 自动修复

3. 复杂任务
   └── architect 模式（强模型规划 + 弱模型执行）
   └── 分步骤，每步 /drop 清理上下文
   └── 要求先计划再执行

4. 自动化
   └── --watch 模式监听文件变化
   └── --message 非交互模式集成到脚本/CI
   └── /run 命令输出自动反馈给 AI
```

---

*最后更新：2026-03 | 基于 Aider 最新版*
