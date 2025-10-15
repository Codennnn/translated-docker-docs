---
description: 了解自动构建的工作方式
keywords: Docker Hub, 自动构建
title: 自动构建
weight: 90
aliases:
- /docker-hub/builds/how-builds-work/
---

{{< summary-bar feature_name="Automated builds" >}}

Docker Hub 可以从外部源代码仓库自动构建镜像，并将构建好的镜像自动推送到你的 Docker 存储库。

![An automated build dashboard](images/index-dashboard.png)

当你配置自动构建（autobuild）时，需要列出要构建为 Docker 镜像的分支和标签。每当你向这些分支推送代码（例如在 GitHub 上），或更新对应的镜像标签时，推送会通过 webhook 触发一次新的构建以生成镜像，随后将镜像推送到 Docker Hub。

> [!NOTE]
>
> 即使已启用自动构建，你仍可以使用 `docker push` 将预构建的镜像推送到该存储库。

如果配置了自动测试，测试会在构建之后、推送到仓库之前运行。你可以借此形成持续集成流程：测试未通过的构建不会推送镜像。自动测试不会自行将镜像推送到仓库。参见[了解自动化镜像测试](automated-testing.md)。

根据你的[订阅计划](https://www.docker.com/pricing)，你可能拥有并发构建能力，即同一时间可以运行 `N` 个自动构建（`N` 取决于订阅）。当达到 `N+1` 时，后续构建将进入队列等待执行。

队列最多可保留 30 个待处理构建，超过部分会被丢弃。并发上限：Pro 为 5；Team 与 Business 为 15。自动构建支持的镜像大小上限为 10 GB。
