---
name: flowus-api
description: "连接、读取、写入 FlowUS 数据库和页面，操作页面 block 内容"
version: 1.0.0
author: Hermes Agent community
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
# 在 ~/.hermes/.env 中添加
FLOWUS_API_TOKEN=你的_token
```

## 快速开始

```bash
# 验证 Token 是否有效
curl -s "https://api.flowus.cn/v1/users/me" \
  -H "Authorization: Bearer $FLOWUS_API_TOKEN"

# 搜索工作区中的页面
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
| SDK 语言 | Java (Gson) / TypeScript (Fetch API) / 可生成 Python/Go/C#/PHP |

## 操作对象模型

```
User ─── 机器人/用户身份
  │
Space ─── 工作空间（顶级容器）
  │
  ├── Page ─── 页面（可以包含子内容）
  │     └── Block ─── 内容块（paragraph/heading/list/divider 等）
  │           └── Block (children) ─── 嵌套子块
  │
  └── Database ─── 数据库（多维表）
        └── Page (database record) ─── 数据库中的一行记录
              └── Block ─── 记录页面的内容块
```

## 完整端点粒度

### 1. 用户操作 (User)

| 操作 | 方法 | 端点 | curl 示例 |
|------|------|------|-----------|
| 获取当前用户 | `GET` | `/v1/users/me` | `curl -s "https://api.flowus.cn/v1/users/me" -H "Authorization: Bearer $TOKEN"` |

**响应示例：**
```json
{"object":"user","id":"...","type":"person","person":{"email":"..."},"name":"...","avatar_url":null}
```

---

### 2. 搜索操作 (Search)

| 操作 | 方法 | 端点 | 说明 |
|------|------|------|------|
| 全局搜索 | `POST` | `/v1/search` | 搜索页面和数据库标题 |
| 搜索页面 | `POST` | `/v1/pages/search` | 限定搜索页面 |

**请求体：**
```json
{"query": "关键词"}
```

**curl 示例：**
```bash
curl -s "https://api.flowus.cn/v1/search" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"query":"关键词"}'
```

---

### 3. 页面操作 (Page)

| 操作 | 方法 | 端点 | 说明 |
|------|------|------|------|
| **获取页面** | `GET` | `/v1/pages/{page_id}` | 获取页面属性元信息 |
| **创建页面** | `POST` | `/v1/pages` | 创建独立页面或数据库记录 |
| **更新页面** | `PATCH` | `/v1/pages/{page_id}` | 更新页面属性字段 |

**创建独立页面：**
```json
{
  "parent": {"type": "space_id", "space_id": "你的空间ID"},
  "properties": {
    "title": {"type": "title", "title": [{"type": "text", "text": {"content": "页面标题"}}]}
  }
}
```

**创建数据库记录（parent 不同）：**
```json
{
  "parent": {"type": "database_id", "database_id": "数据库ID"},
  "properties": {
    "title": {"type": "title", "title": [{"type": "text", "text": {"content": "记录标题"}}]}
  }
}
```

> 空间 ID 可通过 `GET /v1/search` 返回结果的 `parent.space_id` 获得。

**注意：** `PATCH /v1/pages/{id}` 支持更新标准字段（title/select/multi_select/url 等），但不支持写入自定义 rich_text 字段到数据库记录。

---

### 4. Block 操作 (Block)

Block 是页面内的内容单元，也是操作最频繁的层级。

| 操作 | 方法 | 端点 | 说明 |
|------|------|------|------|
| **获取 block** | `GET` | `/v1/blocks/{block_id}` | 获取单个 block 信息 |
| **获取子 block 列表** | `GET` | `/v1/blocks/{block_id}/children` | 获取 block 的直接子级 |
| **追加子 block** | `PATCH` | `/v1/blocks/{block_id}/children` | 向 block 末尾追加子 block |
| **更新 block** | `PATCH` | `/v1/blocks/{block_id}` | 更新 block 内容/类型 |
| **删除 block** | `DELETE` | `/v1/blocks/{block_id}` | 删除 block |

**获取页面内容（根 block = 页面本身）：**
```bash
curl -s "https://api.flowus.cn/v1/blocks/{page_id}/children" \
  -H "Authorization: Bearer $TOKEN"
```

**追加子 block（写入内容到页面）：**
```bash
curl -s -X PATCH "https://api.flowus.cn/v1/blocks/{page_id}/children" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "children": [
      {"object": "block", "type": "paragraph", "data": {"rich_text": [{"type": "text", "text": {"content": "内容"}}]}},
      {"object": "block", "type": "heading_2", "data": {"rich_text": [{"type": "text", "text": {"content": "小标题"}}]}}
    ]
  }'
```

**支持追加的 block 类型：**

| 类型 | data 结构 | 说明 |
|------|-----------|------|
| `paragraph` | `{"rich_text": [...]}` | 段落 |
| `heading_1` | `{"rich_text": [...]}` | 一级标题 |
| `heading_2` | `{"rich_text": [...]}` | 二级标题 |
| `heading_3` | `{"rich_text": [...]}` | 三级标题 |
| `bulleted_list_item` | `{"rich_text": [...]}` | 无序列表项 |
| `numbered_list_item` | `{"rich_text": [...]}` | 有序列表项 |
| `to_do` | `{"rich_text": [...], "checked": false}` | 待办事项 |
| `divider` | `{}` | 分割线 |
| `callout` | `{"rich_text": [...]}` | 标注/提示框 |
| `quote` | `{"rich_text": [...]}` | 引用 |
| `code` | `{"rich_text": [...], "language": "python"}` | 代码块 |
| `image` | `{"url": "...", "caption": "..."}` | 图片 |
| `video` | `{"url": "...", "caption": "..."}` | 视频 |
| `file` | `{"url": "...", "caption": "..."}` | 文件 |

**rich_text 结构：**
```json
{
  "rich_text": [
    {"type": "text", "text": {"content": "普通文本", "link": null}},
    {"type": "text", "text": {"content": "带链接", "link": {"url": "https://..."}}}
  ]
}
```

**更新 block 内容：**
```bash
curl -s -X PATCH "https://api.flowus.cn/v1/blocks/{block_id}" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"data": {"rich_text": [{"type": "text", "text": {"content": "新内容"}}]}}'
```

**删除 block：**
```bash
curl -s -X DELETE "https://api.flowus.cn/v1/blocks/{block_id}" \
  -H "Authorization: Bearer $TOKEN"
```

---

### 5. 数据库操作 (Database)

| 操作 | 方法 | 端点 | 说明 |
|------|------|------|------|
| **获取数据库信息** | `GET` | `/v1/databases/{database_id}` | 获取 schema、字段定义、选项列表 |
| **更新数据库** | `PATCH` | `/v1/databases/{database_id}` | 修改 schema（增删字段、选项） |
| **创建数据库** | `POST` | `/v1/databases` | 创建新数据库 |
| **查询记录** | `POST` | `/v1/databases/{database_id}/query` | 查询数据库中的记录 |

**获取数据库 schema（含所有字段定义和选项）：**
```bash
curl -s "https://api.flowus.cn/v1/databases/{database_id}" \
  -H "Authorization: Bearer $TOKEN"
```

**查询数据库所有记录：**
```bash
curl -s -X POST "https://api.flowus.cn/v1/databases/{database_id}/query" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{}'
```

**新增字段：**
```bash
curl -s -X PATCH "https://api.flowus.cn/v1/databases/{database_id}" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "properties": {
      "新字段名": {"type": "rich_text"}
    }
  }'
```

**支持的字段类型：** `title`, `rich_text`, `number`, `select`, `multi_select`, `date`, `people`, `files`, `checkbox`, `url`, `email`, `phone_number`, `relation`, `formula`

**select 字段添加选项（同时创建字段）：**
```json
{
  "properties": {
    "状态": {
      "type": "select",
      "select": {
        "options": [
          {"name": "待处理", "color": "red"},
          {"name": "进行中", "color": "yellow"},
          {"name": "已完成", "color": "green"}
        ]
      }
    }
  }
}
```

**可用的 color 值：** `default`, `gray`, `brown`, `orange`, `yellow`, `green`, `blue`, `purple`, `pink`, `red`

---

### 6. 属性字段值格式 (Property Values)

创建/更新数据库记录时各字段类型的值格式：

| 类型 | 请求体格式 |
|------|-----------|
| `title` | `{"type":"title","title":[{"type":"text","text":{"content":"标题"}}]}` |
| `rich_text` | `{"type":"rich_text","rich_text":[{"type":"text","text":{"content":"文本"}}]}` |
| `select` | `{"type":"select","select":{"name":"选项名"}}` |
| `multi_select` | `{"type":"multi_select","multi_select":[{"name":"标签1"},{"name":"标签2"}]}` |
| `number` | `{"type":"number","number":42}` |
| `url` | `{"type":"url","url":"https://..."}` |
| `date` | `{"type":"date","date":{"start":"2024-01-01","end":null}}` |
| `checkbox` | `{"type":"checkbox","checkbox":true}` |
| `email` | `{"type":"email","email":"user@example.com"}` |
| `phone_number` | `{"type":"phone_number","phone_number":"+86..."}` |

---

## 常见操作流程

### 读取页面完整内容
```bash
# 1. 获取页面元信息
curl -s "https://api.flowus.cn/v1/pages/{page_id}" -H "Authorization: Bearer $TOKEN"

# 2. 获取页面根 block 列表
curl -s "https://api.flowus.cn/v1/blocks/{page_id}/children" -H "Authorization: Bearer $TOKEN"

# 3. 对于 has_children=true 的 block，递归获取子 block
curl -s "https://api.flowus.cn/v1/blocks/{block_id}/children" -H "Authorization: Bearer $TOKEN"
```

### 向页面写入内容
```bash
# 向页面末尾追加内容
curl -s -X PATCH "https://api.flowus.cn/v1/blocks/{page_id}/children" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"children": [{"object":"block","type":"paragraph","data":{"rich_text":[{"type":"text","text":{"content":"写入的内容"}}]}}]}'
```

### 搜索并操作
```bash
# 1. 搜索找到目标
curl -s "https://api.flowus.cn/v1/search" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"query":"关键词"}' | jq '.results[] | {id, title: .properties.title.title[0].text.content}'

# 2. 按 ID 读取或修改
```

### 数据库操作链路
```bash
# 1. 查看数据库结构
curl -s "https://api.flowus.cn/v1/databases/{id}" -H "Authorization: Bearer $TOKEN"

# 2. 查询所有记录
curl -s -X POST "https://api.flowus.cn/v1/databases/{id}/query" \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" -d '{}'

# 3. 新增记录
curl -s -X POST "https://api.flowus.cn/v1/pages" \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{"parent":{"type":"database_id","database_id":"{id}"},"properties":{...}}'

# 4. 更新记录字段
curl -s -X PATCH "https://api.flowus.cn/v1/pages/{record_id}" \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{"properties":{...}}'

# 5. 向记录正文写入内容
curl -s -X PATCH "https://api.flowus.cn/v1/blocks/{record_id}/children" \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{"children":[...]}'
```

## 注意事项

- Token 存储在环境变量 `FLOWUS_API_TOKEN` 中，不要硬编码在技能文件里
- 所有写入默认写入页面内容区（block），不写入数据库自定义字段
- `PATCH /v1/pages/{id}` 不支持写入自定义 rich_text 字段到数据库记录
- 追加 block 使用 `PATCH /v1/blocks/{block_id}/children`，block_id = page_id 时向页面根追加
- 删除 block 使用 `DELETE /v1/blocks/{block_id}`
- API 返回的 `parent.space_id` 可用于获取空间 ID
- 建议配合 `jq` 解析 JSON 响应

## 常见错误排查

| 错误 | 原因 | 解决方法 |
|------|------|---------|
| `401 Unauthorized` | Token 无效或过期 | 检查 `FLOWUS_API_TOKEN` 是否正确，重新在 FlowUS 设置中生成 Token |
| `404 Not Found` | 页面/数据库/block ID 错误 | 确认 ID 是否完整（完整 UUID 格式），确认 Token 有对应资源的访问权限 |
| `500 Server Error` | FlowUS 服务端问题 | 稍后重试，检查 FlowUS 服务状态 |
| `jwt malformed` | Token 格式不是有效 JWT | 确保使用的是 Bearer Token 而非其他类型凭据 |
| `No authorization token was found` | 未传递认证信息 | 确认请求头中包含 `Authorization: Bearer $TOKEN` |
| `credentials_required` | 缺少认证凭据 | 确认 `$TOKEN` 变量已正确设置且非空 |
| 请求返回空 | 网络问题或 API 超时 | 检查网络连接，添加 `-w "\nHTTP %{http_code}"` 到 curl 查看状态码 |

## 安装

### 作为 Hermes Agent Skill 安装

```bash
# 方法 1：从 Skills Hub 安装（发布后）
hermes skills install flowus-api

# 方法 2：从 GitHub 直接安装
hermes skills install https://raw.githubusercontent.com/你的用户名/flowus-skill/main/SKILL.md --name flowus-api

# 方法 3：手动安装
# 将 SKILL.md 复制到 ~/.hermes/skills/ 目录下
mkdir -p ~/.hermes/skills/productivity
cp SKILL.md ~/.hermes/skills/productivity/flowus-api/
```

### 配置 Token

```bash
echo 'FLOWUS_API_TOKEN=你的_token' >> ~/.hermes/.env
```

### 验证安装

```bash
# 重新加载技能后执行
hermes -s flowus-api chat -q "帮我验证 FlowUS 连接是否正常"
```
