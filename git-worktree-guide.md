# Git Worktree 并行 AI 开发指南

> Git Worktree 让你同时在多个分支上工作，配合 Claude Code 等 AI 工具，可以实现真正的并行多任务开发。

---

## 目录

1. [为什么需要 Worktree](#为什么需要-worktree)
2. [基本操作](#基本操作)
3. [配合 Claude Code 并行开发](#配合-claude-code-并行开发)
4. [推荐目录结构](#推荐目录结构)
5. [实用工作流](#实用工作流)
6. [常见问题](#常见问题)

---

## 为什么需要 Worktree

### 传统方式的痛点

```
# 传统方式：频繁切换分支
git stash          # 保存当前工作
git checkout main  # 切换分支
# 处理 hotfix...
git checkout feature/auth  # 切回来
git stash pop      # 恢复工作
# 上下文已经断了，需要重新回忆
```

### Worktree 的优势

```
# Worktree 方式：多个目录同时工作
~/project/          ← 主分支，处理 hotfix
~/project-feat/     ← feature 分支，开发新功能
~/project-docs/     ← docs 分支，更新文档

# 每个目录独立，互不干扰
# 每个目录可以跑一个 Claude Code 实例
```

**配合 AI 工具的核心优势：**
- 每个 Claude Code 实例有独立的上下文，不会混淆
- 可以同时让多个 AI 实例并行工作
- 避免频繁切换导致的上下文丢失

---

## 基本操作

### 创建 Worktree

```bash
# 基于现有分支创建 worktree
git worktree add ../project-hotfix hotfix/login-fix

# 创建新分支并建立 worktree
git worktree add ../project-feat feature/user-avatar

# 基于特定 commit 创建（只读，用于代码审查）
git worktree add --detach ../project-review abc1234
```

### 查看和管理

```bash
# 查看所有 worktree
git worktree list

# 输出示例：
# /Users/me/project        abc1234 [main]
# /Users/me/project-feat   def5678 [feature/user-avatar]
# /Users/me/project-hotfix ghi9012 [hotfix/login-fix]
```

### 删除 Worktree

```bash
# 删除 worktree（不删除分支）
git worktree remove ../project-feat

# 强制删除（有未提交更改时）
git worktree remove --force ../project-feat

# 清理已删除目录的 worktree 记录
git worktree prune
```

---

## 配合 Claude Code 并行开发

### 基本模式

```bash
# 终端 1：主分支，处理紧急 bug
cd ~/project
claude
> "修复 issue #234，用户登录后 session 没有正确保存"

# 终端 2：feature 分支，开发新功能
cd ~/project-feat
claude
> "实现用户头像上传功能，参考 CLAUDE.md 中的规范"

# 终端 3：文档分支
cd ~/project-docs
claude
> "根据最新的 API 变更更新 README 和 API 文档"
```

### 让 Claude 自动创建 Worktree

在 Claude Code 中直接说：

```
"为 feature/payment-v2 创建一个 worktree，
路径放在 ../project-payment，
然后在那个目录里实现支付宝支付接入"
```

Claude 会执行：
```bash
git worktree add ../project-payment feature/payment-v2
cd ../project-payment
# 然后开始实现功能
```

### 并行任务分配策略

```
适合并行的任务：
✅ 独立功能模块（互不依赖的文件）
✅ 文档更新（不影响代码）
✅ 测试编写（基于已有接口）
✅ Bug fix（影响范围小）

不适合并行的任务：
❌ 修改同一个文件
❌ 有依赖关系的功能（A 依赖 B 的输出）
❌ 数据库 schema 变更（可能冲突）
```

---

## 推荐目录结构

### 方案 1：兄弟目录（推荐）

```
~/projects/
  my-app/           ← 主目录（main 分支）
  my-app-feat/      ← feature 分支
  my-app-hotfix/    ← hotfix 分支
  my-app-docs/      ← docs 分支
```

```bash
# 创建
git worktree add ../my-app-feat feature/new-feature
git worktree add ../my-app-hotfix hotfix/critical-fix
```

### 方案 2：子目录（适合多 worktree 管理）

```
~/projects/
  my-app/
    .git/
    worktrees/      ← 所有 worktree 放这里
      feat-auth/
      feat-payment/
      hotfix-login/
    src/
    ...
```

```bash
# 创建
mkdir -p worktrees
git worktree add worktrees/feat-auth feature/auth-refactor
git worktree add worktrees/feat-payment feature/payment-v2
```

---

## 实用工作流

### 工作流 1：功能开发 + 紧急修复并行

```bash
# 正在开发新功能
git worktree add ../proj-feat feature/user-dashboard
cd ../proj-feat
claude
> "实现用户仪表盘，包括统计图表和最近活动"

# 突然来了紧急 bug
# 不需要中断当前工作！
cd ~/proj  # 回到主目录
git worktree add ../proj-hotfix hotfix/payment-error
cd ../proj-hotfix
claude
> "修复支付失败后订单状态没有回滚的问题，这是 P0 bug"

# 两个 Claude 实例同时工作
# hotfix 完成后合并，不影响 feature 开发
```

### 工作流 2：代码审查

```bash
# 创建只读 worktree 用于审查 PR
git fetch origin pull/156/head:pr-156
git worktree add --detach ../proj-review pr-156

cd ../proj-review
claude
> "审查这个 PR 的代码，重点检查：
  1. 安全漏洞
  2. 性能问题
  3. 是否符合项目规范（参考 CLAUDE.md）
  输出详细的审查报告"

# 审查完成后清理
git worktree remove ../proj-review
```

### 工作流 3：多功能并行开发

```bash
# 同时开发 3 个独立功能
git worktree add ../proj-auth feature/oauth-integration
git worktree add ../proj-search feature/full-text-search
git worktree add ../proj-notify feature/push-notifications

# 三个终端，三个 Claude 实例
# 终端 1
cd ../proj-auth && claude
> "集成 Google OAuth，参考 src/auth/ 现有实现"

# 终端 2
cd ../proj-search && claude
> "用 Elasticsearch 实现全文搜索，支持中文分词"

# 终端 3
cd ../proj-notify && claude
> "实现 Web Push 通知，支持订阅管理"
```

### 工作流 4：实验性重构

```bash
# 创建实验分支，不影响主开发
git worktree add ../proj-experiment experiment/new-architecture

cd ../proj-experiment
claude
> "把现有的 MVC 架构重构为 Clean Architecture，
  先重构 src/users/ 模块作为示例，
  如果效果好我们再推广到其他模块"

# 实验成功 → 合并
# 实验失败 → 直接删除 worktree，主分支不受影响
git worktree remove ../proj-experiment
git branch -D experiment/new-architecture
```

---

## 常见问题

### Q：多个 worktree 共享 node_modules 吗？

不共享。每个 worktree 是独立的工作目录，需要单独安装依赖：

```bash
cd ../proj-feat
pnpm install  # 需要单独安装
```

**优化方案：** 用 pnpm 的 workspace 或符号链接共享 node_modules（高级用法）。

### Q：worktree 之间可以共享 .env 文件吗？

可以用符号链接：

```bash
cd ../proj-feat
ln -s ../proj/.env.local .env.local
```

### Q：如何快速在多个 worktree 间切换？

```bash
# 在 ~/.zshrc 中添加函数
function wt() {
  local base=$(git worktree list | head -1 | awk '{print $1}')
  local base_name=$(basename $base)
  git worktree list | grep -v "^$base" | fzf | awk '{print $1}' | xargs -I{} cd {}
}
```

或者用简单的别名：

```bash
alias proj='cd ~/project'
alias proj-feat='cd ~/project-feat'
alias proj-hotfix='cd ~/project-hotfix'
```

### Q：worktree 中的 Claude Code 会读取主目录的 CLAUDE.md 吗？

会。Claude Code 会读取 git 仓库根目录的 CLAUDE.md，而所有 worktree 共享同一个 git 仓库，所以 CLAUDE.md 是共享的。

### Q：如何避免多个 Claude 实例同时修改同一个文件？

任务分配时注意文件边界：

```
# 好的分配（文件不重叠）
Worktree 1：修改 src/auth/
Worktree 2：修改 src/payments/
Worktree 3：修改 src/notifications/

# 危险的分配（可能冲突）
Worktree 1：修改 src/utils/helpers.ts
Worktree 2：也修改 src/utils/helpers.ts
```

---

## 快速参考

```bash
# 创建 worktree
git worktree add <路径> <分支名>
git worktree add <路径> -b <新分支名>  # 同时创建新分支

# 查看
git worktree list

# 删除
git worktree remove <路径>
git worktree prune  # 清理失效记录

# 典型并行开发启动
git worktree add ../$(basename $PWD)-feat feature/my-feature
cd ../$(basename $PWD)-feat
pnpm install
claude
```

---

*最后更新：2026-03 | 适用于 Git 2.5+*
