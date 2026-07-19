---
name: app-dev-caveats
description: 在中国环境开发 Web APP 的注意事项和踩坑记录
metadata:
  type: project
---

# APP 开发注意事项

## 一、网络环境

### 1.1 GitHub 访问
- **问题**：GitHub 在国内被墙，HTTPS 推送经常超时
- **解决**：使用 SSH over port 443（ssh.github.com:443），配置 `~/.ssh/config`
- **GitHub Pages** 部署后可正常访问（github.io 未被墙）
- **不要用 Gitee Pages**：需要实名认证，门槛高，不实用

### 1.2 外部 API/服务的可达性
- **Google 服务全部被墙**（包括 Chrome Web Speech API 的语音服务器）
- **规则**：任何依赖 Google 服务器的浏览器 API 在国内都不可用
- 在承诺实现功能前，先确认该功能依赖的服务在国内是否可达

## 二、PWA Service Worker

### 2.1 缓存策略
- **错误做法**：cache-first（缓存优先）→ 用户永远看不到更新
- **正确做法**：HTML 页面用 network-first（网络优先），静态资源用 cache-first
- 每次更新必须修改 CACHE_NAME（如 `expense-tracker-v10`），否则旧 SW 不会更新

### 2.2 SW 生命周期
- SW 更新不是即时的——浏览器在后台检测新 SW，安装，等待旧 SW 释放后激活
- `skipWaiting()` + `clients.claim()` 可以加速，但用户首次打开仍可能是旧版
- 用户需要**彻底关闭 App 再重开**才能看到新版本

## 三、手机端兼容性

### 3.1 Web Speech API（语音识别）
- Chrome Android → 走 Google 服务器 → **国内被墙，不可用**
- Safari iOS → 走 Apple Siri 服务器 → 国内可用
- **结论**：语音识别只能做 iOS 端，Android 端需要其他方案

### 3.2 getUserMedia（麦克风权限）
- 必须在用户手势的直接回调中调用（async handler 可以，但嵌套 IIFE 不行）
- 权限一旦授权过就不会再弹窗

### 3.3 测试
- 每次改动后必须在手机端实际验证，不能只看桌面端
- 桌面 Chrome 能用的 API 不代表手机 Chrome 能用

## 四、开发流程

1. **承诺功能前先评估**：该功能依赖的外部服务在国内是否可达？
2. **涉及浏览器 API 的功能**：先查 Can I Use 兼容性，再查是否依赖被墙服务
3. **PWA 每次发布后**：必须验证 SW 版本号和缓存策略
4. **反复失败的坑**：及时记录到本文档，不要再踩第二次
