---
title: Docker Compose
weight: 30
description: 通过本指南学习如何使用 Docker Compose 定义并运行多容器应用，
  全面掌握该工具的核心用法。
keywords: docker compose, docker-compose, compose.yaml, docker compose command, multi-container applications, container orchestration, docker cli
params:
  sidebar:
    group: Open source
grid:
- title: 为什么选择 Compose？
  description: 了解 Docker Compose 的核心优势
  icon: feature_search
  link: /compose/intro/features-uses/
- title: Compose 的工作原理 
  description: 理解 Compose 的工作方式
  icon: category
  link: /compose/intro/compose-application-model/
- title: 安装 Compose
  description: 按照指南安装 Docker Compose。
  icon: download
  link: /compose/install
- title: 快速开始
  description: 在构建一个简单的 Python Web 应用的过程中，
    掌握 Docker Compose 的关键概念。
  icon: explore
  link: /compose/gettingstarted
- title: 查看发行说明
  description: 了解最新的改进与缺陷修复。
  icon: note_add
  link: /compose/release-notes
- title: 查阅 Compose 文件参考
  description: 了解如何为 Docker 应用定义服务、网络和卷，
    以便更好地进行配置。
  icon: polyline
  link: /reference/compose-file
- title: 使用 Compose Bridge
  description: 将你的 Compose 配置文件转换为适配不同平台的配置，例如 Kubernetes。
  icon: move_down
  link: /compose/bridge
- title: 常见问题
  description: 查看常见问题并了解如何反馈。
  icon: help
  link: /compose/faq
- title: 迁移到 Compose v2
  description: 了解如何从 Compose v1 迁移到 v2
  icon: folder_delete
  link: /compose/releases/migrate/
aliases:
- /compose/cli-command/
- /compose/networking/swarm/
- /compose/overview/
- /compose/swarm/
- /compose/completion/
---

Docker Compose 是用于定义并运行多容器应用的工具，它能帮助你获得更简洁高效的开发与部署体验。 

Compose 简化了整个应用栈的管理，你可以在一个 YAML 配置文件中轻松管理服务、网络和卷。随后，只需一条命令，就能根据该配置创建并启动所有服务。

Compose 适用于所有环境：生产、预发布、开发、测试，以及 CI 流程。它还提供了一组命令，用于管理应用的完整生命周期：

 - 启动、停止和重新构建服务
 - 查看正在运行的服务状态
 - 实时查看运行中服务的日志输出
 - 在某个服务上执行一次性命令

{{< grid >}}
