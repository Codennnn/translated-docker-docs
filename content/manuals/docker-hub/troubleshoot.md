---
description: 了解如何排查常见的 Docker Hub 问题。
keywords: Docker Hub, 故障排查
title: Docker Hub 故障排查
linkTitle: 故障排查
weight: 60
tags: [Troubleshooting]
toc_max: 2
---

如果你在使用 Docker Hub 时遇到问题，可参考以下解决方案。

## 达到拉取频率上限（429 状态码）

### 错误信息

出现该问题时，你会在 Docker CLI 或 Docker Engine 日志中看到如下报错：

```text
You have reached your pull rate limit. You may increase the limit by authenticating and upgrading: https://www.docker.com/increase-rate-limits
```

### 可能原因

- 作为已登录的 Docker Personal 用户，已达到你的拉取频率上限。
- 作为未登录用户，基于你的 IPv4 地址或 IPv6 /64 子网，已达到拉取频率上限。

### 解决方案

可以尝试以下任一方案：

- [登录](./usage/pulls.md#authentication)或[升级](../subscription/change.md#upgrade-your-subscription)你的 Docker 账号。
- [查看你的拉取频率限制](./usage/pulls.md#view-hourly-pull-rate-and-limit)，等待额度恢复后再重试。

## 请求过多（429 状态码）

### 错误信息

出现该问题时，你会在 Docker CLI 或 Docker Engine 日志中看到如下报错：

```text
Too Many Requests
```

### 可能原因

- 触发了 [Abuse rate limit](./usage/_index.md#abuse-rate-limit)（滥用速率限制）。

### 解决方案

1. 检查是否有异常的 CI/CD 流水线在访问 Docker Hub，并进行修复。
2. 在自动化脚本中实现带退避的重试机制，避免每分钟发送成千上万的请求。

## 500 状态码

### 错误信息

出现该问题时，你可能会在 Docker CLI 或 Docker Engine 日志中看到如下报错：

```text
Unexpected status code 500
```

### 可能原因

- Docker Hub 服务可能出现了临时性问题。

### 解决方案

1. 访问 [Docker 系统状态页](https://www.dockerstatus.com/)，确认各项服务是否正常。
2. 稍后重试访问 Docker Hub。这可能只是暂时性问题。
3. 如问题仍然存在，请[联系 Docker 支持](https://www.docker.com/support/)反馈。