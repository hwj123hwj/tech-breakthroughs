# OpenClaw 浏览器自动化配置指南

> 日期：2026-03-15
> 环境：Ubuntu 24.04 LTS (Noble Numbat) / 服务器环境（无桌面）

## 问题描述

在 Ubuntu 24.04 服务器上使用 OpenClaw 的浏览器自动化功能时，遇到以下错误：

```
timed out. Restart the OpenClaw gateway
```

浏览器工具无法启动，持续超时。

## 问题根源分析

### 1. Chromium 是 Snap 版本

Ubuntu 24.04 的 `chromium-browser` 包已经变成 **snap 的过渡包**：

```bash
$ which chromium-browser && chromium-browser --version
/usr/bin/chromium-browser
Chromium 145.0.7632.116 snap
```

Snap 版 Chromium 有以下问题：
- **AppArmor 限制**：阻止某些 D-Bus 调用
- **Sandbox 问题**：在服务器环境（无桌面）有很多限制
- **启动超时**：OpenClaw 的浏览器启动器无法在预期时间内完成

错误日志示例：
```
AppArmor policy prevents this sender from sending this message to this recipient
```

### 2. 系统代理配置问题

`~/.curlrc` 配置了代理，但没有排除本地地址：

```bash
$ cat ~/.curlrc
proxy=http://127.0.0.1:7890
```

这导致本地请求（如 `http://127.0.0.1:18800`）被代理拦截，返回 502 错误。

## 解决方案

### Step 1: 安装 Google Chrome（非 Snap 版）

```bash
# 下载 Chrome deb 包
cd /tmp
curl -L -o chrome.deb https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb

# 安装（会自动修复依赖）
sudo dpkg -i chrome.deb
sudo apt-get install -f -y

# 验证安装
$ which google-chrome && google-chrome --version
/usr/bin/google-chrome
Google Chrome 146.0.7680.80
```

### Step 2: 修复代理配置

编辑 `~/.curlrc`，添加 `noproxy` 排除本地地址：

```bash
proxy=http://127.0.0.1:7890
noproxy=localhost,127.0.0.1
```

### Step 3: 配置 OpenClaw 使用 Chrome

编辑 `~/.openclaw/openclaw.json`，添加浏览器配置：

```json
{
  "browser": {
    "executablePath": "/usr/bin/google-chrome",
    "cdpPort": 18800,
    "headless": true,
    "autoStartTimeoutMs": 30000
  }
}
```

**配置说明**：
- `executablePath`: Chrome 可执行文件路径
- `cdpPort`: Chrome DevTools Protocol 端口
- `headless`: 无头模式（服务器环境必须）
- `autoStartTimeoutMs`: 启动超时时间（默认 12s，建议增加到 30s）

### Step 4: 重启 Gateway

```bash
# 停止旧进程
pkill -9 -f openclaw-gateway
pkill -9 chrome

# 启动新进程
openclaw gateway restart
```

### Step 5: 验证浏览器

```bash
# 手动测试 Chrome 是否能正常启动
google-chrome --headless --no-sandbox --disable-gpu --dump-dom https://example.com

# 测试 CDP 连接
curl --noproxy '*' http://127.0.0.1:18800/json/version
```

预期输出：
```json
{
   "Browser": "Chrome/146.0.7680.80",
   "Protocol-Version": "1.3",
   "webSocketDebuggerUrl": "ws://127.0.0.1:18800/devtools/browser/..."
}
```

## 关键命令总结

| 用途 | 命令 |
|------|------|
| 检查浏览器版本 | `chromium-browser --version` |
| 安装 Chrome | `sudo dpkg -i google-chrome.deb && sudo apt-get install -f -y` |
| 测试 headless | `google-chrome --headless --no-sandbox --dump-dom https://example.com` |
| 检查代理配置 | `cat ~/.curlrc` |
| 测试 CDP 连接 | `curl --noproxy '*' http://127.0.0.1:18800/json/version` |
| 查看浏览器状态 | `browser action=status` (在 OpenClaw 中) |

## 经验教训

1. **Ubuntu 24.04 的 chromium-browser 是 snap 版**，不适合服务器自动化
2. **服务器环境必须用 headless 模式**
3. **代理配置要排除本地地址**，否则 CDP 连接会失败
4. **增加 autoStartTimeoutMs 可以避免启动超时**

## 相关文件

- OpenClaw 配置：`~/.openclaw/openclaw.json`
- 代理配置：`~/.curlrc`
- Chrome 安装位置：`/usr/bin/google-chrome`

## 参考

- [OpenClaw 文档](https://docs.openclaw.ai)
- [Chrome Headless 模式](https://developer.chrome.com/docs/chromium/headless)
- [Ubuntu 24.04 Snap Chromium 问题](https://snapcraft.io/chromium)