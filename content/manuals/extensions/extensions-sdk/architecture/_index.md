---
title: 扩展架构
linkTitle: 架构
description: Docker 扩展架构
keywords: Docker, extensions, sdk, metadata
aliases: 
 - /desktop/extensions-sdk/architecture/
weight: 50
---

扩展是运行在 Docker Desktop 内部的应用程序。它们被打包为 Docker 镜像，通过 Docker Hub 分发，用户可以通过 Docker Desktop 仪表板中的市场或 Docker Extensions CLI 进行安装。

扩展可以由以下三个（可选）组件组成：
- 前端（或用户界面）：在 Docker Desktop 仪表板的选项卡中显示的 Web 应用程序
- 后端：在 Docker Desktop 虚拟机中运行的一个或多个容器化服务
- 可执行文件：在安装扩展时 Docker Desktop 复制到主机上的 Shell 脚本或二进制文件

![扩展的三个组件概览](images/extensions-architecture.png?w=600h=400)

扩展不一定需要包含所有这些组件，但至少需要其中一个，具体取决于扩展的功能。为了配置和运行这些组件，Docker Desktop 使用 `metadata.json` 文件。有关更多详细信息，请参阅 [元数据](metadata) 部分。

## 前端

前端本质上是一个由 HTML、JavaScript 和 CSS 构成的 Web 应用程序。它可以由一个简单的 HTML 文件和一些原生 JavaScript 构建，也可以使用任何前端框架，如 React 或 Vue.js。

当 Docker Desktop 安装扩展时，它会从扩展镜像中提取 UI 文件夹，这是由 `metadata.json` 中的 `ui` 部分定义的。有关更多详细信息，请参阅 [UI 元数据部分](metadata.md#ui-section)。

每次用户点击 **Extensions** 选项卡时，Docker Desktop 都会初始化扩展的 UI，就像第一次使用一样。当用户离开该选项卡时，UI 本身及其启动的所有子进程（如果有）都会被终止。

前端可以通过 [Extensions SDK](https://www.npmjs.com/package/@docker/extension-api-client) 调用 `docker` 命令、与扩展后端通信或调用部署在主机上的扩展可执行文件。

> [!TIP]
>
> `docker extension init` 命令会生成一个基于 React 的扩展。但你仍然可以将其作为自己扩展的起点，并使用任何其他前端框架，如 Vue、Angular、Svelte 等，甚至可以使用原生 JavaScript。

了解更多关于为扩展[构建前端](/manuals/extensions/extensions-sdk/build/frontend-extension-tutorial.md)的信息。

## 后端

除了前端应用程序，扩展还可以包含一个或多个后端服务。在大多数情况下，扩展不需要后端，功能可以通过 SDK 调用 docker 命令来实现。但是，在某些情况下，扩展需要后端服务，例如：
- 运行必须比前端生命周期更长的长时间运行进程
- 在本地数据库中存储数据并通过 REST API 提供服务
- 存储扩展状态，例如当按钮启动长时间运行的进程时，这样如果用户离开扩展后再次返回，前端可以从中断的地方继续
- 访问 Docker Desktop 虚拟机中的特定资源，例如通过在 compose 文件中挂载文件夹

> [!TIP]
>
> `docker extension init` 命令会生成一个 Go 后端。但你仍然可以将其作为自己扩展的起点，并使用任何其他语言，如 Node.js、Python、Java、.Net 或任何其他语言和框架。

通常，后端由在 Docker Desktop 虚拟机中运行的一个容器组成。在内部，Docker Desktop 创建一个 Docker Compose 项目，根据 `metadata.json` 的 `vm` 部分的 `image` 选项创建容器，并将其附加到 Compose 项目。有关更多详细信息，请参阅 [UI 元数据部分](metadata.md#vm-section)。

在某些情况下，可以使用 `compose.yaml` 文件代替 `image`。当后端容器需要更具体的选项（如挂载卷或请求[能力](https://docs.docker.com/engine/reference/run/#runtime-privilege-and-linux-capabilities)）而这些选项无法仅通过 Docker 镜像表达时，这非常有用。`compose.yaml` 文件还可以用于添加扩展所需的多个容器，如数据库或消息代理。请注意，如果 Compose 文件定义了多个服务，SDK 只能联系其中的第一个服务。

> [!NOTE]
>
> 在某些情况下，从后端与 Docker 引擎交互也很有用。
> 请参阅从后端[如何使用 Docker 套接字](../guides/use-docker-socket-from-backend.md)。

为了与后端通信，Extensions SDK 提供了[函数](../dev/api/backend.md#get)，用于从前端发出 `GET`、`POST`、`PUT`、`HEAD` 和 `DELETE` 请求。在底层，通信是通过套接字或命名管道完成的，具体取决于操作系统。如果后端监听端口，将很难防止与已在主机或容器中运行的其他应用程序发生冲突。此外，有些用户在受限环境中运行 Docker Desktop，他们无法在机器上打开端口。

![后端和前端通信](images/extensions-arch-2.png?w=500h=300)

最后，后端可以用任何技术构建，只要它可以在容器中运行并在套接字上监听。

了解更多关于为扩展[添加后端](/manuals/extensions/extensions-sdk/build/backend-extension-tutorial.md)的信息。

## 可执行文件

除了前端和后端，扩展还可以包含可执行文件。可执行文件是在安装扩展时安装在主机上的二进制文件或 Shell 脚本。前端可以通过 [扩展 SDK](../dev/api/backend.md#invoke-an-extension-binary-on-the-host) 调用它们。

当扩展需要与第三方 CLI 工具（如 AWS、`kubectl` 等）交互时，这些可执行文件非常有用。将这些可执行文件随扩展一起提供，可以确保 CLI 工具在用户机器上始终以正确的版本可用。

当 Docker Desktop 安装扩展时，它会按照 `metadata.json` 中的 `host` 部分定义将可执行文件复制到主机上。有关更多详细信息，请参阅 [UI 元数据部分](metadata.md#host-section)。

![可执行文件和前端通信](images/extensions-arch-3.png?w=250h=300)

但是，由于它们在用户的机器上执行，因此必须适用于它们运行的平台。例如，如果你想提供 `kubectl` 可执行文件，你需要为 Windows、Mac 和 Linux 提供不同的版本。多架构镜像也需要包含为正确架构（AMD / ARM）构建的二进制文件。

有关更多详细信息，请参阅 [host 元数据部分](metadata.md#host-section)。

了解如何[调用主机二进制文件](../guides/invoke-host-binaries.md)。
