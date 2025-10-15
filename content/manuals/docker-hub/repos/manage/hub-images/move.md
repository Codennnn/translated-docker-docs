---
description: 了解如何在不同存储库之间迁移镜像。
keywords: Docker Hub, Hub, 存储库内容, 迁移
title: 在存储库之间迁移镜像
linkTitle: 迁移镜像
weight: 40
---

将镜像在多个存储库间进行整合与组织，有助于简化工作流，无论是个人项目还是组织协作。
本主题介绍如何在 Docker Hub 存储库之间迁移镜像，确保内容在正确的账号或命名空间下保持可访问与有序。

## 个人到个人（Personal to personal）

在整合个人存储库时，你可以先从原存储库拉取私有镜像，再将其推送到你拥有的另一个存储库。为避免丢失私有镜像，请执行以下步骤：

1. 使用个人订阅[注册](https://app.docker.com/signup)一个新的 Docker 账号。
2. 使用原有 Docker 账号登录 [Docker](https://app.docker.com/login)。
3. 拉取你的镜像：

   ```console
   $ docker pull namespace1/docker101tutorial
   ```

4. 使用新创建的 Docker 用户名为你的私有镜像打标签，例如：

   ```console
   $ docker tag namespace1/docker101tutorial new_namespace/docker101tutorial
   ```
5. 在命令行使用 `docker login`，以新账号登录，并将新打标签的私有镜像推送到新账号的命名空间：

   ```console
   $ docker push new_namespace/docker101tutorial
   ```

此前账号中的私有镜像现已在新账号中可用。

## 个人到组织（Personal to an organization）

为避免丢失私有镜像，你可以先从个人账号拉取私有镜像，再推送到你拥有的组织。

1. 访问 [Docker Hub](https://hub.docker.com) 并选择 **My Hub**。
2. 选择目标组织，并确认你的用户账号已加入该组织。
3. 使用原有 Docker 账号登录 [Docker Hub](https://hub.docker.com)，并拉取你的镜像：

   ```console
   $ docker pull namespace1/docker101tutorial
   ```
4. 使用新组织的命名空间为镜像打标签：

   ```console
   $ docker tag namespace1/docker101tutorial <new_org>/docker101tutorial
   ```
5. 将新打标签的镜像推送到新组织的命名空间：

   ```console
   $ docker push new_org/docker101tutorial
   ```

原先用户账号中的私有镜像现已可供你的组织使用。