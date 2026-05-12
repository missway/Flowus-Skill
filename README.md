# FlowUS API Skill

通过 FlowUS 官方 REST API 连接、读取、写入 FlowUS（息流）工作区的页面、数据库和内容块。

## 功能

- 🔍 搜索工作区中的页面和数据库
- 📄 读取/创建/更新页面
- ✏️ 追加/更新/删除内容块（paragraph、heading、list、code 等）
- 🗄️ 管理数据库 schema（字段、选项）和记录
- 🔑 支持 15+ 种属性字段类型

## 安装

### 前置依赖

- [Hermes Agent](https://hermes-agent.nousresearch.com) 已安装
- FlowUS 账号

### 获取 API Token

1. 登录 [FlowUS](https://flowus.cn)
2. 进入 **设置 → 集成 → 创建机器人集成**
3. 复制生成的 Token

### 安装技能

```bash
# 安装技能
mkdir -p ~/.hermes/skills/productivity/flowus-api
cp SKILL.md ~/.hermes/skills/productivity/flowus-api/

# 配置 Token
echo 'FLOWUS_API_TOKEN=你的_token' >> ~/.hermes/.env

# 验证
hermes -s flowus-api chat -q "搜索我的 FlowUS 工作区"
```

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
