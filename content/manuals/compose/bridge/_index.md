---
description: 了解 Compose Bridge 如何将 Docker Compose 文件转换为 Kubernetes 清单，实现平台间的无缝迁移
keywords: docker compose bridge, compose to kubernetes, docker compose kubernetes integration, docker compose kustomize, compose bridge docker desktop
title: Compose Bridge 概览
linkTitle: Compose Bridge
weight: 50
---

{{< summary-bar feature_name="Compose bridge" >}}

Compose Bridge 会将你的 Docker Compose 配置转换为特定平台的格式——主要是 Kubernetes 清单。默认转换会生成 Kubernetes 清单以及一个 Kustomize overlay，适用于在启用 Kubernetes 的 Docker Desktop 上进行部署。  

这是一款灵活的工具，你可以直接使用[默认转换](usage.md)，也可以[自定义转换](customize.md)以满足特定项目的需求与约束。  

Compose Bridge 大幅简化了从 Docker Compose 迁移到 Kubernetes 的过程，帮助你在保持 Docker Compose 简洁与高效的同时，更轻松地发挥 Kubernetes 的能力。

## 工作原理

Compose Bridge 使用“转换”（transformation）机制，将 Compose 模型转换为其他形式。 

每个转换以 Docker 镜像的形式交付：它会把完全解析后的 Compose 模型作为 `/in/compose.yaml` 接收，并在 `/out` 目录下生成任意目标格式的文件。

Compose Bridge 提供了基于 Go 模板的 Kubernetes 转换，你可以通过替换或追加模板，轻松进行扩展与定制。

关于这些转换的工作方式，以及如何为你的项目进行定制，请参见《[自定义](customize.md)》。

## 下一步

- [使用 Compose Bridge](usage.md)
- [了解如何自定义 Compose Bridge](customize.md)