# Widget 组件文档

## 1. 概述

Basjoo Widget 是一个可嵌入网站的聊天组件，提供实时 AI 客服功能。

## 2. 特性

- 📱 响应式设计
- 🌐 多语言支持
- 💬 流式响应 (SSE)
- 💾 会话持久化 (localStorage)
- 🎨 可定制外观
- 🔒 来源追踪

## 3. 使用方式

### 3.1 基础嵌入

在网页中添加以下代码:

```html
<script src="https://your-basjoo-domain.com/sdk.js"></script>
<script>
  window.BasjooWidget.init({
    agentId: 'agt_1234567890ab',
    apiBase: 'https://your-basjoo-domain.com'
  });
</script>
```

### 3.2 完整配置

```html
<script src="https://your-basjoo-domain.com/sdk.js"></script>
<script>
  window.BasjooWidget.init({
    agentId: 'agt_1234567890ab',
    apiBase: 'https://your-basjoo-domain.com',
    themeColor: '#06B6D4',
    title: '在线客服',
    welcomeMessage: '您好，有什么可以帮助您的？',
    language: 'auto',
    position: 'right',
    theme: 'light'
  });
</script>
```

## 4. 配置选项

| 选项 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `agentId` | string | 必需 | Agent ID |
| `apiBase` | string | 自动检测 | API 基础地址 |
| `themeColor` | string | '#06B6D4' | 主题颜色 |
| `title` | string | 'AI 客服' | Widget 标题 |
| `welcomeMessage` | string | - | 欢迎消息 |
| `language` | string | 'auto' | 语言设置 |
| `position` | 'left' \| 'right' | 'right' | 位置 |
| `theme` | 'light' \| 'dark' \| 'auto' | 'auto' | 主题 |

## 5. API

### 5.1 init()

初始化 Widget。

```typescript
window.BasjooWidget.init(config: WidgetConfig): void
```

### 5.2 open()

打开聊天窗口。

```typescript
window.BasjooWidget.open(): void
```

### 5.3 close()

关闭聊天窗口。

```typescript
window.BasjooWidget.close(): void
```

### 5.4 sendMessage()

发送消息。

```typescript
window.BasjooWidget.sendMessage(message: string): void
```

### 5.5 destroy()

销毁 Widget 实例。

```typescript
window.BasjooWidget.destroy(): void
```

## 6. 事件

### 6.1 事件监听

```typescript
window.BasjooWidget.on('event', callback);

window.BasjooWidget.on('message', (message) => {
  console.log('收到消息:', message);
});
```

### 6.2 支持的事件

| 事件 | 说明 | 数据 |
|------|------|------|
| `ready` | Widget 初始化完成 | - |
| `open` | 聊天窗口打开 | - |
| `close` | 聊天窗口关闭 | - |
| `message` | 收到消息 | ChatMessage |
| `send` | 发送消息 | ChatMessage |
| `error` | 发生错误 | Error |

## 7. 数据类型

### 7.1 WidgetConfig

```typescript
interface WidgetConfig {
  agentId: string;
  apiBase?: string;
  themeColor?: string;
  logoUrl?: string;
  title?: string;
  welcomeMessage?: string;
  language?: 'auto' | string;
  position?: 'left' | 'right';
  theme?: 'light' | 'dark' | 'auto';
}
```

### 7.2 ChatMessage

```typescript
interface ChatMessage {
  role: 'user' | 'assistant';
  content: string;
  sources?: Source[];
  timestamp: Date;
}
```

### 7.3 Source

```typescript
interface Source {
  type: 'url' | 'qa';
  title?: string;
  url?: string;
  snippet?: string;
  question?: string;
  id?: string;
}
```

## 8. 样式定制

### 8.1 CSS 变量

```css
:root {
  --basjoo-primary: #06B6D4;
  --basjoo-bg: #ffffff;
  --basjoo-text: #1f2937;
  --basjoo-border: #e5e7eb;
}
```

### 8.2 自定义样式

```html
<style>
  .basjoo-widget {
    --basjoo-primary: #FF5733;
  }
</style>
```

## 9. 流式响应

Widget 支持 SSE 流式响应，提供实时聊天体验。

### 9.1 SSE 事件

```text
event: content
data: {"content": "您", "sources": [], "elapsed": 0}

event: content
data: {"content": "好", "sources": [], "elapsed": 50}

event: done
data: {"message_id": 123, "session_id": "...", "usage": {...}}
```

### 9.2 错误处理

```typescript
window.BasjooWidget.on('error', (error) => {
  console.error('Widget 错误:', error);
});
```

## 10. 会话管理

### 10.1 localStorage 键

| 键 | 说明 |
|------|------|
| `basjoo_visitor_id` | 访客 ID |
| `basjoo_session_id` | 会话 ID |
| `basjoo_messages` | 聊天消息历史 |

### 10.2 会话持久化

Widget 自动保存聊天历史到 localStorage，并在下次访问时恢复。

## 11. 多语言

### 11.1 自动检测

设置 `language: 'auto'` 时，Widget 会自动检测访客浏览器语言。

### 11.2 手动设置

```typescript
window.BasjooWidget.init({
  agentId: 'agt_xxx',
  language: 'zh-CN'  // 强制使用中文
});
```

## 12. 源追踪

### 12.1 来源显示

AI 回复中的引用会自动显示为可点击链接。

### 12.2 来源格式

```markdown
根据我们的[产品介绍](https://example.com/products)，...
```

## 13. 人工接管

### 13.1 接管流程

1. AI 回复标记 `taken_over: true`
2. 管理员在后台接管会话
3. Widget 自动切换到轮询模式
4. 显示管理员回复

### 13.2 轮询机制

当会话被接管时，Widget 每 3 秒轮询一次获取新消息。

## 14. 错误码

| 错误码 | 说明 |
|--------|------|
| `API_KEY_INVALID` | API 密钥无效 |
| `API_KEY_MISSING` | API 密钥缺失 |
| `PROVIDER_RATE_LIMITED` | 提供商限流 |
| `PROVIDER_UNAVAILABLE` | 提供商不可用 |
| `MODEL_NOT_FOUND` | 模型未找到 |
| `NETWORK_ERROR` | 网络错误 |

## 15. 构建

### 15.1 开发构建

```bash
cd widget
npm run build:dev
```

输出: `dist/basjoo-widget.js`

### 15.2 生产构建

```bash
npm run build:prod
```

输出: `dist/basjoo-widget.min.js`

### 15.3 同步到后端

```bash
npm run sync-widget
```

## 16. 示例

### 16.1 基础示例

```html
<!DOCTYPE html>
<html>
<head>
  <title>Basjoo Widget Demo</title>
</head>
<body>
  <h1>欢迎访问</h1>
  
  <script src="/sdk.js"></script>
  <script>
    window.BasjooWidget.init({
      agentId: 'agt_1234567890ab'
    });
  </script>
</body>
</html>
```

### 16.2 完整示例

```html
<!DOCTYPE html>
<html>
<head>
  <title>Custom Widget Demo</title>
  <style>
    .custom-widget {
      --basjoo-primary: #FF5733;
      --basjoo-border-radius: 20px;
    }
  </style>
</head>
<body>
  <script src="/sdk.js"></script>
  <script>
    window.BasjooWidget.init({
      agentId: 'agt_1234567890ab',
      apiBase: 'https://your-basjoo.com',
      themeColor: '#FF5733',
      title: '技术支持',
      welcomeMessage: '您好，有什么技术问题可以帮助您？',
      language: 'auto',
      position: 'left',
      theme: 'light'
    });
    
    // 监听事件
    window.BasjooWidget.on('ready', () => {
      console.log('Widget 已就绪');
    });
    
    window.BasjooWidget.on('message', (msg) => {
      console.log('新消息:', msg);
    });
  </script>
</body>
</html>
```
