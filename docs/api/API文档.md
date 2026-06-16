# API 文档

## 基础信息

- **Base URL**: `http://localhost:8000`
- **认证方式**: Bearer Token (JWT)
- **Content-Type**: `application/json`

## 认证 API

### POST /api/admin/register

管理员注册

**请求体**:
```json
{
  "email": "admin@example.com",
  "password": "securepassword",
  "name": "Admin User"
}
```

**响应** (201):
```json
{
  "id": 1,
  "email": "admin@example.com",
  "name": "Admin User",
  "is_active": true,
  "created_at": "2024-01-01T00:00:00Z"
}
```

### POST /api/admin/login

管理员登录

**请求体**:
```json
{
  "email": "admin@example.com",
  "password": "securepassword"
}
```

**响应** (200):
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "bearer"
}
```

### GET /api/admin/me

获取当前用户信息

**Headers**: `Authorization: Bearer <token>`

**响应** (200):
```json
{
  "id": 1,
  "email": "admin@example.com",
  "name": "Admin User",
  "is_active": true,
  "created_at": "2024-01-01T00:00:00Z"
}
```

---

## 公共 API (无需认证)

### GET /api/v1/agent:default

获取默认 Agent 配置

**响应** (200):
```json
{
  "id": "agt_1234567890ab",
  "name": "AI Agent",
  "widget_title": "AI 客服",
  "widget_color": "#06B6D4",
  "welcome_message": "您好！我是 Basjoo 助手",
  "is_active": true
}
```

### POST /api/v1/chat

聊天请求 (非流式)

**请求体**:
```json
{
  "agent_id": "agt_1234567890ab",
  "message": "你好，我想了解产品",
  "locale": "zh-CN",
  "session_id": "visitor_session_001",
  "visitor_id": "visitor_001"
}
```

**响应** (200):
```json
{
  "reply": "您好！很高兴为您服务...",
  "sources": [
    {
      "type": "url",
      "title": "产品介绍",
      "url": "https://example.com/products",
      "snippet": "我们的产品涵盖..."
    }
  ],
  "usage": {
    "prompt_tokens": 150,
    "completion_tokens": 80,
    "total_tokens": 230
  },
  "session_id": "visitor_session_001",
  "message_id": 123,
  "taken_over": false
}
```

### POST /api/v1/chat/stream

流式聊天 (SSE)

**请求体**: 同 `/api/v1/chat`

**响应**: `text/event-stream`

```text
event: content
data: {"content": "您", "sources": [], "elapsed": 0}

event: content
data: {"content": "好", "sources": [], "elapsed": 100}

event: sources
data: {"sources": [...]}

event: done
data: {"message_id": 123, "session_id": "...", "usage": {...}, "taken_over": false}
```

### POST /api/v1/contexts

检索上下文 (测试检索)

**请求体**:
```json
{
  "agent_id": "agt_1234567890ab",
  "query": "产品特点",
  "top_k": 5,
  "locale": "zh-CN"
}
```

**响应** (200):
```json
{
  "contexts": [
    {
      "type": "url",
      "url": "https://example.com/products",
      "title": "产品介绍",
      "score": 0.85
    }
  ]
}
```

---

## Agent 管理 API

### GET /api/v1/agents

列出所有 Agent

**Headers**: `Authorization: Bearer <token>`

**响应** (200):
```json
{
  "agents": [
    {
      "id": "agt_1234567890ab",
      "name": "AI Agent",
      "model": "gpt-4o-mini",
      "provider_type": "openai",
      "is_active": true,
      "url_count": 10,
      "file_count": 5,
      "active_session_count": 3,
      "created_at": "2024-01-01T00:00:00Z"
    }
  ],
  "total": 1
}
```

### POST /api/v1/agents

创建 Agent

**请求体**:
```json
{
  "name": "Support Bot",
  "description": "客户支持机器人",
  "agent_type": "website_support",
  "channel_mode": "web_widget",
  "system_prompt": "你是一个有帮助的客服助手",
  "persona_type": "customer-service",
  "widget_title": "在线客服",
  "welcome_message": "您好，有什么可以帮助您的？"
}
```

**响应** (201):
```json
{
  "id": "agt_newagent123",
  "name": "Support Bot",
  "workspace_id": 1,
  "is_active": true,
  "created_at": "2024-01-01T00:00:00Z"
}
```

### GET /api/v1/agents/{agent_id}

获取 Agent 详情

**响应** (200): 返回完整的 AgentConfig 对象

### PUT /api/v1/agents/{agent_id}

更新 Agent 配置

**请求体** (部分字段):
```json
{
  "name": "New Name",
  "model": "gpt-4o",
  "temperature": 0.8,
  "top_k": 10,
  "similarity_threshold": 0.02,
  "widget_color": "#FF5733"
}
```

### DELETE /api/v1/agents/{agent_id}

软删除 Agent (设置 deleted_at)

**响应** (200):
```json
{
  "message": "Agent deleted successfully"
}
```

---

## URL 管理 API

### POST /api/v1/urls:create

创建 URL 知识源

**请求体**:
```json
{
  "urls": [
    "https://example.com/about",
    "https://example.com/products"
  ]
}
```

**响应** (201):
```json
{
  "urls": [
    {
      "id": 1,
      "url": "https://example.com/about",
      "normalized_url": "https://example.com/about",
      "status": "pending",
      "is_indexed": false,
      "created_at": "2024-01-01T00:00:00Z"
    }
  ],
  "total": 2,
  "quota": {
    "used": 2,
    "max": 500
  },
  "job_id": "job_123456",
  "auto_fetch_queued": true
}
```

### GET /api/v1/urls:list

列出 URL 知识源

**Query 参数**:
- `agent_id` (required): Agent ID
- `status`: 过滤状态 (pending, fetching, success, failed)
- `page`: 页码
- `page_size`: 每页数量

**响应** (200):
```json
{
  "urls": [...],
  "total": 50,
  "quota": {
    "used": 10,
    "max": 500
  }
}
```

### POST /api/v1/urls:refetch

重新抓取 URLs

**请求体**:
```json
{
  "url_ids": [1, 2, 3],
  "force": false
}
```

**响应** (200):
```json
{
  "job_id": "job_789",
  "status": "queued",
  "message": "Refetch job queued"
}
```

### DELETE /api/v1/urls/{url_id}

删除单个 URL

**响应** (200):
```json
{
  "deleted": true
}
```

### POST /api/v1/urls:clear_all

清空所有 URLs

**响应** (200):
```json
{
  "deleted": 50,
  "message": "All URLs cleared"
}
```

### POST /api/v1/urls/crawl

全站爬取

**请求体**:
```json
{
  "url": "https://example.com",
  "max_depth": 2,
  "max_pages": 20
}
```

**响应** (200):
```json
{
  "job_id": "job_crawl_001",
  "status": "queued",
  "discovered": 25,
  "created": 20,
  "message": "Site crawl started"
}
```

---

## 文件管理 API

### POST /api/v1/files:upload

上传知识文件

**Content-Type**: `multipart/form-data`

**表单字段**:
- `files`: 文件 (支持 pdf, txt, csv, md, docx, xlsx)
- `agent_id`: Agent ID

**响应** (200):
```json
{
  "uploaded": 2,
  "failed": 0,
  "files": [
    {
      "id": "kf_file001",
      "filename": "product-manual.pdf",
      "file_size": 1024000,
      "file_type": "pdf",
      "status": "processing",
      "created_at": "2024-01-01T00:00:00Z"
    }
  ],
  "errors": []
}
```

### GET /api/v1/files:list

列出文件

**Query 参数**:
- `agent_id` (required): Agent ID
- `status`: 过滤状态

**响应** (200):
```json
{
  "files": [...],
  "total": 10,
  "quota": {
    "used": 5,
    "max": 20
  }
}
```

### DELETE /api/v1/files/{file_id}

删除文件

---

## 索引管理 API

### POST /api/v1/index:rebuild

重建索引

**请求体**:
```json
{
  "force": false
}
```

- `force=false` (默认): 仅重建未索引的
- `force=true`: 重建所有

**响应** (200):
```json
{
  "job_id": "job_index_001",
  "status": "queued",
  "message": "Index rebuild started"
}
```

### GET /api/v1/index:status

获取索引状态

**响应** (200):
```json
{
  "agent_id": "agt_123",
  "job_id": "job_index_001",
  "status": "running",
  "result": {
    "chunks_indexed": 150,
    "errors": []
  }
}
```

### GET /api/v1/index:info

获取索引信息

**响应** (200):
```json
{
  "agent_id": "agt_123",
  "urls_indexed": 10,
  "files_indexed": 5,
  "index_exists": true,
  "status": "ready"
}
```

---

## 会话管理 API

### GET /api/v1/sessions

列出会话

**Query 参数**:
- `agent_id` (required): Agent ID
- `status`: 过滤状态 (active, taken_over, closed)
- `page`, `page_size`: 分页

**响应** (200):
```json
{
  "items": [
    {
      "id": "sess_abc123",
      "session_id": "visitor_session_001",
      "visitor_id": "visitor_001",
      "visitor_country": "China",
      "visitor_city": "Shanghai",
      "status": "active",
      "message_count": 5,
      "created_at": "2024-01-01T00:00:00Z",
      "updated_at": "2024-01-01T00:05:00Z",
      "last_message": "产品价格是多少？"
    }
  ],
  "total": 100
}
```

### GET /api/v1/chat/messages

获取会话消息

**Query 参数**:
- `session_id` (required): 会话 ID
- `role`: 过滤角色 (user, assistant)

**响应** (200):
```json
{
  "messages": [
    {
      "id": 1,
      "role": "user",
      "content": "你好",
      "created_at": "2024-01-01T00:00:00Z"
    },
    {
      "id": 2,
      "role": "assistant",
      "content": "您好！有什么可以帮助您的？",
      "sources": [...],
      "created_at": "2024-01-01T00:00:01Z"
    }
  ]
}
```

### POST /api/v1/chat/{session_id}/takeover

接管会话

**响应** (200):
```json
{
  "session_id": "sess_abc123",
  "status": "taken_over",
  "message": "Session taken over successfully"
}
```

### PUT /api/v1/sessions/{session_id}/close

关闭会话

**响应** (200):
```json
{
  "id": "sess_abc123",
  "status": "closed"
}
```

---

## 配额 API

### GET /api/v1/quota

获取配额信息

**响应** (200):
```json
{
  "max_agents": 10,
  "max_urls": 500,
  "max_files": 100,
  "max_messages_per_day": 1500,
  "max_total_text_mb": 20,
  "used_agents": 2,
  "used_urls": 50,
  "used_files": 10,
  "used_messages_today": 150,
  "used_total_text_mb": 2.5,
  "remaining_urls": 450,
  "remaining_files": 90,
  "remaining_messages_today": 1350
}
```

---

## 知识库 API (多租户)

### POST /api/v1/tenants/{tenant_id}/knowledge_bases

创建知识库

**请求体**:
```json
{
  "name": "Product KB",
  "embedding_model": "BAAI/bge-m3",
  "chunk_size": 512,
  "chunk_overlap": 64
}
```

### GET /api/v1/tenants/{tenant_id}/knowledge_bases

列出知识库

### GET /api/v1/tenants/{tenant_id}/knowledge_bases/{kb_id}

获取知识库详情

### PUT /api/v1/tenants/{tenant_id}/knowledge_bases/{kb_id}

更新知识库配置

### DELETE /api/v1/tenants/{tenant_id}/knowledge_bases/{kb_id}

删除知识库

### POST /api/v1/tenants/{tenant_id}/knowledge_bases/{kb_id}/documents

上传文档

**Content-Type**: `multipart/form-data`
- `data`: JSON 字符串 (metadata)
- `file`: 文件

**响应** (202):
```json
{
  "uploaded": 1,
  "failed": 0,
  "documents": [
    {
      "id": "doc_uuid",
      "filename": "guide.pdf",
      "status": "pending"
    }
  ]
}
```

### GET /api/v1/tenants/{tenant_id}/knowledge_bases/{kb_id}/documents

列出文档

### DELETE /api/v1/tenants/{tenant_id}/knowledge_bases/{kb_id}/documents/{doc_id}

删除文档

### GET /api/v1/tenants/{tenant_id}/knowledge_bases/{kb_id}/documents/{doc_id}/progress

获取处理进度

---

## 模型列表 API

### POST /api/v1/models/list

获取可用模型列表

**请求体**:
```json
{
  "provider_type": "deepseek",
  "api_key": "sk-xxx"  // 可选，使用保存的密钥
}
```

**响应** (200):
```json
{
  "models": [
    "deepseek-chat",
    "deepseek-coder"
  ]
}
```

---

## 错误响应

### 格式

```json
{
  "detail": "错误描述信息"
}
```

### 常见错误

| 状态码 | 说明 |
|--------|------|
| 400 | 请求参数错误 |
| 401 | 认证失败 |
| 403 | 权限不足 |
| 404 | 资源不存在 |
| 410 | 资源已删除 |
| 413 | 请求体过大 (最大 10MB) |
| 422 | 数据验证失败 |
| 429 | 请求过于频繁 |
| 500 | 服务器错误 |

### LLM 错误码

| code | 说明 |
|------|------|
| `API_KEY_INVALID` | API 密钥无效 |
| `API_KEY_MISSING` | API 密钥缺失 |
| `PROVIDER_RATE_LIMITED` | 提供商限流 |
| `PROVIDER_UNAVAILABLE` | 提供商不可用 |
| `MODEL_NOT_FOUND` | 模型未找到 |
| `PROVIDER_ERROR` | 其他提供商错误 |
