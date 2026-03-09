# OpenClaw Cron 任务经验教训

## 问题描述

在设置 OpenClaw 定时任务提醒"买牙膏"时，任务虽然在设定时间执行了，但飞书消息发送失败，导致用户没有收到提醒。

## 问题细节

### 任务创建命令

```bash
# 第一次尝试（UTC 时间 19:00，即北京时间凌晨 3:00）
openclaw cron create --name "买牙膏提醒" --at "2026-03-09T19:00:00Z" --message "记得买牙膏！"

# 第二次尝试（北京时间 21:00）
openclaw cron create --name "买牙膏提醒" --at "2026-03-09T13:00:00Z" --message "记得买牙膏！" --delete-after-run
```

### 错误日志

```
cron: job added
...
cron: disabling one-shot job after error
error: Delivering to Feishu requires target <chatId|user:openId|chat:chatId>
reason: permanent error
```

### 问题分析

1. ✅ **任务创建成功** - cron job 被正确创建
2. ✅ **任务按时执行** - 在 13:00 UTC（北京时间 21:00）触发
3. ❌ **消息发送失败** - 缺少 `--to` 参数，导致飞书不知道发给谁

## 解决方案

### 正确的命令

```bash
openclaw cron create \
  --name "买牙膏提醒" \
  --at "2026-03-09T13:00:00Z" \
  --message "记得买牙膏！" \
  --delete-after-run \
  --to "user:ou_b92cf79b91e36efefef21dea8e19c587"
```

### 关键点

1. **必须指定 `--to` 参数**
   - 飞书使用 `user:<openId>` 格式
   - 例如：`user:ou_b92cf79b91e36efefef21dea8e19c587`

2. **时区转换**
   - OpenClaw cron 使用 **UTC 时间**
   - 中国是 **UTC+8**，需要手动转换
   - 北京时间 21:00 = UTC 13:00

3. **任务类型**
   - `--at`: 单次执行任务
   - `--cron`: 周期性任务
   - `--every`: 间隔执行任务

## 经验总结

### ✅ 正确做法

```bash
# 单次提醒（UTC 时间）
openclaw cron create \
  --name "任务名称" \
  --at "2026-03-09T13:00:00Z" \
  --message "提醒内容" \
  --to "user:<openId>"

# 周期性提醒（每天 21:00 UTC）
openclaw cron create \
  --name "每日提醒" \
  --cron "0 21 * * *" \
  --message "每天提醒" \
  --to "user:<openId>"

# 间隔提醒（每 2 小时）
openclaw cron create \
  --name "间隔提醒" \
  --every "2h" \
  --message "间隔提醒" \
  --to "user:<openId>"
```

### ❌ 常见错误

1. **缺少 `--to` 参数**
   ```bash
   # 错误
   openclaw cron create --name "提醒" --message "内容"

   # 正确
   openclaw cron create --name "提醒" --message "内容" --to "user:xxx"
   ```

2. **时区错误**
   ```bash
   # 错误：认为本地时间是 UTC
   openclaw cron create --at "2026-03-09T21:00:00Z"

   # 正确：转换时区
   北京时间 21:00 = UTC 13:00
   openclaw cron create --at "2026-03-09T13:00:00Z"
   ```

3. **时间格式错误**
   ```bash
   # 错误
   openclaw cron create --at "2026-03-09 21:00"

   # 正确：ISO 8601 格式
   openclaw cron create --at "2026-03-09T13:00:00Z"
   ```

## 其他注意事项

### 查看任务列表

```bash
openclaw cron list
```

### 删除任务

```bash
# 查看任务 ID
openclaw cron list

# 删除任务
openclaw cron delete <任务ID>
```

### 临时文件清理

```bash
# 清理日志
rm -rf ~/.openclaw/logs/*

# 清理临时文件
rm -rf /tmp/tech-breakthroughs
```

### 查看执行日志

```bash
# 查看今天的日志
grep "买牙膏" /tmp/openclaw/openclaw-*.log

# 查看所有 cron 相关日志
grep "cron:" ~/.openclaw/logs/*.log
```

## 实用脚本

### 自动转换时区的提醒

```bash
#!/bin/bash
# 北京时间 → UTC 转换
beijing_time="21:00"
date=$(date -d "$beijing_time" -u +"%Y-%m-%dT%H:%M:%SZ")

openclaw cron create \
  --name "提醒" \
  --at "$date" \
  --message "内容" \
  --to "user:ou_b92cf79b91e36efefef21dea8e19c587"
```

### 常用飞书目标

```bash
# 个人消息
--to "user:ou_xxx"

# 群组消息
--to "chat:oc_xxx"

# 指定频道
--channel "feishu"
```

## 相关资源

- OpenClaw 文档: https://docs.openclaw.ai
- 飞书开放平台: https://open.feishu.cn/document/
- ISO 8601 时间格式: https://en.wikipedia.org/wiki/ISO_8601

## 更新记录

- 2026-03-09: 初始版本 - 记录 cron 任务飞书推送失败问题
