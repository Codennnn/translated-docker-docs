---
title: Docker 实践教程概览
linkTitle: Docker 实践教程
keywords: Docker 基础，如何启动 Docker 容器，容器设置，环境搭建，
  Docker，如何安装与配置 Docker，Docker 容器指南，Docker 入门
description: 通过本实践教程快速上手 Docker 基础。你将
  学习容器、镜像，并将你的第一个应用容器化。
aliases:
- /guides/get-started/
- /get-started/hands-on-overview/
- /guides/workshop/
---

本实践教程约 45 分钟，提供逐步操作指引，帮助你开始使用 Docker。你将学到：

- 构建镜像并以容器形式运行。
- 通过 Docker Hub 分享镜像。
- 使用包含数据库的多个容器部署 Docker 应用。
- 使用 Docker Compose 运行应用。

> [!NOTE]
>
> 若需快速了解 Docker 以及容器化能带来的好处，请参阅 [快速开始](/get-started/introduction/_index.md)。

## 什么是容器？

容器是在宿主机上运行的一个沙盒化进程，与该宿主机上其他进程相互隔离。这种隔离基于 Linux 长期提供的 [内核命名空间与 cgroups](https://medium.com/@saschagrunert/demystifying-containers-part-i-kernel-space-2c53d6979504) 功能。Docker 让这些能力更易上手。总结来说，容器：

- 是镜像的可运行实例。你可以通过 Docker API 或 CLI 创建、启动、停止、移动或删除容器。
- 可以运行在本地机器、虚拟机，或部署到云端。
- 具有可移植性（可运行在任意操作系统上）。
- 与其他容器相互隔离，并运行其自身的软件、二进制、配置等。

如果你熟悉 `chroot`，可以把容器理解为 `chroot` 的扩展：文件系统来自镜像。但容器还提供了 `chroot` 所不具备的额外隔离能力。

## 什么是镜像？

运行中的容器使用一个隔离的文件系统。该文件系统由镜像提供；镜像必须包含运行应用所需的一切——依赖、配置、脚本、二进制等。镜像还包含容器的其他配置，例如环境变量、默认运行命令以及其他元数据。

## 下一步

在本节中，你已了解容器与镜像。

接下来，你将把一个简单应用容器化，亲手实践这些概念。

{{< button text="容器化一个应用" url="02_our_app.md" >}}
