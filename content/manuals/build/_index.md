---
title: Docker Build
weight: 20
description: 全面了解 Docker Build，帮助你打包与封装代码，并可在任意环境交付与运行
keywords: build, buildx, buildkit
params:
  sidebar:
    group: Open source
grid:
- title: 打包你的软件
  description: '将应用构建并打包，实现本地或云端随处运行。'
  icon: inventory_2
  link: /build/concepts/overview/
- title: 多阶段构建
  description: 通过最小依赖保持镜像小而安全。
  icon: stairs
  link: /build/building/multi-stage/
- title: 多平台镜像
  description: 在不同计算架构上无缝构建、推送、拉取并运行镜像。
  icon: content_copy
  link: /build/building/multi-platform/
- title: BuildKit
  description: 了解开源构建引擎 BuildKit。
  icon: construction
  link: /build/buildkit/
- title: 构建驱动
  description: 配置构建运行的位置与方式。
  icon: engineering
  link: /build/builders/drivers/
- title: 导出器
  description: 不仅能导出 Docker 镜像，还可导出任意构建产物。
  icon: output
  link: /build/exporters/
- title: 构建缓存
  description: 避免重复执行代价高的操作，如安装依赖。
  icon: cycle
  link: /build/cache/
- title: Bake
  description: 使用 Bake 编排你的构建。
  icon: cake
  link: /build/bake/
aliases:
- /buildx/working-with-buildx/
- /develop/develop-images/build_enhancements/
---

Docker Build 是 Docker Engine 使用最广泛的功能之一。只要你在创建镜像，就在使用 Docker Build。它是软件开发生命周期中的关键环节，用于打包与封装代码，使其能够在任何地方交付与运行。

但 Docker Build 远不止是一条用于构建镜像的命令，也不仅仅是打包代码。它是一整套工具与特性的生态，既覆盖常见工作流任务，也支持更复杂的高级场景。

{{< grid >}}
