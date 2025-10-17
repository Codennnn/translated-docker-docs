---
description: 全面了解 Docker 平台：它的用途、架构与底层技术。
keywords: what is a docker, docker daemon, why use docker, docker architecture, what
  to use docker for, docker client, what is docker for, why docker, uses for docker,
  what is docker container used for, what are docker containers used for
title: 什么是 Docker？
weight: 20
aliases:
 - /introduction/understanding-docker/
 - /engine/userguide/basics/
 - /engine/introduction/understanding-docker/
 - /engine/understanding-docker/
 - /engine/docker-overview/
 - /get-started/overview/
 - /guides/docker-overview/
---

Docker 是一个用于开发、交付和运行应用的开放平台。它让你可以将应用和基础设施解耦，从而更快速地交付软件。使用 Docker，你可以像管理应用一样管理基础设施。借助 Docker 在打包、测试和部署代码方面的方法论，你可以大幅缩短从编写代码到部署生产的时间差。

## Docker 平台

Docker 提供了将应用打包并运行在一种松散隔离环境（容器）中的能力。隔离性和安全性允许你在同一主机上同时运行多个容器。容器是轻量的，并包含运行应用所需的一切，因此你无需依赖主机上已安装的内容。你可以在协作中共享容器，并确保所有人拿到的都是一致、可运行的容器。

Docker 还提供了用于管理容器全生命周期的工具与平台：

* 在容器中开发你的应用及其依赖组件。
* 让容器成为应用分发和测试的基本单元。
* 就绪后，将应用作为容器或编排服务部署到生产环境 —— 无论生产在本地数据中心、云服务商，或混合环境，迁移方式都一致。

## 我可以用 Docker 做什么？

### 更快且一致地交付应用

Docker 通过在本地容器中提供标准化环境来简化开发流程，
这非常适合持续集成与持续交付（CI/CD）工作流。

考虑如下场景：

- 开发者在本地编写代码，并通过 Docker 容器与同事共享工作成果。
- 他们使用 Docker 将应用推送到测试环境，并运行自动化与手动测试。
- 发现缺陷后，开发者可在开发环境修复并重新部署到测试环境进行验证。
- 测试完成后，只需将更新后的镜像推送到生产环境即可交付给客户。

### 弹性部署与伸缩

基于容器的平台使工作负载具有高度可移植性。
Docker 容器可以运行在开发者的笔记本、本地数据中心的物理/虚拟机、云提供商，
或这些环境的任意组合中。

Docker 的可移植性与轻量特性也便于动态管理工作负载，
可根据业务需要在近乎实时的粒度上扩缩应用与服务。

### 在同等硬件上运行更多工作负载

Docker 轻量且快速，是基于虚拟机的超管模式的高性价比替代方案。因此你可以更充分地利用服务器资源以实现业务目标。Docker 适用于高密度场景，以及需要以更少资源完成更多工作的中小规模部署。

## Docker 架构

Docker 采用客户端-服务端架构。Docker 客户端与 Docker 守护进程通信，由守护进程承担构建、运行和分发容器的繁重工作。客户端与守护进程可以运行在同一主机上，也可以将客户端连接到远程守护进程。二者通过 REST API 通信，基于 UNIX 套接字或网络接口。另一个 Docker 客户端是 Docker Compose，它用于管理由一组容器组成的应用。

![Docker 架构图](images/docker-architecture.webp)

### Docker 守护进程

Docker 守护进程（`dockerd`）监听 Docker API 请求，并管理镜像、容器、网络和卷等对象。守护进程还能与其他守护进程通信，以便管理 Docker 服务。

### Docker 客户端

Docker 客户端（`docker`）是许多用户与 Docker 交互的主要方式。当你使用 `docker run` 等命令时，客户端会将命令发送给 `dockerd` 执行。`docker` 命令使用 Docker API，且可同时与多个守护进程通信。

### Docker Desktop

Docker Desktop 是一款适用于 Mac、Windows 或 Linux 的易安装应用，可用于构建和分享容器化应用与微服务。Docker Desktop 包含 Docker 守护进程（`dockerd`）、Docker 客户端（`docker`）、Docker Compose、Docker Content Trust、Kubernetes 和 Credential Helper。更多信息参见 [Docker Desktop](/manuals/desktop/_index.md)。

### Docker 仓库（Registry）

Docker 仓库存放 Docker 镜像。Docker Hub 是任何人都可使用的公共仓库，Docker 默认会从 Docker Hub 拉取镜像。你也可以搭建私有仓库。

当你使用 `docker pull` 或 `docker run` 时，Docker 会从已配置的仓库拉取所需镜像；当你使用 `docker push` 时，Docker 会将你的镜像推送到已配置的仓库。

### Docker 对象

使用 Docker 时，你会创建和使用镜像、容器、网络、卷、插件等对象。本节简要概述其中的一些对象。

#### 镜像（Images）

镜像是只读模板，包含创建 Docker 容器的指令。镜像通常基于另一个镜像并做少量定制。例如，你可以构建一个基于 `ubuntu` 的镜像，并安装 Apache 和你的应用，以及让应用运行所需的配置。

你可以自己创建镜像，也可以直接使用他人发布到仓库的镜像。要构建自己的镜像，你需要编写 Dockerfile，使用简洁的语法定义创建和运行镜像所需的步骤。Dockerfile 中的每条指令都会在镜像中创建一层。当你修改 Dockerfile 并重建镜像时，仅会重建变更的那些层。这也是与其他虚拟化技术相比，镜像为何更轻量、小巧和快速的原因之一。

#### 容器（Containers）

容器是镜像的可运行实例。你可以使用 Docker API 或 CLI 创建、启动、停止、移动或删除容器；也可以将容器连接到一个或多个网络、为其挂载存储，甚至基于容器当前状态创建新镜像。

默认情况下，容器与其他容器及宿主机之间具有良好的隔离。你可以控制容器在网络、存储或其他底层子系统上的隔离程度。

容器由其镜像以及你在创建或启动时提供的配置选项共同定义。当容器被删除时，任何未持久化存储的状态更改都会消失。

##### `docker run` 示例

以下命令运行一个 `ubuntu` 容器，交互式地附着到本地终端会话，并执行 `/bin/bash`：

```console
$ docker run -i -t ubuntu /bin/bash
```

运行该命令时，会发生以下情况（假设你使用默认的仓库配置）：

1. 如果本地不存在 `ubuntu` 镜像，Docker 会从已配置的仓库拉取它（等同于手动运行 `docker pull ubuntu`）。
2. Docker 创建一个新容器（等同于手动运行 `docker container create`）。
3. Docker 为容器分配一个读写文件系统，作为其最上层。这使运行中的容器能够在本地文件系统中创建或修改文件和目录。
4. Docker 创建一个网络接口，将容器连接到默认网络（因为你未指定任何网络选项）。这包括为容器分配 IP 地址。默认情况下，容器可通过宿主机的网络连接访问外部网络。
5. Docker 启动容器并执行 `/bin/bash`。由于容器以交互方式运行并附着到你的终端（`-i` 和 `-t` 标志），你可以通过键盘输入，Docker 会将输出记录到你的终端。
6. 当你运行 `exit` 退出 `/bin/bash` 时，容器会停止，但不会被删除。你可以再次启动它或将其删除。

## 底层技术

Docker 使用 [Go 编程语言](https://golang.org/) 编写，并利用 Linux 内核的多项特性来实现其功能。Docker 使用名为 `namespaces` 的技术来提供称为容器的隔离工作空间。当你运行一个容器时，Docker 会为该容器创建一组命名空间。

这些命名空间提供了隔离层。容器的各个方面都在各自的命名空间中运行，其访问权限仅限于对应的命名空间。

## 进一步阅读

- [安装 Docker](/get-started/get-docker.md)
- [Docker 入门](/get-started/introduction/_index.md)
