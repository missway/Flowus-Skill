# FlowUS API Skill

通过 FlowUS 官方 REST API 连接、读取、写入 FlowUS（息流）工作区的页面、数据库和内容块。

这是一个**通用 API 参考文档**，不绑定特定 AI Agent——Hermes Agent、Claude Code、Cursor、Windsurf、Codex 等任何能执行 curl 或 HTTP 请求的 Agent 均可使用。

## 功能

- 🔍 搜索工作区中的页面和数据库
- 📄 读取/创建/更新页面
- ✏️ 追加/更新/删除内容块（paragraph、heading、list、code 等）
- 🗄️ 管理数据库 schema（字段、选项）和记录
- 🔑 支持 15+ 种属性字段类型

## 前置准备

### 获取 API Token

1. 登录 [FlowUS](https://flowus.cn)
2. 进入 **设置 → 集成 → 创建机器人集成**
3. 复制生成的 Token，存入环境变量：

```bash
export FLOWUS_API_TOKEN=你的_token
```

### Token 验证

```bash
curl -s "https://api.flowus.cn/v1/users/me" \
  -H "Authorization: Bearer $FLOWUS_API_TOKEN"
```

返回用户信息即表示 Token 有效。

## 使用示例

```bash
# 搜索页面
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
  -d '{"children":[{"object":"block","type":"paragraph","data":{"rich_text":[{"type":"text","text":{"content":"Hello FlowUS!"}}]}}]}'
```

## 在 Hermes Agent 中使用

本仓库的 `SKILL.md` 是标准的 Hermes Agent 技能格式，可直接安装：

```bash
mkdir -p ~/.hermes/skills/productivity/flowus-api
cp SKILL.md ~/.hermes/skills/productivity/flowus-api/
echo 'FLOWUS_API_TOKEN=你的_token' >> ~/.hermes/.env
```

## 在其他 Agent 中使用

- **Claude Code**：将 SKILL.md 内容作为文档上下文提供，或直接引用此处 curl 命令
- **Codex / Cursor / Windsurf**：配合已有的 [flowus-mcp-server](https://github.com/missway/flowus-mcp-server) 使用 MCP 协议更佳
- **通用**：本文档中的所有 curl 命令可在任何 shell 环境中执行

## API 参考

完整 API 规范见 [SKILL.md](SKILL.md)，涵盖：

- 用户操作（`GET /v1/users/me`）
- 搜索（`POST /v1/search`）
- 页面 CRUD（`GET/PATCH/POST /v1/pages`）
- Block 操作（`GET/PATCH/DELETE /v1/blocks`）
- 数据库操作（`GET/PATCH/POST /v1/databases`）

## 数据来源

- [FlowUS 开发者文档](https://flowus.cn/share/df7cd54f-1c21-4fc1-9fd8-ce81be1918a5)
- [FlowUS 官方 SDK (GitHub)](https://github.com/next-space/flowus-api-sdk)

## License

MIT
