# 飞书机器人发送视频的正确方式

## 问题描述

使用 OpenClaw 的 `message` 工具发送视频文件时，收到的不是视频卡片，而是文件形式。用户需要手动下载才能播放，体验不好。

## 根本原因

飞书 API 对不同文件类型有不同的消息类型：
- **PDF/CSV/文档** → `msg_type: file`
- **视频/音频** → `msg_type: media`

OpenClaw 的飞书插件在处理 `media` 参数时，底层可能没有正确区分这两种类型，导致视频被当作普通文件发送。

## 解决方案：直接调用飞书 API

### 完整流程

```
1. 获取 tenant_access_token
2. 上传视频获取 file_key
3. 用 msg_type: media 发送消息
```

### 代码实现

#### 1. 获取 Token

```bash
curl -X POST "https://open.feishu.cn/open-apis/auth/v3/tenant_access_token/internal" \
  -H "Content-Type: application/json" \
  -d '{"app_id":"YOUR_APP_ID","app_secret":"YOUR_APP_SECRET"}'
```

#### 2. 上传视频

```bash
curl -X POST "https://open.feishu.cn/open-apis/im/v1/files" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -F "file_type=mp4" \
  -F "file_name=视频名.mp4" \
  -F "file=@/path/to/video.mp4"
```

返回：
```json
{"code":0,"data":{"file_key":"file_v3_xxxx"},"msg":"success"}
```

#### 3. 发送视频消息

```bash
curl -X POST "https://open.feishu.cn/open-apis/im/v1/messages?receive_id_type=open_id" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "receive_id": "ou_xxxx",
    "msg_type": "media",
    "content": "{\"file_key\": \"file_v3_xxxx\"}"
  }'
```

### 关键参数

| 参数 | 说明 |
|------|------|
| `file_type` | 必须与实际格式一致（mp4/mov 等） |
| `msg_type` | 必须是 `media`，不是 `file` |
| `receive_id_type` | 用 `open_id` 或 `user_id`，注意权限 |
| 文件大小 | ≤ 30MB |

### 常见错误

1. **msg_type 错误**：用了 `file` 发视频 → 改成 `media`
2. **权限不足**：用 `user_id` 时需要 `contact:user.employee_id:readonly` 权限 → 改用 `open_id`
3. **file_type 不匹配**：上传时写 `pdf`，实际是 `mp4` → 必须一致
4. **file_key 错误**：用了云文档的 `file_token` → 必须用 `im/v1/files` 上传返回的 `file_key`

## 效果对比

| 方式 | 收到的形式 |
|------|-----------|
| OpenClaw message + filePath | 文件（需下载） |
| OpenClaw message + media | 文件（需下载）⚠️ |
| 飞书 API + msg_type: media | 视频卡片（可直接播放）✅ |

## 进阶：带预览图

飞书视频默认没有预览图，需要手动上传封面图并在发送时指定 `image_key`。

### 完整流程

```bash
# 1. 截取视频第1秒作为封面图
ffmpeg -y -i video.mp4 -ss 00:00:01 -vframes 1 cover.jpg

# 2. 上传封面图
curl -X POST "https://open.feishu.cn/open-apis/im/v1/images" \
  -H "Authorization: Bearer $TOKEN" \
  -F "image_type=message" \
  -F "image=@cover.jpg"
# 返回：{"data":{"image_key":"img_v3_xxxx"}}

# 3. 发送时带 image_key
curl -X POST "https://open.feishu.cn/open-apis/im/v1/messages?receive_id_type=open_id" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "receive_id": "ou_xxxx",
    "msg_type": "media",
    "content": "{\"file_key\": \"file_v3_xxx\", \"image_key\": \"img_v3_xxx\"}"
  }'
```

### 关于时长（duration）

飞书 API 支持 `duration` 参数（毫秒），但测试发现：
- API 返回的 duration 始终为 0
- 飞书会自动从视频文件提取时长
- 用户反馈：**不点开视频时显示时长为 0，点开后正常**

这可能是飞书客户端的显示问题，目前没有找到通过 API 强制显示时长的方法。

### 注意事项

- `image_key` 必须和 `file_key` 在同一个 token 会话中上传
- 预览图建议从视频中间截取，避免黑屏
- 封面图格式支持 jpg/png

## 后续优化

OpenClaw 飞书插件需要修改，根据文件 MIME 类型自动选择 `file` 或 `media` 消息类型。

---

**日期**: 2026-03-06
**环境**: OpenClaw + 飞书机器人 + macOS
**文件**: 抖音视频.mp4 (2.7MB)