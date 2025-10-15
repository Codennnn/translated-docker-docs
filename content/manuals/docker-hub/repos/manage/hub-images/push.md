---
description: 了解如何向 Docker Hub 的存储库添加内容。
keywords: Docker Hub, Hub, 存储库内容, 推送
title: 推送镜像到存储库
linkTitle: 推送镜像
weight: 30
---

要向 Docker Hub 的存储库添加内容，你需要先为本地 Docker 镜像打标签，然后将其推送到目标存储库。
这样便于与你的团队或社区分享镜像，或在不同环境中复用。

1. 为 Docker 镜像打标签

   使用 `docker tag` 命令为镜像指定标签，标签通常包含 Docker Hub 命名空间与存储库名。通用语法：

   ```console
   $ docker tag [SOURCE_IMAGE[:TAG]] [NAMESPACE/REPOSITORY[:TAG]]
   ```

   示例：

   如果你的本地镜像名为 `my-app`，希望将其打上 `v1.0` 标签并推送到 `my-namespace/my-repo`，运行：

   ```console
   $ docker tag my-app my-namespace/my-repo:v1.0
   ```

2. 推送镜像到 Docker Hub

   使用 `docker push` 命令将已打好标签的镜像上传到 Docker Hub 指定存储库。

   示例：

   ```console
   $ docker push my-namespace/my-repo:v1.0
   ```

   上述命令会将 `v1.0` 标签的镜像推送到 `my-namespace/my-repo` 存储库。

3. 在 Docker Hub 上验证镜像
