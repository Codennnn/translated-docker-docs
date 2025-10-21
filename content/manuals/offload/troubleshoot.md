---
title: Docker Offload 故障排除
linktitle: 故障排除
weight: 800
description: 了解如何解决 Docker Offload 相关问题。
keywords: 云, 故障排除, 云模式, Docker Desktop, 云构建器, 使用
tags: [Troubleshooting]
---

使用 Docker Offload 需要满足以下条件：

- 身份验证
- 活跃的互联网连接
- 没有限制性代理或防火墙阻止访问 Docker Cloud 的流量
- Docker Offload Beta 版访问权限
- Docker Desktop 4.43 或更高版本

Docker Desktop 使用 Offload 在云端运行构建和容器。如果构建或容器运行失败、回退到本地运行或报告会话错误，请使用以下步骤帮助解决问题。

1. 确保 Docker Desktop 中已启用 Docker Offload：

   1. 打开 Docker Desktop 并登录。
   2. 前往 **设置** > **Beta 功能**。
   3. 确保 **Docker Offload** 已勾选。

2. 使用以下命令检查连接是否处于活动状态：

   ```console
   $ docker offload status
   ```

3. 要获取更多信息，请运行以下命令：

   ```console
   $ docker offload diagnose
   ```

4. 如果未连接，请启动新会话：

   ```console
   $ docker offload start
   ```

5. 使用 `docker login` 验证身份验证。

6. 如有需要，您可以先登出然后再次登录：

   ```console
   $ docker logout
   $ docker login
   ```

7. 验证您的使用情况和计费信息。有关更多信息，请参阅 [Docker Offload 使用指南](/offload/usage/)。