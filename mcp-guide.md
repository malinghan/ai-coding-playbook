# MCP（Model Context Protocol）使用指南

> MCP 是 Anthropic 开放的协议，让 AI 工具连接外部数据源和服务。配置一次，Claude Code、Cursor、Windsurf 等工具都能使用。

---

## 目录

1. [什么是 MCP](#什么是-mcp)
2. [配置方式](#配置方式)
3. [常用 MCP 服务器](#常用-mcp-服务器)
4. [按工具配置](#按工具配置)
5. [自定义 MCP 服务器](#自定义-mcp-服务器)
6. [实用场景示例](#实用场景示例)
7. [安全注意事项](#安全注意事项)

---

## 什么是 MCP

MCP（Model Context Protocol）是一个开放标准，让 AI 模型能够安全地连接外部工具和数据源。

```
AI 工具（Claude Code / Cursor / Windsurf）
    ↕ MCP 协议
MCP 服务器（文件系统 / 数据库 / GitHub / Slack / ...）
    ↕
外部数据和服务
```

**核心价值：**
- AI 可以直接查询数据库，不需要你复制粘贴 SQL 结果
- AI 可以操作 GitHub Issues/PR，不需要你手动执行 gh 命令
- AI 可以搜索网络，获取最新信息
- 一次配置，多个工具复用

---

## 配置方式

### Claude Code

编辑 `~/.claude/settings.json`（全局）或项目 `.claude/settings.json`：

```json
{
  "mcpServers": {
    "服务器名称": {
      "command": "启动命令",
      "args": ["参数1", "参数2"],
      "env": {
        "ENV_VAR": "value"
      }
    }
  }
}
```

### Cursor

编辑 `~/.cursor/mcp.json` 或项目 `.cursor/mcp.json`：

```json
{
  "mcpServers": {
    "服务器名称": {
      "command": "启动命令",
      "args": ["参数1", "参数2"]
    }
  }
}
```

### Windsurf

编辑 `~/.codeium/windsurf/mcp_config.json`：

```json
{
  "mcpServers": {
    "服务器名称": {
      "command": "启动命令",
      "args": ["参数1", "参数2"]
    }
  }
}
```

---

## 常用 MCP 服务器

### 文件系统

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "/Users/yourname/projects"
      ]
    }
  }
}
```

**用途：** 让 AI 读写指定目录的文件，适合跨项目操作。

### GitHub

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "ghp_xxxx"
      }
    }
  }
}
```

**用途：** 操作 GitHub Issues、PR、代码搜索、仓库管理。

```
# 使用示例
"在 GitHub 上创建 issue：修复用户登录超时，标签加 bug 和 priority:high"
"搜索 GitHub 上关于 React Server Components 的最新讨论"
"查看 PR #234 的 review 意见，帮我回复"
```

### PostgreSQL

```json
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-postgres",
        "postgresql://localhost/mydb"
      ]
    }
  }
}
```

**用途：** 直接查询数据库，分析数据，生成报告。

```
# 使用示例
"查询过去 7 天每天的新注册用户数"
"找出所有没有关联订单的用户"
"分析 orders 表的索引使用情况，建议优化方案"
```

### SQLite

```json
{
  "mcpServers": {
    "sqlite": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-sqlite",
        "/path/to/database.db"
      ]
    }
  }
}
```

### Brave Search（网络搜索）

```json
{
  "mcpServers": {
    "brave-search": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-brave-search"],
      "env": {
        "BRAVE_API_KEY": "your_brave_api_key"
      }
    }
  }
}
```

**用途：** 让 AI 搜索最新信息，获取文档、新闻、技术资料。

```
# 使用示例
"搜索 Next.js 15 的 breaking changes"
"查找 Prisma 5 的迁移指南"
```

### Puppeteer（浏览器控制）

```json
{
  "mcpServers": {
    "puppeteer": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-puppeteer"]
    }
  }
}
```

**用途：** 控制浏览器，截图、爬取网页、自动化测试。

```
# 使用示例
"截图 https://example.com，检查页面布局是否正常"
"爬取竞品网站的定价页面，整理成表格"
```

### Slack

```json
{
  "mcpServers": {
    "slack": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-slack"],
      "env": {
        "SLACK_BOT_TOKEN": "xoxb-xxxx",
        "SLACK_TEAM_ID": "T0XXXX"
      }
    }
  }
}
```

**用途：** 读取 Slack 消息，发送通知，搜索历史记录。

### Memory（持久化记忆）

```json
{
  "mcpServers": {
    "memory": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-memory"]
    }
  }
}
```

**用途：** 跨会话持久化知识图谱，让 AI 记住项目相关信息。

---

## 按工具配置

### 完整的 Claude Code 配置示例

`~/.claude/settings.json`：

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
    },
    "brave-search": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-brave-search"],
      "env": {
        "BRAVE_API_KEY": "BSA_xxxx"
      }
    }
  }
}
```

### 查看已连接的 MCP

```bash
# Claude Code
/mcp

# 查看可用工具
# Claude 会列出所有 MCP 服务器提供的工具
```

---

## 自定义 MCP 服务器

如果现有服务器不满足需求，可以自己写一个。

### 最简单的 MCP 服务器（Node.js）

```typescript
// my-mcp-server.ts
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import {
  CallToolRequestSchema,
  ListToolsRequestSchema,
} from "@modelcontextprotocol/sdk/types.js";

const server = new Server(
  { name: "my-server", version: "1.0.0" },
  { capabilities: { tools: {} } }
);

// 定义工具列表
server.setRequestHandler(ListToolsRequestSchema, async () => ({
  tools: [
    {
      name: "get_deployment_status",
      description: "获取指定环境的部署状态",
      inputSchema: {
        type: "object",
        properties: {
          environment: {
            type: "string",
            enum: ["staging", "production"],
            description: "目标环境",
          },
        },
        required: ["environment"],
      },
    },
  ],
}));

// 实现工具逻辑
server.setRequestHandler(CallToolRequestSchema, async (request) => {
  if (request.params.name === "get_deployment_status") {
    const { environment } = request.params.arguments as { environment: string };
    // 调用你的内部 API
    const status = await fetchDeploymentStatus(environment);
    return {
      content: [{ type: "text", text: JSON.stringify(status, null, 2) }],
    };
  }
  throw new Error(`Unknown tool: ${request.params.name}`);
});

async function fetchDeploymentStatus(env: string) {
  // 实现你的逻辑
  return { environment: env, status: "healthy", version: "1.2.3" };
}

const transport = new StdioServerTransport();
await server.connect(transport);
```

### 注册自定义服务器

```json
{
  "mcpServers": {
    "my-server": {
      "command": "npx",
      "args": ["tsx", "/path/to/my-mcp-server.ts"]
    }
  }
}
```

### 常见自定义场景

- **内部 API 集成**：让 AI 查询公司内部系统
- **CI/CD 状态**：让 AI 查看构建和部署状态
- **监控告警**：让 AI 查询 Datadog/Grafana 指标
- **文档系统**：让 AI 搜索 Confluence/Notion 文档
- **任务管理**：让 AI 操作 Jira/Linear tickets

---

## 实用场景示例

### 场景 1：数据库驱动的开发

配置 PostgreSQL MCP 后：

```
"查看 users 表的结构，然后帮我写一个查询，
找出过去 30 天注册但从未下单的用户"

→ AI 自动查询表结构，生成并执行 SQL，返回结果
```

### 场景 2：GitHub 工作流自动化

配置 GitHub MCP 后：

```
"查看 PR #156 的所有 review 意见，
帮我逐条回复，说明我已经修复了哪些，
哪些是设计决策不打算改"

→ AI 读取 PR comments，生成回复草稿
```

### 场景 3：基于最新文档开发

配置 Brave Search MCP 后：

```
"搜索 Prisma 5 的 relation 查询语法，
然后把 src/repositories/user.ts 中的查询更新到新语法"

→ AI 搜索最新文档，然后修改代码
```

### 场景 4：跨项目操作

配置 Filesystem MCP 后：

```
"在 ~/projects/shared-utils 中找到 formatDate 函数，
把它复制到当前项目的 src/utils/date.ts"

→ AI 读取其他项目的文件，复制到当前项目
```

---

## 安全注意事项

### 最小权限原则

只给 MCP 服务器必要的权限：

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "/Users/yourname/projects"  // 只允许访问 projects 目录，不是整个 home
      ]
    }
  }
}
```

### 数据库只读访问

对于生产数据库，使用只读账号：

```json
{
  "mcpServers": {
    "prod-db": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-postgres",
        "postgresql://readonly_user:password@prod-db/mydb"  // 只读账号
      ]
    }
  }
}
```

### 敏感信息用环境变量

不要把 token 直接写在配置文件里：

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"  // 引用环境变量
      }
    }
  }
}
```

```bash
# ~/.zshrc
export GITHUB_TOKEN="ghp_xxxx"
```

### 不要连接生产数据库的写权限

```
# 危险：AI 可能意外修改生产数据
postgres://admin@prod-db/mydb

# 安全：只读账号
postgres://readonly@prod-db/mydb

# 更安全：只连接开发/测试数据库
postgres://admin@localhost/mydb_dev
```

---

## MCP 服务器速查表

| 服务器 | npm 包 | 主要用途 | 需要的凭证 |
|--------|--------|----------|-----------|
| filesystem | `@modelcontextprotocol/server-filesystem` | 读写本地文件 | 无 |
| github | `@modelcontextprotocol/server-github` | GitHub 操作 | Personal Access Token |
| postgres | `@modelcontextprotocol/server-postgres` | PostgreSQL 查询 | 数据库连接字符串 |
| sqlite | `@modelcontextprotocol/server-sqlite` | SQLite 查询 | 数据库文件路径 |
| brave-search | `@modelcontextprotocol/server-brave-search` | 网络搜索 | Brave API Key |
| puppeteer | `@modelcontextprotocol/server-puppeteer` | 浏览器控制 | 无 |
| slack | `@modelcontextprotocol/server-slack` | Slack 操作 | Bot Token |
| memory | `@modelcontextprotocol/server-memory` | 持久化记忆 | 无 |

更多服务器：[MCP 服务器目录](https://github.com/modelcontextprotocol/servers)

---

*最后更新：2026-03 | 基于 MCP 协议规范*
