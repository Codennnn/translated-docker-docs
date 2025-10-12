---
description: 了解 Docker Compose 在容器化应用开发与部署中的优势与典型用例
keywords: docker compose, compose use cases, compose benefits, container orchestration, development environments, testing containers, yaml file
title: 为什么选择 Compose？
weight: 20
aliases: 
- /compose/features-uses/
---

## Docker Compose 的关键优势

使用 Docker Compose 可在开发、部署与管理容器化应用方面带来多重提升：

- 简化控制：通过一个 YAML 文件定义与管理多容器应用，简化编排与复用。

- 高效协作：可共享的 YAML 文件促进开发与运维协作，优化流程与问题处理，整体效率更高。

- 快速开发：Compose 会缓存用于创建容器的配置。重启未发生变化的服务时，Compose 会复用已有容器，从而能快速迭代环境变更。

- 跨环境可移植：Compose 文件支持变量，可按环境或用户自定义组合配置。

## Docker Compose 的常见用例

Compose 的使用场景很多，常见示例如下：

### 开发环境

在软件开发过程中，将应用运行在隔离环境并与之交互至关重要。可使用 Compose 命令行工具创建该环境并进行交互。

[Compose 文件](/reference/compose-file/_index.md)可用于文档化与配置应用的所有服务依赖（数据库、队列、缓存、Web 服务 API 等）。借助 Compose 命令行工具，只需一条命令（`docker compose up`）即可为每个依赖创建并启动一个或多个容器。

这些特性共同提供了一种便捷的项目起步方式：Compose 能把冗长的“开发者上手指南”简化为一份可读的 Compose 文件与少量命令。

### 自动化测试环境

自动化测试是任何持续部署（CD）或持续集成（CI）流程中的关键环节。端到端自动化测试需要一个可供运行的隔离环境。Compose 能便捷地为测试套件创建与销毁隔离的测试环境。将完整环境定义在[Compose 文件](/reference/compose-file/_index.md)中后，只需几条命令即可创建与销毁：

```console
$ docker compose up -d
$ ./run_tests
$ docker compose down
```

### 单机部署

Compose 传统上侧重于开发与测试工作流，但在新版本中持续增强了面向生产的能力。

关于生产环境特性的使用，参见《[生产环境中的 Compose](/manuals/compose/how-tos/production.md)》。

## 下一步

- [了解 Compose 的发展历程](history.md)
- [理解 Compose 的工作原理](compose-application-model.md)
- [试试快速开始](../gettingstarted.md)
