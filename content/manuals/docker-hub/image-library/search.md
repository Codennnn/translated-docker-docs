---
description: 了解如何浏览与搜索 Docker Hub 的海量资源。
keywords: Docker Hub, Hub, 浏览, 搜索, 镜像库
title: Docker Hub 搜索
linkTitle: 搜索
weight: 10
---

[Docker Hub 搜索界面](https://hub.docker.com/search) 可帮助你探索数百万条资源。为便于精准定位所需内容，它提供了多种筛选条件，既可缩小搜索范围，也便于发现不同类型的内容。

## 筛选条件

搜索功能支持多类筛选条件来依据你的需求缩小结果范围，例如按产品、类别与可信内容等进行筛选，帮助你更快找到最适合项目的资源。

### 产品

Docker Hub 的内容库包含三类产品，分别面向开发者与组织的不同需求：镜像、插件与扩展。

#### 镜像

Docker Hub 托管了数百万个容器镜像，是容器化应用与方案的首选仓库。这些镜像包括：

- 操作系统镜像：如 Ubuntu、Debian、Alpine 等 Linux 发行版基础镜像，或 Windows Server 镜像。
- 数据库与存储镜像：如 MySQL、PostgreSQL、MongoDB 等预配置数据库，简化应用开发。
- 语言与框架镜像：如 Java、Python、Node.js、Ruby、.NET 等常用镜像，提供预构建环境以加快开发。

Docker Hub 上的镜像以预构建、可复用的“积木”形式简化开发流程，无需从零开始。无论你是初学者构建第一个容器，还是企业管理复杂架构，Docker Hub 镜像都能提供可靠的基石。

#### 插件

Docker Hub 的插件可用于扩展与定制 Docker Engine，以满足特定需求。插件与 Docker Engine 直接集成，提供如下能力：

- 网络插件：增强网络功能，便于对接复杂网络基础设施。
- 卷插件：提供高级存储选项，支持跨多种后端的持久化与分布式存储。
- 鉴权插件：提供细粒度的访问控制，强化 Docker 环境安全。

借助 Docker 插件，团队可以按需定制 Docker Engine 的运行能力，并确保与现有基础设施与工作流的兼容性。

了解插件更多信息，参见 [Docker Engine 托管插件系统](/manuals/engine/extend/_index.md)。

#### 扩展

Docker Hub 为 Docker Desktop 提供扩展，用于增强核心功能。这些扩展旨在简化软件开发生命周期，主要提供以下工具：

- 系统优化与监控：管理资源并优化 Docker Desktop 性能。
- 容器管理：简化容器的部署与监控。
- 数据库管理：在容器内高效进行数据库操作。
- Kubernetes 与云集成：打通本地环境与云原生/Kubernetes 工作流。
- 可视化工具：通过图形化视图洞察容器资源使用情况。

这些扩展通过减少上下文切换、将关键工具集成到 Docker Desktop 界面中，帮助开发者与团队构建更高效、统一的工作流。

了解扩展更多信息，参见 [Docker Extensions](/manuals/extensions/_index.md)。

### 可信内容

Docker Hub 的可信内容提供经过策展的高质量与安全镜像，旨在让开发者对所用资源的可靠性与安全性更有信心。这些镜像稳定、定期更新，并遵循行业最佳实践，是构建与部署应用的坚实基础。可信内容包括 Docker 官方镜像、Verified Publisher 镜像与 Docker 赞助的开源软件镜像。

更多详情参见 [Trusted content](./trusted-content.md)。

### 类别

Docker Hub 通过“类别”便于查找与探索容器镜像。类别会按主要使用场景对镜像分组，帮助你快速定位构建、部署与运行应用所需的工具与资源。

{{% include "hub-categories.md" %}}

### 操作系统

**Operating systems**（操作系统）筛选器可将搜索范围限定为与特定宿主操作系统兼容的容器镜像。无论你的目标环境是 Linux、Windows，还是同时兼容两者，都可借此确保所用镜像与之匹配。

- **Linux**：获取适配 Linux 环境的大量镜像，这些镜像提供构建与运行 Linux 应用所需的基础环境。
- **Windows**：浏览 Windows 容器镜像。

> [!NOTE]
>
> The **Operating systems** filter is only available for images. If you select
> the **Extensions** or **Plugins** filter, then the **Operating systems**
> filter isn't available.

### 架构

**Architectures**（架构）筛选器可让你查找支持特定 CPU 架构的镜像，确保从开发设备到生产服务器的硬件兼容性。

- **ARM**：选择兼容 ARM 处理器的镜像，常见于物联网与嵌入式设备。
- **ARM 64**：查找适配现代 ARM 处理器的 64 位镜像，例如 AWS Graviton 或 Apple Silicon。
- **IBM POWER**：查找为 IBM Power Systems 优化的镜像，适用于企业级工作负载。
- **PowerPC 64 LE**：获取适配小端序 PowerPC 64 位架构的镜像。
- **IBM Z**：查找适配 IBM Z 大型机的镜像，确保企业级硬件兼容性。
- **x86**：选择兼容 32 位 x86 架构的镜像，适合旧设备或轻量场景。
- **x86-64**：筛选适配现代 64 位 x86 系统的镜像，广泛用于桌面、服务器与云基础设施。

> [!NOTE]
>
> The **Architectures** filter is only available for images. If you select the
> **Extensions** or **Plugins** filter, then the **Architectures** filter isn't
> available.

### Docker 已审查

**Reviewed by Docker**（Docker 已审查）筛选器在选择扩展时提供额外保障。你可以据此确认某个 Docker Desktop 扩展是否经过 Docker 的质量与可靠性审查。

- **Reviewed**：已通过 Docker 审查流程、符合高标准的扩展。
- **Not Reviewed**：尚未经过 Docker 审查的扩展。

> [!NOTE]
>
> **Reviewed by Docker** 筛选器仅适用于扩展。要显示该筛选器，需要在 **Products** 中只选择 **Extensions**。