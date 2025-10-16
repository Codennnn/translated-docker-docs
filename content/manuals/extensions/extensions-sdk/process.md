---
description: 了解创建扩展的端到端流程。
title: 构建与发布流程
keyword: Docker Extensions, sdk, build, create, publish
aliases:
 - /desktop/extensions-sdk/process/
weight: 10
---

本文档按照你创建扩展时的实际步骤进行编排。

创建 Docker 扩展主要分为两部分：

1. 构建基础
2. 发布扩展

> [!NOTE]
>
> 创建 Docker 扩展无需付费。[Docker Extension SDK](https://www.npmjs.com/package/@docker/extension-api-client) 采用 Apache 2.0 许可，免费可用。任何人都可以自由创建并分享扩展。
> 
> 对于扩展自身采用何种许可证，也没有强制要求；你可以在创建扩展时自行决定。

## 第一部分：构建基础

构建过程包括：

- 安装最新版 Docker Desktop。
- 准备项目目录与文件，包括扩展源码及所需的扩展专用文件。
- 编写用于在 Docker Desktop 中构建、发布与运行扩展的 `Dockerfile`。
- 配置元数据文件（需置于镜像文件系统根目录）。
- 构建并安装扩展。

如需参考实现，可查看 [samples 目录](https://github.com/docker/extensions-sdk/tree/main/samples) 中的示例。

> [!TIP]
>
> 在创建扩展时，请遵循 [设计规范](design/design-guidelines.md) 与 [UI 样式指南](design/_index.md)，以确保视觉一致性，并满足 [AA 级无障碍标准](https://www.w3.org/WAI/WCAG2AA-Conformance)。

## 第二部分：发布与分发扩展

Docker Desktop 会在扩展市场（Extensions Marketplace）展示已发布的扩展。扩展市场是经过策划的集合，开发者可以在此发现提升开发体验的扩展，并将自己的扩展上传分享给所有人。

如果你希望将扩展发布到市场，请阅读 [发布文档](extensions/publish.md)。

{{% include "extensions-form.md" %}}

## 下一步

如果你想快速上手创建 Docker 扩展，请查看 [快速开始](quickstart.md)。

或者，从“第一部分：构建基础”开始，深入了解扩展创建流程中每一步的细节。

若需完整的构建流程演示，推荐观看以下 DockerCon 2022 的视频讲解。

<iframe width="560" height="315" src="https://www.youtube.com/embed/Yv7OG-EGJsg" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
