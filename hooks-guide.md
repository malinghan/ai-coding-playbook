# Claude Code Hooks 自动化指南

> Hooks 让你在 Claude Code 的特定事件发生时自动执行 shell 命令，实现格式化、测试、通知等工作流自动化。

---

## 目录

1. [什么是 Hooks](#什么是-hooks)
2. [配置方式](#配置方式)
3. [Hook 事件类型](#hook-事件类型)
4. [环境变量](#环境变量)
5. [实用 Hook 示例](#实用-hook-示例)
6. [高级用法](#高级用法)
7. [调试技巧](#调试技巧)

---

## 什么是 Hooks

Hooks 是在 Claude Code 工具调用前后自动执行的 shell 命令。

```
用户请求 Claude 修改文件
    ↓
Claude 调用 Write 工具
    ↓
[PostToolUse Hook 触发]
    ↓
自动运行 prettier 格式化
    ↓
文件已格式化，符合代码规范
```

**核心用途：**
- 自动格式化：每次 Claude 写文件后自动跑 prettier/eslint
- 安全审计：记录所有 bash 命令执行日志
- 质量保证：文件修改后自动跑相关测试
- 通知：任务完成后发送桌面通知

---

## 配置方式

### 配置文件位置

```
~/.claude/settings.json          ← 全局（所有项目）
.claude/settings.json            ← 项目级（当前项目）
```

项目级配置优先级更高，且可以提交到 git 供团队共享。

### 基本结构

```json
{
  "hooks": {
    "事件名称": [
      {
        "matcher": "工具名称匹配模式",
        "hooks": [
          {
            "type": "command",
            "command": "要执行的 shell 命令"
          }
        ]
      }
    ]
  }
}
```

---

## Hook 事件类型

| 事件 | 触发时机 |
|------|----------|
| `PreToolUse` | 工具调用之前 |
| `PostToolUse` | 工具调用之后 |
| `Notification` | Claude 发送通知时 |
| `Stop` | Claude 完成响应时 |
| `SubagentStop` | 子 Agent 完成时 |

### matcher 匹配规则

```json
// 匹配单个工具
"matcher": "Write"

// 匹配多个工具（用 | 分隔）
"matcher": "Write|Edit"

// 匹配所有工具（空字符串）
"matcher": ""

// 匹配 Bash 工具
"matcher": "Bash"
```

---

## 环境变量

Hook 命令中可以使用以下环境变量：

| 变量 | 说明 | 适用事件 |
|------|------|----------|
| `$CLAUDE_FILE_PATH` | 被操作的文件路径 | Write、Edit、Read |
| `$CLAUDE_TOOL_NAME` | 当前工具名称 | 所有 |
| `$CLAUDE_TOOL_INPUT` | 工具输入（JSON） | 所有 |
| `$CLAUDE_TOOL_OUTPUT` | 工具输出（JSON） | PostToolUse |
| `$CLAUDE_SESSION_ID` | 当前会话 ID | 所有 |

---

## 实用 Hook 示例

### 1. 自动格式化（最常用）

每次 Claude 写文件后自动格式化：

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
    ]
  }
}
```

**针对不同文件类型：**

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "case \"$CLAUDE_FILE_PATH\" in *.ts|*.tsx|*.js|*.jsx|*.json|*.css) prettier --write \"$CLAUDE_FILE_PATH\" 2>/dev/null || true ;; *.py) black \"$CLAUDE_FILE_PATH\" 2>/dev/null || true ;; *.go) gofmt -w \"$CLAUDE_FILE_PATH\" 2>/dev/null || true ;; esac"
          }
        ]
      }
    ]
  }
}
```

### 2. ESLint 自动修复

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "if [[ \"$CLAUDE_FILE_PATH\" == *.ts || \"$CLAUDE_FILE_PATH\" == *.tsx ]]; then eslint --fix \"$CLAUDE_FILE_PATH\" 2>/dev/null || true; fi"
          }
        ]
      }
    ]
  }
}
```

### 3. 命令执行日志

记录所有 Claude 执行的 bash 命令，便于审计：

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "echo \"[$(date '+%Y-%m-%d %H:%M:%S')] CMD: $CLAUDE_TOOL_INPUT\" >> ~/.claude/command-log.txt"
          }
        ]
      }
    ]
  }
}
```

### 4. 自动运行测试

文件修改后自动运行相关测试：

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "if [[ \"$CLAUDE_FILE_PATH\" == src/* ]]; then pnpm test --run --reporter=verbose 2>&1 | tail -20; fi"
          }
        ]
      }
    ]
  }
}
```

### 5. 任务完成通知

Claude 完成响应后发送桌面通知（macOS）：

```json
{
  "hooks": {
    "Stop": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "osascript -e 'display notification \"Claude 已完成任务\" with title \"Claude Code\"' 2>/dev/null || true"
          }
        ]
      }
    ]
  }
}
```

**Linux（使用 notify-send）：**

```json
{
  "hooks": {
    "Stop": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "notify-send 'Claude Code' 'Claude 已完成任务' 2>/dev/null || true"
          }
        ]
      }
    ]
  }
}
```

### 6. 防止修改特定文件

在 PreToolUse 中阻止 Claude 修改某些文件：

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "if [[ \"$CLAUDE_FILE_PATH\" == *prisma/migrations/* ]]; then echo '禁止修改迁移文件！请使用 pnpm db:migrate 生成' >&2; exit 1; fi"
          }
        ]
      }
    ]
  }
}
```

> 注意：Hook 命令返回非零退出码会阻止工具调用执行。

### 7. 自动备份

修改重要文件前自动备份：

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "if [ -f \"$CLAUDE_FILE_PATH\" ]; then cp \"$CLAUDE_FILE_PATH\" \"$CLAUDE_FILE_PATH.bak\" 2>/dev/null || true; fi"
          }
        ]
      }
    ]
  }
}
```

---

## 高级用法

### 组合多个 Hook

同一个事件可以配置多个 Hook，按顺序执行：

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
          },
          {
            "type": "command",
            "command": "eslint --fix \"$CLAUDE_FILE_PATH\" 2>/dev/null || true"
          },
          {
            "type": "command",
            "command": "echo \"[$(date)] Modified: $CLAUDE_FILE_PATH\" >> .claude/changes.log"
          }
        ]
      }
    ]
  }
}
```

### 用脚本处理复杂逻辑

对于复杂的 Hook 逻辑，把命令放到脚本文件中：

`.claude/hooks/post-write.sh`:
```bash
#!/bin/bash
FILE="$CLAUDE_FILE_PATH"

# 格式化
case "$FILE" in
  *.ts|*.tsx|*.js|*.jsx)
    prettier --write "$FILE" 2>/dev/null
    eslint --fix "$FILE" 2>/dev/null
    ;;
  *.py)
    black "$FILE" 2>/dev/null
    isort "$FILE" 2>/dev/null
    ;;
  *.go)
    gofmt -w "$FILE" 2>/dev/null
    ;;
esac

# 记录日志
echo "[$(date '+%Y-%m-%d %H:%M:%S')] $FILE" >> .claude/modified-files.log
```

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "bash .claude/hooks/post-write.sh"
          }
        ]
      }
    ]
  }
}
```

### 完整的团队配置示例

`.claude/settings.json`（提交到 git）：

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "bash .claude/hooks/format.sh 2>/dev/null || true"
          }
        ]
      }
    ],
    "PreToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "bash .claude/hooks/guard.sh 2>/dev/null || true"
          }
        ]
      },
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "echo \"[$(date '+%Y-%m-%d %H:%M:%S')] $CLAUDE_TOOL_INPUT\" >> .claude/bash-log.txt 2>/dev/null || true"
          }
        ]
      }
    ],
    "Stop": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "osascript -e 'display notification \"任务完成\" with title \"Claude Code\"' 2>/dev/null || true"
          }
        ]
      }
    ]
  }
}
```

---

## 调试技巧

### 查看 Hook 是否触发

```bash
# 在 Hook 命令中加入日志
"command": "echo \"Hook triggered: $CLAUDE_TOOL_NAME on $CLAUDE_FILE_PATH\" >> /tmp/claude-hooks.log && prettier --write \"$CLAUDE_FILE_PATH\""

# 实时查看日志
tail -f /tmp/claude-hooks.log
```

### 测试 Hook 命令

在终端中手动模拟 Hook 执行：

```bash
# 模拟环境变量
export CLAUDE_FILE_PATH="src/utils/date.ts"
export CLAUDE_TOOL_NAME="Write"

# 运行 Hook 命令
prettier --write "$CLAUDE_FILE_PATH"
```

### 常见问题

**Hook 没有触发？**
- 检查 `matcher` 是否正确（区分大小写）
- 检查配置文件 JSON 格式是否正确
- 用 `/doctor` 命令检查 Claude Code 配置

**Hook 命令报错但不想阻止工具执行？**
- 在命令末尾加 `|| true`，确保退出码为 0
- 把错误输出重定向到 `/dev/null`：`2>/dev/null`

**Hook 执行太慢影响体验？**
- 把耗时操作放到后台：`command &`
- 只在必要时运行（用条件判断）

---

## 快速参考

```json
// 最常用的 Hook 配置
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Write|Edit",
      "hooks": [{
        "type": "command",
        "command": "prettier --write \"$CLAUDE_FILE_PATH\" 2>/dev/null || true"
      }]
    }],
    "Stop": [{
      "matcher": "",
      "hooks": [{
        "type": "command",
        "command": "osascript -e 'display notification \"完成\" with title \"Claude Code\"' 2>/dev/null || true"
      }]
    }]
  }
}
```

---

*最后更新：2026-03 | 基于 Claude Code CLI*
