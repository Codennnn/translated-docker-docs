---
title: 关于 Docker Offload
linktitle: 关于
weight: 15
description: 了解 Docker Offload 的功能、特性及其工作原理。
keywords: 云端, 构建, 远程构建器
---

Docker Offload 是一项完全托管的服务，允许您使用熟悉的 Docker 工具（包括 Docker Desktop、Docker CLI 和 Docker Compose）在云端构建和运行容器。它将您的本地开发工作流扩展到可扩展的云端环境，使您能够将计算密集型任务转移到云端，加速构建过程，并在整个软件生命周期中安全地管理容器工作负载。

Docker Offload 还支持 GPU 加速实例，让您能够容器化并运行计算密集型工作负载，如 Docker Model Runner 以及其他受益于 GPU 加速的机器学习或数据处理任务。

## 主要功能

Docker Offload 包含以下功能，以支持现代容器工作流：

- **云端构建**：在远程完全托管的 BuildKit 实例上执行构建
- **GPU 加速**：使用支持 NVIDIA L4 GPU 的环境进行机器学习、媒体处理和其他计算密集型工作负载
- **临时云运行器**：为每个容器会话自动配置和拆除云环境
- **共享构建缓存**：通过智能共享缓存层加速多台机器和团队成员之间的构建速度
- **混合工作流**：使用 Docker Desktop 或 CLI 在本地和远程执行之间无缝切换
- **安全通信**：在 Docker Desktop 和云环境之间使用加密隧道，支持安全密钥和镜像拉取
- **端口转发和绑定挂载**：即使在云端运行容器，也能保持本地开发体验
- **VDI 友好**：在虚拟桌面环境或不支持嵌套虚拟化的系统中使用 Docker Offload

## 为何使用 Docker Offload？

Docker Offload 专为在本地和云环境中工作的现代开发团队而设计。它可以帮助您：

- 将繁重的构建和运行任务转移到快速、可扩展的基础设施上
- 加速开发和测试过程中的反馈循环
- 运行需要比本地环境更多资源的容器
- 构建和运行 AI 应用程序，即时访问支持 GPU 的环境
- 使用 Docker Compose 管理需要云资源的复杂多服务应用程序
- 保持环境一致性，无需管理自定义基础设施
- 在受限或低功耗环境（如 VDI）中高效开发

Docker Offload 是高速开发工作流的理想选择，既能提供云端的灵活性，又不牺牲本地工具的简洁性。

## Docker Offload 的工作原理

Docker Offload 通过将 Docker Desktop 连接到安全的专用云资源，替代了在本地构建或运行容器的需求。

### 使用 Docker Offload 进行构建

当您使用 Docker Offload 进行构建时，`docker buildx build` 命令会将构建请求发送到云端的远程 BuildKit 实例，而不是在本地执行。您的工作流程保持不变，只是执行环境发生了变化。

构建在由 Docker 配置和管理的基础设施上运行：

- 每个云构建器都是一个独立的 Amazon EC2 实例，拥有自己的 EBS 卷
- 远程构建器使用共享缓存加速多台机器和团队成员之间的构建
- 构建结果在传输过程中加密，并发送到您指定的目标（如仓库或本地镜像存储）

Docker Offload 自动管理构建器的生命周期，无需配置或维护基础设施。

> [!NOTE]
>
> Docker Offload 构建器目前托管在美国东部区域。其他地区的用户可能会遇到较高的延迟。

### 使用 Docker Offload 运行容器

当您使用 Docker Offload 运行容器时，Docker Desktop 会创建一个安全 SSH 隧道，连接到在云端运行的 Docker 守护进程。您的容器完全在该远程环境中启动和管理。

具体过程如下：

1. Docker Desktop 连接到云端并触发容器创建。
2. Docker Offload 拉取所需镜像并在云端启动容器。
3. 容器运行期间，连接保持打开状态。
4. 当容器停止运行时，环境会自动关闭并清理。

这种设置避免了在本地运行容器的开销，即使在低功耗机器（包括不支持嵌套虚拟化的机器）上也能实现快速、可靠的容器运行。这使得 Docker Offload 成为使用虚拟桌面、云端托管开发机或旧硬件的开发者的理想选择。

Docker Offload 还支持 GPU 加速工作负载。需要 GPU 访问的容器可以在配置了 NVIDIA L4 GPU 的云实例上运行，以实现高效的 AI 推理、媒体处理和通用 GPU 加速。这使得模型评估、图像处理和硬件加速 CI 测试等计算密集型工作流能够在云端无缝运行。

尽管在远程运行，绑定挂载和端口转发等功能仍然可以无缝工作，从 Docker Desktop 和 CLI 中提供类似本地的体验。

Docker Offload 为每个会话配置一个临时云环境。当您与 Docker Desktop 交互或主动使用容器时，环境保持活跃状态。如果检测到约 5 分钟内无活动，会话将自动关闭。这包括该环境中的任何容器、镜像或卷，它们会在会话结束时被删除。

## 下一步

按照 [Docker Offload 快速入门指南](/offload/quickstart/) 亲身体验 Docker Offload。