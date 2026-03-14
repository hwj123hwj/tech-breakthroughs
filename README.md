# 技术突破与经验总结

## 🚀 **关键技术突破**

### 🔧 **绕过 acpx 崩溃问题**
- ❌ **问题**：`acpx + Codex ACP` 组合持续以 **退出码 5** 崩溃，无法完成任务
- ✅ **解决方案**：**codex-orchestrator skill + Codex CLI 直接运行**
- **原理**：绕过 acpx 中间层，直接使用 Codex CLI 执行命令，避免了 ACP 协议的兼容性问题
- **成果**：成功完成 B站推荐系统的 7 个核心模块开发

### 📊 **B站 API 限流应对**
- ❌ **问题**：短时间请求过多导致 API 限流（错误代码 -799）
- ✅ **解决方案**：分批次、低频率请求
- **优化方案**：
  - 每次只检查最近 30 个 UP 主
  - 每个 UP 主只获取 1 个视频
  - 增加请求间隔，避免触发限流保护

### 🛠 **bilibili_api 版本兼容**
- ❌ **问题**：`bilibili_api 4.1.0` 结构与预期不符，`User` 类不存在，API 函数参数不同
- ✅ **解决方案**：
  - 替换 `Credential` 为 `Verify` 类
  - 使用 `get_videos_raw` 代替 `get_videos`
  - 动态适配不同 API 返回结构

### 🌐 **PinchTab 浏览器自动化**
- ✅ **优势**：Token 成本比截图模式便宜 **5-13 倍**
- ✅ **特性**：多实例并行、Profile 持久化、Dashboard 可视化
- ✅ **场景**：大量自动化任务、多账号并行、反爬虫要求高的网站
- 📖 [详细文档](./PinchTab-Browser-Automation.md)

## 🎯 **项目架构**

### B站视频智能推荐系统
- **数据库**：PostgreSQL + pgvector 扩展（向量搜索）
- **核心模块**：7 个 Python 脚本
- **评分算法**：多维度加权评分（Tags 40%、Keywords 30%、Vector 20%、UP 10%）

## 📈 **开发流程**

1. **环境准备**：安装 Docker、PostgreSQL、pgvector
2. **模块开发**：使用 Codex CLI 生成 7 个核心模块
3. **测试优化**：修复 API 兼容问题，优化请求策略
4. **Skill 封装**：将项目封装为 OpenClaw Skill，便于重复使用

## 📚 **详细文档**

- [OpenClaw Cron 使用经验](./OpenClaw-Cron-Lessons.md) - 创建 cron 任务的经验教训
- [PinchTab 浏览器自动化](./PinchTab-Browser-Automation.md) - 高效低成本的浏览器自动化工具
- [SiliconFlow 模型选择](./SiliconFlow-Models.md) - ASR、Embedding、AI 摘要模型对比

## 📝 **经验教训**

1. **工具选择**：当 ACP 协议出现问题时，直接使用 Codex CLI 是更可靠的方案
2. **API 限流**：处理第三方 API 时，必须考虑限流机制，避免账号被限制
3. **版本兼容**：Python 库版本差异可能导致 API 调用失败，需要动态适配
4. **模块化设计**：将项目拆分为多个独立模块，便于开发、测试和维护
5. **文档重要性**：编写详细的 Skill 文档，确保后续可以快速理解和使用
