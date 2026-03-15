# CLAUDE.md 深度指南

> CLAUDE.md 是 Claude Code 的"项目说明书"，也是让 AI 真正理解你项目的关键。写好它，能让 Claude 的每次输出都符合你的期望。

---

## 目录

1. [为什么 CLAUDE.md 很重要](#为什么-claudemd-很重要)
2. [文件位置与加载顺序](#文件位置与加载顺序)
3. [内容结构最佳实践](#内容结构最佳实践)
4. [各类项目模板](#各类项目模板)
5. [高级技巧](#高级技巧)
6. [与其他工具的对应关系](#与其他工具的对应关系)

---

## 为什么 CLAUDE.md 很重要

没有 CLAUDE.md，Claude 每次都要重新猜测：
- 你用什么包管理器（npm？yarn？pnpm？bun？）
- 测试怎么运行（`npm test`？`vitest --run`？`pytest`？）
- 代码风格是什么（tabs 还是 spaces？函数式还是 OOP？）
- 哪些文件不能动（legacy 代码？自动生成的文件？）

有了 CLAUDE.md，这些都不需要重复说明。

**效果对比：**
```
没有 CLAUDE.md：
用户："帮我写个测试"
Claude：生成了用 Jest 写的测试
用户："我们用 Vitest，而且要用 --run 不要 watch 模式"
Claude：重新生成
用户："还有我们用 pnpm 不用 npm"
...（反复纠正）

有 CLAUDE.md：
用户："帮我写个测试"
Claude：直接生成符合项目规范的 Vitest 测试，用 pnpm 命令
```

---

## 文件位置与加载顺序

Claude Code 按以下顺序加载 CLAUDE.md，所有文件都会被读取：

```
1. ~/.claude/CLAUDE.md          ← 个人全局偏好（不提交到 git）
2. ~/CLAUDE.md                  ← 用户主目录（可选）
3. ./CLAUDE.md                  ← 项目根目录（提交到 git，团队共享）
4. ./src/CLAUDE.md              ← 子目录（当 Claude 操作该目录时加载）
5. ./src/api/CLAUDE.md          ← 更深层子目录
```

**实践建议：**
- `~/.claude/CLAUDE.md`：个人偏好，如"总是用中文回复"、"我偏好函数式风格"
- `./CLAUDE.md`：项目规范，提交到 git，团队共享
- `./src/legacy/CLAUDE.md`：特殊目录说明，如"这里的代码不要修改"

---

## 内容结构最佳实践

### 必写内容

**1. 技术栈（让 Claude 知道用什么工具）**

```markdown
## 技术栈
- Runtime: Node.js 20
- 语言: TypeScript 5.4
- 框架: Fastify 4
- ORM: Prisma 5 + PostgreSQL 16
- 测试: Vitest + Supertest
- 包管理: pnpm 9（不用 npm/yarn）
```

**2. 常用命令（最重要，避免 Claude 猜错命令）**

```markdown
## 常用命令
- `pnpm dev` — 启动开发服务器（端口 3000）
- `pnpm test --run` — 运行测试（单次，非 watch 模式）
- `pnpm test:coverage` — 运行测试并生成覆盖率报告
- `pnpm build` — 构建生产版本
- `pnpm lint` — 运行 ESLint
- `pnpm db:migrate` — 执行数据库迁移
- `pnpm db:seed` — 填充测试数据
- `pnpm db:studio` — 打开 Prisma Studio
```

**3. 禁止事项（比"要做什么"更重要）**

```markdown
## 禁止事项
- 不修改 `src/legacy/` 目录（历史遗留代码，有专人维护）
- 不直接修改 `prisma/migrations/` 下的文件（用 `pnpm db:migrate` 生成）
- 不提交 `.env` 文件（用 `.env.example` 作为模板）
- 不使用 `any` 类型（用 `unknown` 替代）
- 不在 controller 层写业务逻辑（放到 service 层）
- 不使用 `console.log`（用 `logger` 模块）
```

### 推荐内容

**4. 项目架构（帮助 Claude 理解代码组织）**

```markdown
## 目录结构
```
src/
  api/          # 路由层：只做参数解析和响应格式化
  services/     # 业务逻辑层
  repositories/ # 数据访问层（只有这里可以操作数据库）
  models/       # 类型定义和 Zod schema
  utils/        # 纯函数工具
  middleware/   # Fastify 中间件
tests/          # 测试文件（镜像 src/ 结构）
prisma/         # 数据库 schema 和迁移
```
```

**5. 编码规范**

```markdown
## 编码规范
- 函数优先于类（除非有充分理由用类）
- 错误处理：使用自定义 AppError 类，不直接 throw string
- 异步：统一用 async/await，不用 .then() 链
- 命名：变量/函数用 camelCase，类型/接口用 PascalCase，常量用 UPPER_SNAKE_CASE
- 测试：每个测试只验证一件事，描述用中文
```

**6. 环境说明**

```markdown
## 环境配置
- 开发环境变量在 `.env.local`（不提交）
- 参考 `.env.example` 了解所需变量
- 数据库：本地 PostgreSQL，端口 5432，数据库名 `myapp_dev`
- Redis：本地 Redis，端口 6379
```

---

## 各类项目模板

### Node.js / TypeScript 后端

```markdown
# 项目名称

## 技术栈
- Node.js 20 + TypeScript 5
- Fastify 4（HTTP 框架）
- Prisma 5 + PostgreSQL（数据库）
- Redis（缓存和队列）
- Vitest（测试）
- pnpm（包管理）

## 常用命令
- `pnpm dev` — 启动开发服务器
- `pnpm test --run` — 运行测试（单次）
- `pnpm build` — 构建
- `pnpm lint` — ESLint 检查
- `pnpm db:migrate` — 数据库迁移
- `pnpm db:seed` — 填充测试数据

## 目录结构
- `src/api/` — 路由（只做参数解析）
- `src/services/` — 业务逻辑
- `src/repositories/` — 数据访问
- `src/types/` — 类型定义

## 禁止事项
- 不用 npm/yarn，只用 pnpm
- 不用 any 类型
- 不在 api 层写业务逻辑
- 不直接操作数据库（必须通过 repository）
- 不提交 .env 文件
- 不修改 prisma/migrations/ 下的文件

## 编码规范
- 错误处理用 AppError 类（src/utils/errors.ts）
- 日志用 logger（src/utils/logger.ts），不用 console.log
- 所有 async 函数必须有错误处理
```

### Next.js 全栈

```markdown
# 项目名称

## 技术栈
- Next.js 15（App Router）
- TypeScript 5
- Tailwind CSS 4
- Prisma + PostgreSQL
- NextAuth.js（认证）
- Vitest + Testing Library（测试）
- pnpm

## 常用命令
- `pnpm dev` — 启动开发服务器（端口 3000）
- `pnpm test --run` — 运行测试
- `pnpm build` — 构建
- `pnpm lint` — ESLint + TypeScript 检查

## 目录结构（App Router）
- `app/` — 页面和路由
- `app/api/` — API 路由
- `components/` — React 组件
- `lib/` — 工具函数和配置
- `hooks/` — 自定义 React hooks

## 禁止事项
- 不用 Pages Router（只用 App Router）
- 不在 Server Component 中使用 useState/useEffect
- 不在客户端组件中直接操作数据库
- 不用内联样式（用 Tailwind）
- 不用 any 类型

## 编码规范
- Server Component 优先，只在必要时用 'use client'
- 数据获取在 Server Component 中做
- 表单用 Server Actions
- 样式只用 Tailwind，不用 CSS Modules
```

### Python / FastAPI

```markdown
# 项目名称

## 技术栈
- Python 3.12
- FastAPI（HTTP 框架）
- SQLAlchemy 2.0（async）+ PostgreSQL
- Pydantic v2（数据验证）
- pytest + httpx（测试）
- uv（包管理）

## 常用命令
- `uv run uvicorn app.main:app --reload` — 启动开发服务器
- `uv run pytest` — 运行测试
- `uv run pytest -x` — 遇到第一个失败就停止
- `uv run alembic upgrade head` — 执行数据库迁移
- `uv run alembic revision --autogenerate -m "描述"` — 生成迁移文件
- `uv run ruff check .` — Lint 检查
- `uv run mypy app/` — 类型检查

## 目录结构
- `app/api/` — 路由（只做参数解析和响应格式化）
- `app/services/` — 业务逻辑
- `app/repositories/` — 数据访问
- `app/models/` — SQLAlchemy 模型
- `app/schemas/` — Pydantic schema
- `tests/` — 测试（镜像 app/ 结构）

## 禁止事项
- 不用 pip，只用 uv
- 不用同步数据库操作（必须 async）
- 不在 API 层写业务逻辑
- 不直接在 service 层操作数据库（通过 repository）
- 不修改 alembic/versions/ 下的迁移文件
- 不提交 .env 文件

## 编码规范
- 类型注解必须完整
- 所有 Pydantic 模型用 v2 语法
- 错误处理用自定义 AppException
- 测试用 pytest fixtures，不直接连接真实数据库
```

### React 前端

```markdown
# 项目名称

## 技术栈
- React 18 + TypeScript 5
- Vite（构建工具）
- Tailwind CSS
- React Query v5（数据获取）
- Zustand（状态管理）
- React Hook Form + Zod（表单）
- Vitest + Testing Library（测试）
- pnpm

## 常用命令
- `pnpm dev` — 启动开发服务器
- `pnpm test --run` — 运行测试
- `pnpm build` — 构建
- `pnpm preview` — 预览构建结果

## 目录结构
- `src/components/` — 通用组件
- `src/features/` — 功能模块（每个功能一个目录）
- `src/hooks/` — 自定义 hooks
- `src/stores/` — Zustand stores
- `src/api/` — API 调用函数
- `src/types/` — 类型定义

## 禁止事项
- 不用 class components（只用函数组件）
- 不用内联样式（用 Tailwind）
- 不在组件中直接调用 API（用 React Query hooks）
- 不在组件中写业务逻辑（放到 hooks 或 stores）
- 不用 any 类型

## 编码规范
- 组件文件用 PascalCase（UserCard.tsx）
- hooks 文件用 camelCase，以 use 开头（useUserData.ts）
- 每个功能模块有自己的 index.ts 导出
```

---

## 高级技巧

### 用子目录 CLAUDE.md 隔离特殊区域

```markdown
# src/legacy/CLAUDE.md

⚠️ 警告：这是遗留代码区域

这个目录下的代码不要修改，除非用户明确要求。
如果需要修改这里的功能，建议在 src/ 下创建新的实现，
然后逐步迁移，而不是直接修改这里的代码。

这里的代码：
- 不遵循现代编码规范
- 没有测试覆盖
- 有已知的技术债务
```

### 动态引用其他文件

```markdown
## 参考文档
- API 设计规范：见 docs/api-design.md
- 数据库 schema：见 prisma/schema.prisma
- 环境变量说明：见 .env.example
```

### 写给 AI 的特殊指令

```markdown
## AI 工作指南

### 修改代码前
1. 先读取相关文件，理解现有实现
2. 对于复杂任务，先列出修改计划，等确认后再执行
3. 不要一次修改太多文件

### 测试
- 修改代码后，运行 `pnpm test --run` 确认测试通过
- 如果测试失败，先修复测试再继续

### 提交
- 不要自动提交，等用户确认后再提交
- commit message 遵循 Conventional Commits 规范
```

### 用 `/init` 自动生成初稿

```bash
# 在项目根目录运行
claude
> /init

# Claude 会分析项目结构，自动生成 CLAUDE.md 初稿
# 然后手动补充和调整
```

---

## 与其他工具的对应关系

| 工具 | 配置文件 | 位置 | 是否提交到 git |
|------|----------|------|---------------|
| Claude Code | `CLAUDE.md` | 项目根目录 | ✅ 推荐 |
| Cursor | `.cursor/rules/*.mdc` | 项目根目录 | ✅ 推荐 |
| GitHub Copilot | `.github/copilot-instructions.md` | 项目根目录 | ✅ 推荐 |
| Codex CLI | `AGENTS.md` | 项目根目录 | ✅ 推荐 |
| Aider | `.aider.conf.yml` | 项目根目录 | ✅ 推荐 |
| Windsurf | `.windsurfrules` | 项目根目录 | ✅ 推荐 |

**最佳实践：** 如果团队同时使用多个工具，可以维护一个主配置文件，然后用符号链接或脚本同步到各工具的配置文件。

```bash
# 以 CLAUDE.md 为主，同步到其他工具
cp CLAUDE.md .github/copilot-instructions.md
cp CLAUDE.md AGENTS.md
# 或者用符号链接
ln -s CLAUDE.md AGENTS.md
```

---

*最后更新：2026-03 | 基于 Claude Code CLI*
