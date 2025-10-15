---
description: 了解如何在 Docker Hub 存储库中管理镜像
keywords: Docker Hub, Hub, 镜像, 内容
title: 镜像管理
linkTitle: 镜像
weight: 60
---

Docker Hub 提供了一系列强大的功能，帮助你管理与组织存储库内容，确保镜像与工件可访问、受版本控制且便于分享。
本节涵盖镜像管理的关键任务，包括打标签、推送镜像、在存储库之间迁移镜像，以及对受支持的软件工件的管理。


- [标签（Tags）](./tags.md)：在同一存储库内，标签用于为镜像的不同迭代进行版本化与组织管理。
  本主题介绍标签的概念，并指导你在 Docker Hub 中创建、查看与删除标签。
- [镜像管理](./manage.md)：管理你的镜像与镜像索引，以优化存储库空间占用。
- [软件工件](./oci-artifacts.md)：Docker Hub 支持 OCI（Open Container Initiative）工件，
  允许你存储、管理与分发超出标准 Docker 镜像范围的多类内容，如 Helm Chart、
  漏洞报告等。本节提供 OCI 工件概览及推送到 Docker Hub 的示例。
- [推送镜像到 Hub](./push.md)：将本地镜像推送到 Docker Hub，使其可供你的团队或 Docker 社区使用。
  学习如何配置镜像并使用 `docker push` 命令上传。
- [在存储库之间迁移镜像](./move.md)：在不同存储库之间组织内容，有助于简化协作与资源管理。
  本主题详述如何将镜像从一个 Docker Hub 存储库迁移到另一个，无论是个人整合还是与组织共享。