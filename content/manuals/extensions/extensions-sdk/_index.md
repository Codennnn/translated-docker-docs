---
title: 扩展 SDK 概览
linkTitle: 扩展 SDK
description: Docker 扩展 SDK 文档总览索引
keywords: Docker, Extensions, sdk
aliases:
 - /desktop/extensions-sdk/dev/overview/
 - /desktop/extensions-sdk/
grid:
  - title: "构建与发布流程"
    description: 了解扩展的构建与发布流程。
    icon: "checklist"
    link: "/extensions/extensions-sdk/process/"
  - title: "快速开始"
    description: 按照快速开始指南，快速创建一个基础的 Docker 扩展。
    icon: "explore"
    link: "/extensions/extensions-sdk/quickstart/"
  - title: "设计规范"
    description: 确保你的扩展遵循 Docker 的设计规范与设计原则。
    icon: "design_services"
    link: "/extensions/extensions-sdk/design/design-guidelines/"
  - title: "发布你的扩展"
    description: 了解如何将扩展发布到扩展市场（Marketplace）。
    icon: "publish"
    link: "/extensions/extensions-sdk/extensions/"
  - title: "与 Kubernetes 交互"
    description: 了解如何在扩展中间接与 Kubernetes 集群交互。
    icon: "multiple_stop"
    link: "/extensions/extensions-sdk/guides/kubernetes/"
  - title: "多架构扩展"
    description: 将你的扩展构建为多架构镜像。
    icon: "content_copy"
    link: "/extensions/extensions-sdk/extensions/multi-arch/"
---

本节资源可帮助你创建自己的 Docker 扩展。

Docker CLI 提供了一组命令，帮助你将扩展以特定格式的 Docker 镜像进行打包、构建并发布。

镜像文件系统的根目录包含一个 `metadata.json` 文件，用于描述扩展的内容；它是 Docker 扩展的核心组成部分。

扩展可以包含前端 UI，以及在宿主机或 Desktop 虚拟机中运行的后端组件。更多信息请参阅 [Architecture](architecture/_index.md)。

扩展通常通过 Docker Hub 分发。但你也可以在本地进行开发，而无需将扩展推送到 Docker Hub。详见 [Extensions distribution](extensions/DISTRIBUTION.md)。

{{% include "extensions-form.md" %}}

{{< grid >}}
