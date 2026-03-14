# SiliconFlow 模型选择经验

## 模型对比

### ASR 语音转文本

| 模型 | 格式支持 | 识别质量 | 免费 |
|------|----------|----------|------|
| `TeleAI/TeleSpeechASR` | WAV | ✅ 准确，无干扰 | 是 |
| `FunAudioLLM/SenseVoiceSmall` | MP4/WAV | ⚠️ 有 emoji 干扰 | 是 |

**推荐**：`TeleAI/TeleSpeechASR`

注意：电信模型需要先转 WAV 格式：
```bash
ffmpeg -i video.mp4 -vn -acodec pcm_s16le -ar 16000 -ac 1 audio.wav
```

### Embedding 向量嵌入

| 模型 | 最大 tokens | 向量维度 | 免费 |
|------|-------------|----------|------|
| `BAAI/bge-m3` | **8192** | 1024 | 是 |
| `BAAI/bge-large-zh-v1.5` | 512 | 1024 | 是 |
| `Qwen/Qwen3-Embedding-0.6B` | 32768 | 1024 | 是 |

**推荐**：`BAAI/bge-m3`
- 支持长文本（8192 tokens）
- 与现有数据库兼容（1024 维）

### AI 文本生成（摘要等）

| 模型 | 用途 | 免费 |
|------|------|------|
| `Qwen/Qwen2.5-7B-Instruct` | 通用对话、摘要生成 | 是 |
| `DeepSeek-V3` | 复杂推理 | 是 |

**推荐**：`Qwen/Qwen2.5-7B-Instruct` 用于生成摘要

---

## 知识库技能最佳实践

### 入库流程

1. **获取内容** - 从 URL 或用户提供的内容
2. **生成 AI 摘要** - 一句话总结内容要点
3. **生成 Embedding** - 使用 title + ai_summary，语义更精准
4. **保存到数据库** - PostgreSQL + pgvector

### 小红书视频入库

```bash
# 1. 登录小红书
uv run ~/.agents/skills/xiaohongshu-skills/scripts/cli.py login

# 2. 一键入库
uv run ~/.agents/skills/knowledge-skill/scripts/knowledge_save_from_url.py \
  --url "https://www.xiaohongshu.com/discovery/item/xxx"
```

流程：
- Playwright 拦截视频 URL
- ffmpeg 转 WAV
- TeleSpeechASR 转录
- Qwen 生成摘要
- bge-m3 生成向量

---

**更新时间**：2026-03-14