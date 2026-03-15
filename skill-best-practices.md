# Skills（自定义命令）使用技巧与最佳实践

> Skills 是 Claude Code 的可复用提示词模板系统，用 `/skill-name` 触发，让你把常用工作流封装成一条命令。

---

## 目录

1. [什么是 Skills](#什么是-skills)
2. [内置 Skills 一览](#内置-skills-一览)
3. [创建自定义 Skill](#创建自定义-skill)
4. [Skill 文件格式与技巧](#skill-文件格式与技巧)
5. [参数传递](#参数传递)
6. [团队共享 Skills](#团队共享-skills)
7. [实用 Skill 示例库](#实用-skill-示例库)
8. [提示词技巧](#提示词技巧)
9. [与 Hooks 配合使用](#与-hooks-配合使用)

---

## 什么是 Skills

Skills 本质上是存放在特定目录的 Markdown 文件，Claude Code 在触发时会把文件内容作为提示词注入对话。

```
用户输入 /deploy-check
    ↓
Claude Code 读取 ~/.claude/commands/deploy-check.md
    ↓
把文件内容作为提示词发送给 Claude
    ↓
Claude 按照 Skill 定义的步骤执行任务
```

**核心价值：**
- 把重复性工作流封装成一条命令
- 团队统一 AI 行为，减少提示词差异
- 积累最佳实践，避免每次重新描述

---

## 内置 Skills 一览

Claude Code 预置了以下常用 Skills：

| Skill | 触发命令 | 说明 |
|-------|----------|------|
| commit | `/commit` | 分析 diff，生成规范的 git commit message |
| review-code | `/review-code` | 全面代码审查，关注质量和潜在问题 |
| write-tests | `/write-tests` | 为当前代码生成测试用例 |
| explain-code | `/explain-code` | 用图表和类比解释代码工作原理 |
| simplify | `/simplify` | 审查代码质量，重构优化 |
| claude-api | `/claude-api` | 使用 Claude API / Anthropic SDK 构建应用 |
| frontend-design | `/frontend-design` | 创建高质量前端界面 |

---

## 创建自定义 Skill

### 目录结构

```bash
# 个人全局 Skills（所有项目可用）
~/.claude/commands/

# 项目级 Skills（仅当前项目，可提交到 git）
.claude/commands/
```

### 创建第一个 Skill

```bash
# 创建目录
mkdir -p ~/.claude/commands

# 创建 Skill 文件
touch ~/.claude/commands/my-skill.md
```

`~/.claude/commands/my-skill.md`:
```markdown
分析当前项目的代码质量，输出报告：
1. 找出所有超过 100 行的函数
2. 找出重复代码片段
3. 找出缺少错误处理的地方
4. 用表格格式输出：问题类型 | 文件位置 | 严重程度 | 建议
```

使用时输入 `/my-skill` 即可触发。

---

## Skill 文件格式与技巧

### 基本结构

```markdown
# Skill 名称（可选，作为标题）

## 背景说明（可选）
简短描述这个 Skill 的用途和适用场景。

## 执行步骤
1. 第一步：做什么
2. 第二步：做什么
3. 第三步：输出什么格式

## 约束条件
- 不要修改 xxx 文件
- 只关注 src/ 目录
- 输出用中文
```

### 让 Skill 更精准的技巧

**明确输出格式：**
```markdown
输出格式要求：
- 用 Markdown 表格展示结果
- 每个问题单独一行
- 严重程度分为：高 / 中 / 低
```

**指定范围边界：**
```markdown
只分析以下范围：
- src/ 目录下的 TypeScript 文件
- 不包括 *.test.ts 和 *.spec.ts
- 不包括自动生成的文件（*.generated.ts）
```

**加入项目上下文：**
```markdown
项目技术栈：Next.js + TypeScript + Prisma
测试框架：Vitest
包管理器：pnpm
```

---

## 参数传递

在触发 Skill 时可以附加参数，Claude 会把参数作为额外上下文：

```bash
# 不带参数
/commit

# 带参数（描述本次提交的背景）
/commit 修复了用户登录后 token 没有刷新的问题

# 带文件路径
/write-tests src/api/users.ts

# 带 issue 编号
/commit fixes #234
```

在 Skill 文件中可以引用参数：

```markdown
# commit

分析当前 git diff，生成符合 Conventional Commits 规范的提交信息。

如果用户提供了额外说明（$ARGUMENTS），将其作为提交信息的补充背景。

要求：
- type: feat/fix/docs/refactor/test/chore
- scope: 影响的模块（可选）
- 简短描述不超过 72 字符
- 如有必要，添加详细说明
```

---

## 团队共享 Skills

把项目级 Skills 提交到 git，团队成员自动获得相同的工作流：

```bash
# 项目目录结构
my-project/
  .claude/
    commands/
      deploy-check.md    # 部署前检查
      db-migrate.md      # 数据库迁移辅助
      api-review.md      # API 设计审查
  src/
  ...
```

```bash
# 提交到 git
git add .claude/commands/
git commit -m "chore: add team claude skills"
```

**团队 Skills 的好处：**
- 新成员 clone 项目后立即获得所有工作流
- 统一代码审查标准
- 把团队最佳实践固化到工具中

---

## 实用 Skill 示例库

### `/deploy-check` — 部署前检查

`.claude/commands/deploy-check.md`:
```markdown
在部署前执行完整检查，按顺序完成以下步骤：

1. 运行 `pnpm test --run`，确认所有测试通过
2. 运行 `pnpm build`，确认构建成功，无 TypeScript 错误
3. 检查是否有未提交的更改（git status）
4. 检查 .env.example 和 .env 的 key 是否一致
5. 检查 package.json 中是否有 `console.log` 或调试代码
6. 输出检查报告，列出所有问题和建议
```

### `/api-review` — API 设计审查

`.claude/commands/api-review.md`:
```markdown
审查当前文件或指定文件的 API 设计，检查以下方面：

## 检查项
- RESTful 规范：URL 命名、HTTP 方法使用是否正确
- 输入验证：所有参数是否有验证，是否防止注入
- 错误处理：是否有统一的错误响应格式
- 认证授权：敏感接口是否有权限检查
- 响应格式：是否一致，是否有分页支持
- 性能：是否有 N+1 查询风险

## 输出格式
用表格列出所有问题：
| 问题描述 | 位置 | 严重程度 | 修复建议 |
```

### `/changelog` — 生成 Changelog

`.claude/commands/changelog.md`:
```markdown
根据 git log 生成 CHANGELOG 条目。

步骤：
1. 运行 `git log --oneline` 查看最近的提交
2. 按类型分组：Features、Bug Fixes、Breaking Changes、Other
3. 过滤掉 chore、docs 等不面向用户的提交
4. 用 Keep a Changelog 格式输出

输出示例：
## [版本号] - 日期

### Features
- 新增用户头像上传功能

### Bug Fixes
- 修复登录后 token 未刷新的问题
```

### `/security-check` — 安全审查

`.claude/commands/security-check.md`:
```markdown
对当前代码库进行安全审查，重点检查 OWASP Top 10：

1. SQL 注入：检查所有数据库查询是否使用参数化查询
2. XSS：检查所有用户输入是否有转义处理
3. 认证问题：检查 JWT/Session 处理是否安全
4. 敏感信息泄露：检查是否有硬编码的密钥、密码
5. 权限控制：检查 API 是否有适当的权限验证
6. 依赖漏洞：运行 `pnpm audit` 检查已知漏洞

输出：按严重程度排序的问题列表，每个问题附带修复建议。
```

### `/refactor-plan` — 重构规划

`.claude/commands/refactor-plan.md`:
```markdown
在开始重构之前，先制定详细计划。

分析目标代码，输出：

## 现状分析
- 当前代码的主要问题
- 技术债务评估

## 重构方案
- 需要修改的文件列表
- 每个文件的具体改动
- 改动的先后顺序（依赖关系）

## 风险评估
- 可能影响的功能
- 需要更新的测试
- 向后兼容性考虑

## 验证方案
- 如何验证重构后功能正常

等待确认后再开始执行。
```

---

## 提示词技巧

### 在 Skill 中使用条件逻辑

```markdown
# write-tests

为指定文件生成测试用例。

如果用户指定了文件路径（$ARGUMENTS），只为该文件生成测试。
如果没有指定，分析当前对话上下文中最近修改的文件。

测试要求：
- 使用 Vitest + Testing Library
- 覆盖正常流程、边界条件、错误情况
- Mock 所有外部依赖
- 测试描述用中文
```

### 让 Skill 先分析再执行

```markdown
# 重要：在执行任何修改之前，先输出你的计划
# 等待用户确认后再开始

分析 src/auth/ 目录，制定重构方案...
```

### 引用项目规范

```markdown
# 遵循项目 CLAUDE.md 中定义的规范
# 如果 CLAUDE.md 不存在，使用通用最佳实践

生成符合项目规范的代码...
```

---

## 与 Hooks 配合使用

Skills 和 Hooks 可以组合，实现更强大的自动化：

```json
// .claude/settings.json
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
    ]
  }
}
```

配合 `/write-tests` Skill：
```
1. 触发 /write-tests
2. Claude 生成测试文件（Write 工具）
3. Hook 自动运行 prettier 格式化
4. 测试文件直接符合代码风格规范
```

---

## 推荐工作流总结

```
1. 初始化
   └── mkdir -p ~/.claude/commands（个人全局）
   └── mkdir -p .claude/commands（项目级）
   └── 把常用工作流写成 Skill

2. 日常使用
   └── /commit — 每次提交
   └── /review-code — PR 前自查
   └── /write-tests — 新功能完成后
   └── /deploy-check — 发布前

3. 团队协作
   └── 把 .claude/commands/ 提交到 git
   └── 在 CLAUDE.md 中说明可用的 Skills
   └── 定期更新 Skill，固化最佳实践

4. 进阶
   └── Skill + Hooks 组合自动化
   └── 用 $ARGUMENTS 让 Skill 支持参数
   └── 为不同场景创建专用 Skill
```

---

*最后更新：2026-03 | 基于 Claude Code CLI*
