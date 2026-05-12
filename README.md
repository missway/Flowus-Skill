# FlowUS API Skill

> FlowUS (息流) API integration — read/write pages, blocks, databases via REST API
>
> 通过 FlowUS 官方 REST API 连接、读取、写入 FlowUS 工作区的页面、数据库和内容块

---

**🇬🇧 English** · [中文 🇨🇳](#中文版)

This is a **universal API reference document**. It works with any AI agent that can execute HTTP requests — Hermes Agent, Claude Code, Cursor, Codex, Windsurf, or plain curl in your terminal.

---

## Features

- 🔍 Search pages and databases in your workspace
- 📄 Read, create, update pages
- ✏️ Append, update, delete content blocks (paragraph, heading, list, code, etc.)
- 🗄️ Manage database schema (fields, options) and records
- 🔑 Support for 15+ property field types

## Prerequisites

1. A [FlowUS](https://flowus.cn) account
2. An API token — go to **Settings → Integration → Create Bot Integration**, copy the token
3. Set the token as an environment variable:

```bash
export FLOWUS_API_TOKEN=your_token_here
```

### Verify Token

```bash
curl -s "https://api.flowus.cn/v1/users/me" \
  -H "Authorization: Bearer $FLOWUS_API_TOKEN"
```

You should see your user info in the response.

## Quick Examples

```bash
# Search all pages/databases
curl -s "https://api.flowus.cn/v1/search" \
  -H "Authorization: Bearer $FLOWUS_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"query":""}'

# Get page content
curl -s "https://api.flowus.cn/v1/blocks/{page_id}/children" \
  -H "Authorization: Bearer $FLOWUS_API_TOKEN"

# Write content to a page
curl -s -X PATCH "https://api.flowus.cn/v1/blocks/{page_id}/children" \
  -H "Authorization: Bearer $FLOWUS_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"children":[{"object":"block","type":"paragraph","data":{"rich_text":[{"type":"text","text":{"content":"Hello FlowUS!"}}]}}]}'
```

## For Hermes Agent Users

To install as a Hermes Agent skill:

```bash
mkdir -p ~/.hermes/skills/productivity/flowus-api
cp SKILL.md ~/.hermes/skills/productivity/flowus-api/
echo 'FLOWUS_API_TOKEN=your_token' >> ~/.hermes/.env
```

## For Other Agents

- **Claude Code**: Provide `SKILL.md` as context, or use the curl commands directly
- **Codex / Cursor / Windsurf**: Pair with the [flowus-mcp-server](https://github.com/missway/flowus-mcp-server) for MCP-based integration
- **Any terminal**: All curl commands work in any shell

## API Reference

Full API specification is in [SKILL.md](SKILL.md), covering:

- User — `GET /v1/users/me`
- Search — `POST /v1/search`
- Pages — `GET/PATCH/POST /v1/pages`
- Blocks — `GET/PATCH/DELETE /v1/blocks`
- Databases — `GET/PATCH/POST /v1/databases`

## Sources

- [FlowUS Developer Docs](https://flowus.cn/share/df7cd54f-1c21-4fc1-9fd8-ce81be1918a5)
- [FlowUS Official SDK (GitHub)](https://github.com/next-space/flowus-api-sdk)

---

<h1 id="中文版">🇨🇳 中文版</h1>

## 功能

- 🔍 搜索工作区中的页面和数据库
- 📄 读取、创建、更新页面
- ✏️ 追加、更新、删除内容块（paragraph、heading、list、code 等）
- 🗄️ 管理数据库 schema（字段、选项）和记录
- 🔑 支持 15+ 种属性字段类型

## 前置准备

1. 拥有 [FlowUS](https://flowus.cn) 账号
2. 获取 API Token：**设置 → 集成 → 创建机器人集成** → 复制 Token
3. 将 Token 存入环境变量：

```bash
export FLOWUS_API_TOKEN=你的_token
```

### 验证 Token

```bash
curl -s "https://api.flowus.cn/v1/users/me" \
  -H "Authorization: Bearer $FLOWUS_API_TOKEN"
```

返回用户信息即表示 Token 有效。

## 快速示例

```bash
# 搜索所有页面/数据库
curl -s "https://api.flowus.cn/v1/search" \
  -H "Authorization: Bearer $FLOWUS_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"query":""}'

# 获取页面内容
curl -s "https://api.flowus.cn/v1/blocks/{page_id}/children" \
  -H "Authorization: Bearer $FLOWUS_API_TOKEN"

# 向页面写入内容
curl -s -X PATCH "https://api.flowus.cn/v1/blocks/{page_id}/children" \
  -H "Authorization: Bearer $FLOWUS_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"children":[{"object":"block","type":"paragraph","data":{"rich_text":[{"type":"text","text":{"content":"你好 FlowUS!"}}]}}]}'
```

## 在 Hermes Agent 中使用

```bash
mkdir -p ~/.hermes/skills/productivity/flowus-api
cp SKILL.md ~/.hermes/skills/productivity/flowus-api/
echo 'FLOWUS_API_TOKEN=你的_token' >> ~/.hermes/.env
```

## 在其他 Agent 中使用

- **Claude Code**：将 SKILL.md 作为文档上下文提供
- **Codex / Cursor / Windsurf**：配合 [flowus-mcp-server](https://github.com/missway/flowus-mcp-server) 使用 MCP 协议
- **通用**：所有 curl 命令在任何 shell 环境中均可执行

## API 参考

完整 API 规范见 [SKILL.md](SKILL.md)，涵盖：

- 用户操作 — `GET /v1/users/me`
- 搜索 — `POST /v1/search`
- 页面 CRUD — `GET/PATCH/POST /v1/pages`
- Block 操作 — `GET/PATCH/DELETE /v1/blocks`
- 数据库操作 — `GET/PATCH/POST /v1/databases`

## 数据来源

- [FlowUS 开发者文档](https://flowus.cn/share/df7cd54f-1c21-4fc1-9fd8-ce81be1918a5)
- [FlowUS 官方 SDK (GitHub)](https://github.com/next-space/flowus-api-sdk)

---

## License / 许可证

MIT
