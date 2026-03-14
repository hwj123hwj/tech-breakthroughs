# PinchTab 浏览器自动化工具

## 📦 安装

```bash
npm install -g pinchtab
# 或
curl -fsSL https://pinchtab.com/install.sh | bash
```

## ✨ 核心优势

| 特性 | PinchTab | 传统浏览器自动化 |
|------|----------|------------------|
| Token 成本 | ✅ 便宜 5-13x（文本提取 ~800 tokens/page） | ❌ 截图贵（4000-10000 tokens/page） |
| 多实例 | ✅ 原生支持隔离 profiles | ❌ 通常单实例 |
| Dashboard | ✅ Web UI 监控（http://localhost:9867） | ❌ 无可视化 |
| Stealth | ✅ 高级反检测注入 | ⚠️ 基础/无 |
| Profile 持久化 | ✅ 登录一次，永久保持 | ⚠️ 需手动管理 |
| 元素定位 | ✅ 无障碍 refs（如 `e5`），稳定可靠 | ❌ CSS selectors 易失效 |

## 🚀 快速开始

### 启动服务
```bash
pinchtab
# 服务运行在 http://localhost:9867
```

### 创建浏览器实例
```bash
curl -X POST http://localhost:9867/instances/launch \
  -H "Content-Type: application/json" \
  -d '{"name":"work","mode":"headless"}'
# 返回: {"instanceId":"inst_xxx",...}
```

### 打开标签页
```bash
curl -X POST http://localhost:9867/instances/inst_xxx/tabs/open \
  -H "Content-Type: application/json" \
  -d '{"url":"https://example.com"}'
# 返回: {"tabId":"TAB_ID",...}
```

### 获取可交互元素（token 高效！）
```bash
curl "http://localhost:9867/tabs/TAB_ID/snapshot?filter=interactive"
# 只返回可交互元素，大幅节省 tokens
```

### 提取文本内容
```bash
curl "http://localhost:9867/tabs/TAB_ID/text"
# 纯文本提取，~800 tokens/page
```

### 点击元素
```bash
curl -X POST "http://localhost:9867/tabs/TAB_ID/action" \
  -H "Content-Type: application/json" \
  -d '{"kind":"click","ref":"e5"}'
```

### 输入文本
```bash
curl -X POST "http://localhost:9867/tabs/TAB_ID/action" \
  -H "Content-Type: application/json" \
  -d '{"kind":"type","ref":"e3","text":"hello world"}'
```

### 停止实例
```bash
curl -X POST http://localhost:9867/instances/inst_xxx/stop
```

## 🆚 与 OpenClaw 内置 Browser 工具对比

| 特性 | PinchTab | OpenClaw Browser |
|------|----------|------------------|
| Token 成本 | ✅ 便宜 5-13x | ❌ 截图模式贵 |
| 多实例并行 | ✅ 原生支持 | ❌ 单实例 |
| Dashboard | ✅ Web UI 可视化 | ❌ 无 |
| Stealth 反检测 | ✅ 高级注入 | ⚠️ 基础 |
| 接管现有 Chrome 标签 | ❌ | ✅ Chrome Relay |
| 零配置 | ❌ 需安装 | ✅ 开箱即用 |
| 实时交互 | ✅ Dashboard 可视化 | ⚠️ 有限 |

## 🎯 适用场景

### 推荐 PinchTab
- 大量自动化任务（关注 token 成本）
- 多账号并行操作
- 需要长期保持登录状态
- 需要可视化监控浏览器状态
- 反爬虫要求高的网站

### 推荐 OpenClaw 内置 Browser
- 偶尔使用，快速上手
- 需要接管当前 Chrome 标签页
- 不想安装额外工具
- 简单的一次性任务

## 💡 最佳实践

### 1. Profile 持久化
```bash
# 创建带 profile 的实例，登录状态会保存
curl -X POST http://localhost:9867/instances/launch \
  -H "Content-Type: application/json" \
  -d '{"name":"linkedin","mode":"headless","profile":"linkedin-profile"}'
```

### 2. 并行处理多个网站
```bash
# 创建多个实例，同时操作不同网站
curl -X POST http://localhost:9867/instances/launch \
  -H "Content-Type: application/json" \
  -d '{"name":"site1","mode":"headless"}'

curl -X POST http://localhost:9867/instances/launch \
  -H "Content-Type: application/json" \
  -d '{"name":"site2","mode":"headless"}'
```

### 3. 节省 Token 的 snapshot 策略
```bash
# 只获取可交互元素，避免全页截图
curl "http://localhost:9867/tabs/TAB_ID/snapshot?filter=interactive"

# 或者只提取文本内容
curl "http://localhost:9867/tabs/TAB_ID/text"
```

### 4. 错误处理
```bash
# 检查实例状态
curl http://localhost:9867/instances

# 检查标签页状态
curl http://localhost:9867/tabs/TAB_ID
```

## 🔗 相关链接

- 官网：https://pinchtab.com
- 文档：https://pinchtab.com/docs
- 安装脚本：https://pinchtab.com/install.sh

---

**更新时间**：2026-03-13
**来源**：OpenClaw 工作环境实际使用经验