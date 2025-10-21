---
title: Docker Offload 使用与计费
linktitle: 使用与计费
weight: 30
description: 了解 Docker Offload 的使用情况以及如何监控您的云资源。
keywords: 云, 使用, 云分钟, 共享缓存, 热门仓库, 云构建器, Docker Offload
---

{{< summary-bar feature_name="Docker Offload" >}}

> [!NOTE]
>
> Docker Offload Beta 版本授予的所有免费试用额度将在授予后 90 天内过期。使用额度过期后，如需继续使用 Docker Offload，您可以在 [Docker Home 计费](https://app.docker.com/billing)页面启用按需使用。

## Docker Offload 计费

对于 Docker Offload，您可以在 [Docker Home 计费](https://app.docker.com/billing)页面的 **Docker Offload** 部分查看和配置计费信息。在此页面，您可以：

- 查看您的包含使用量
- 查看云资源的费率
- 启用或禁用按需使用
- 添加或更改付款方式

有关计费的更多常规信息，请参阅 [计费](../billing/_index.md)。

## Docker Offload 概览

Docker Home 中的 Docker Offload 概览页面提供了关于您或团队如何使用云资源来构建和运行容器的可见性。

要查看 **概览** 页面：

1. 登录 [Docker Home](https://app.docker.com/)。
2. 选择要管理 Docker Offload 的账户。
3. 选择 **Offload** > **概览**。

以下部分描述了 **概览** 页面上的可用小部件。

### Offload 分钟数

此小部件显示随时间推移使用的 Offload 分钟总数。Offload 分钟数表示在 Offload 环境中运行构建和容器所花费的时间。您可以使用此图表来：

- 跟踪您的 Offload 使用趋势。
- 发现使用量激增，这可能表示 CI 变更或构建问题。
- 估算相对于订阅限制的使用量。

### 构建缓存使用情况

此小部件显示所有构建中缓存重用的数据，帮助您了解 Docker Offload 使用构建缓存的效果。它提供了以下洞察：

- 缓存命中与未命中的百分比。
- 通过重用缓存层节省的预估构建时间。
- 通过调整 Dockerfile 或构建策略来提高缓存效率的机会。

### 热门构建仓库

此小部件突出显示 Docker Offload 中构建活动最频繁的仓库。此小部件帮助您了解哪些项目消耗最多的云资源以及它们的构建效率。

它包括聚合指标和每个仓库的详细信息，为您提供全面的视图。

使用此小部件可以：

- 识别构建热点：查看哪些仓库消耗最多的构建时间和资源。
- 发现趋势：监控跨项目的构建活动演变。
- 评估效率：检查哪些仓库从缓存重用中受益最多。
- 针对性改进：标记缓存命中率低或失败率高的仓库以进行优化。

### 前 10 个镜像

此小部件显示 Docker Offload 运行会话中使用最多的前 10 个镜像。它提供了关于哪些镜像使用最频繁的洞察，帮助您了解团队的容器使用模式。
