---
title: Testcontainers
weight: 40
description: 了解如何使用 Testcontainers 以你熟悉的编程语言以编程方式运行容器。
keywords: docker APIs, docker, testcontainers documentation, testcontainers, testcontainers oss, testcontainers oss documentation,
  docker compose, docker-compose, java, golang, go
params:
  sidebar:
    group: Open source
intro:
- title: What is Testcontainers?
  description: 了解 Testcontainers 的作用及其核心优势
  icon: feature_search
  link: https://testcontainers.com/getting-started/#what-is-testcontainers
- title: The Testcontainers workflow
  description: 理解 Testcontainers 的工作流程
  icon: explore
  link: https://testcontainers.com/getting-started/#testcontainers-workflow
quickstart:
- title: Testcontainers for Go
  description: 一个 Go 包，简化在自动化集成/冒烟测试中创建与清理基于容器的依赖。
  icon: /icons/go.svg
  link: https://golang.testcontainers.org/quickstart/
- title: Testcontainers for Java
  description: 一个支持 JUnit 测试的 Java 库，为任何可在 Docker 容器中运行的组件提供轻量、一次性实例。
  icon: /icons/java.svg
  link: https://java.testcontainers.org/
---

Testcontainers 是一组开源库，提供简洁而轻量的 API，用真实的服务（封装在 Docker 容器中）来快速启动本地开发与测试所需的依赖。使用 Testcontainers，你可以在不使用 mock 或内存服务的情况下，编写依赖与生产环境相同服务的测试。

{{< grid items=intro >}}

## 快速开始

### 支持的语言

Testcontainers 支持最常用的编程语言；其中以下实现由 Docker 赞助开发：

- [Go](https://golang.testcontainers.org/quickstart/)
- [Java](https://java.testcontainers.org/quickstart/junit_5_quickstart/)

其余实现由社区驱动，并由独立贡献者维护。

### 先决条件

Testcontainers 需要与 Docker API 兼容的容器运行时。在开发过程中，Testcontainers 会针对 Linux 上的较新 Docker 版本，以及 Mac 与 Windows 上的 Docker Desktop 进行积极测试。这些 Docker 环境会被 Testcontainers 自动检测并使用，无需额外配置。

你也可以配置 Testcontainers 以适配其他 Docker 设置，例如远程 Docker 主机或替代实现。但这些场景并未在主要开发流程中被积极测试，因此可能无法使用全部 Testcontainers 功能，且可能需要额外的手动配置。

如果你对自己的环境配置细节或其是否支持运行基于 Testcontainers 的测试仍有疑问，可在 [Slack](https://slack.testcontainers.org/) 上联系 Testcontainers 团队与社区用户。

 {{< grid items=quickstart >}}
