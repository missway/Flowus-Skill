---
name: flowus-api
description: "连接、读取、写入 FlowUS 数据库和页面，操作页面 block 内容"
version: 1.0.0
tags: [flowus, api, database, knowledge-base]
homepage: https://github.com/你的用户名/flowus-skill
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [flowus, api, data-access]
---

# FlowUS API 集成工具

通过 FlowUS 官方 REST API 连接、读取、写入 FlowUS 息流工作区的页面、数据库和内容块。

**通用性**：本文档提供 FlowUS API 的完整 curl 参考，不绑定特定 AI Agent——Hermes Agent、Claude Code、Cursor、Codex 等任何能执行 HTTP 请求的 Agent 均可直接使用文档中的命令。

## 技能范畴

本技能只提供 FlowUS API 的对象操作知识（访问、读取、写入、编辑、删除等），包括：

- 连接 FlowUS API
- 读取/搜索页面、数据库、block
- 写入/更新页面、数据库记录、block 内容
- 创建/删除 block、数据库记录
- 操作数据库 schema（字段管理）

> 超出此范畴的请求（如内容优化、文案撰写、对比分析、格式转换等），应拒绝使用本技能处理，转而通过调用其他技能、MCP 工具或调用多模态大模型能力来完成。

## 前置条件

1. **FlowUS 账号**：需要拥有 FlowUS 息流账号
2. **API Token**：在 FlowUS 设置 → 集成中创建机器人集成，获取 Token
3. **将 Token 存入环境变量**：

```bash
echo 'FLOWUS_API_TOKEN=你的_token' >> ~/.hermes/.env
```

## 快速开始

```bash
# 验证 Token
curl -s "https://api.flowus.cn/v1/users/me" \
  -H "Authorization: Bearer $FLOWUS_API_TOKEN"

# 搜索工作区
curl -s "https://api.flowus.cn/v1/search" \
  -H "Authorization: Bearer $FLOWUS_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"query":""}'
```

## 参考文档

- **FlowUS 开发者文档**：https://flowus.cn/share/df7cd54f-1c21-4fc1-9fd8-ce81be1918a5
- **FlowUS 官方 SDK & OpenAPI**：https://github.com/next-space/flowus-api-sdk
- **FlowUS MCP 连接指南**：https://flowus.cn/share/7fc4e81f-dfbc-4e54-9852-3b22dad590a9

## 基础信息

| 项目 | 值 |
|------|-----|
| Base URL | `https://api.flowus.cn` |
| 认证方式 | `Authorization: Bearer <token>` |
| Token 来源 | FlowUS 设置 → 集成 → 创建机器人集成 |
| SDK 语言 | Java / TypeScript / 可生成 Python/Go/C#/PHP |

## 操作对象模型

```
User
  │
Space
  │
  ├── Page ─── Block (children)
  │
  └── Database ─── Page(record) ─── Block
```

## API 端点速查

### 用户
| 操作 | 方法 | 端点 |
|------|------|------|
| 获取当前用户 | `GET` | `/v1/users/me` |

### 搜索
| 操作 | 方法 | 端点 |
|------|------|------|
| 全局搜索 | `POST` | `/v1/search` |
| 搜索页面 | `POST` | `/v1/pages/search` |

请求体：`{"query": "关键词"}`

### 页面
| 操作 | 方法 | 端点 |
|------|------|------|
| 获取页面 | `GET` | `/v1/pages/{page_id}` |
| 创建页面 | `POST` | `/v1/pages` |
| 更新页面 | `PATCH` | `/v1/pages/{page_id}` |

创建时 parent: `{"type": "space_id", "space_id": "..."}` 或 `{"type": "database_id", "database_id": "..."}`

### Block
| 操作 | 方法 | 端点 |
|------|------|------|
| 获取 block | `GET` | `/v1/blocks/{block_id}` |
| 获取子 block | `GET` | `/v1/blocks/{block_id}/children` |
| **追加子 block** | `PATCH` | `/v1/blocks/{block_id}/children` |
| 更新 block | `PATCH` | `/v1/blocks/{block_id}` |
| 删除 block | `DELETE` | `/v1/blocks/{block_id}` |

**追加格式：** `{"children": [{"object": "block", "type": "paragraph", "data": {"rich_text": [{"type": "text", "text": {"content": "内容"}}]}}]}`

**支持的类型：** paragraph, heading_1/2/3, bulleted_list_item, numbered_list_item, to_do, divider, callout, quote, code, image, video, file

### 数据库
| 操作 | 方法 | 端点 |
|------|------|------|
| 获取信息 | `GET` | `/v1/databases/{database_id}` |
| 更新 schema | `PATCH` | `/v1/databases/{database_id}` |
| 创建数据库 | `POST` | `/v1/databases` |
| 查询记录 | `POST` | `/v1/databases/{database_id}/query` |

**字段类型：** title, rich_text, number, select, multi_select, date, people, files, checkbox, url, email, phone_number, relation, formula

### 属性值格式

| 类型 | 格式 |
|------|------|
| title | `{"type":"title","title":[{"type":"text","text":{"content":"标题"}}]}` |
| rich_text | `{"type":"rich_text","rich_text":[{"type":"text","text":{"content":"文本"}}]}` |
| select | `{"type":"select","select":{"name":"选项名"}}` |
| multi_select | `{"type":"multi_select","multi_select":[{"name":"标签1"}]}` |
| number | `{"type":"number","number":42}` |
| url | `{"type":"url","url":"https://..."}` |
| date | `{"type":"date","date":{"start":"2024-01-01"}}` |
| checkbox | `{"type":"checkbox","checkbox":true}` |

## 常见操作流程

**读取页面内容：**
1. `GET /v1/pages/{id}` → 获取元信息
2. `GET /v1/blocks/{id}/children` → 获取根 block
3. 对有子级的 block 递归第 2 步

**写入内容到页面（推荐方式）：**
```bash
PATCH /v1/blocks/{page_id}/children
Body: {"children": [...]}
```

**数据库操作链路：**
1. `GET /v1/databases/{id}` → 看结构
2. `POST /v1/databases/{id}/query` → 查记录
3. `POST /v1/pages` (parent.database_id) → 新增
4. `PATCH /v1/pages/{record_id}` → 更新字段
5. `PATCH /v1/blocks/{record_id}/children` → 写正文

## 注意事项

- Token 仅存 `.env`，不在技能中硬编码
- 默认写入页面内容区（block），不写自定义字段
- `PATCH /v1/pages/{id}` 不支持自定义 rich_text 字段写入数据库记录
- 追加 block 用 `PATCH`, block_id=page_id 时向页面根追加
- `parent.space_id` 可从搜索结果中获得
- 建议配合 `jq` 解析 JSON

## 常见错误排查

| 错误 | 原因 | 解决 |
|------|------|------|
| `401 Unauthorized` | Token 无效 | 检查 `FLOWUS_API_TOKEN`，重新生成 |
| `404 Not Found` | ID 错误 | 确认 UUID 完整，确认有访问权限 |
| `500 Server Error` | 服务端问题 | 稍后重试 |
| `jwt malformed` | 非 JWT 格式 | 确认是 Bearer Token |
| `No authorization token` | 未传认证头 | 加 `-H "Authorization: Bearer $TOKEN"` |
| 请求返回空 | 网络/超时 | 加 `-w "\nHTTP %{http_code}"` 看状态码 |

## Install

```bash
hermes skills install https://raw.githubusercontent.com/你的用户名/flowus-skill/main/SKILL.md --name flowus-api
```
