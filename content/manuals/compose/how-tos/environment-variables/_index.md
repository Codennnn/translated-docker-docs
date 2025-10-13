---
title: 在 Compose 中使用环境变量
linkTitle: 使用环境变量
weight: 40
description: 说明如何在 Docker Compose 中设置、使用与管理环境变量。
keywords: compose, orchestration, environment, env file
aliases:
- /compose/environment-variables/
---

在 Docker Compose 中，环境变量与变量插值可帮助你创建可复用且灵活的配置，使基于 Docker 的应用能够更轻松地在不同环境中管理与部署。

> [!TIP]
>
> 在使用环境变量之前，建议先通读本文，全面了解 Docker Compose 中的环境变量。

本节包括：

- [如何在容器环境中设置环境变量](set-environment-variables.md)。
- [容器环境中环境变量的优先级如何生效](envvars-precedence.md)。
- [预定义的环境变量](envvars.md)。

还包括： 
- 如何在 Compose 文件中使用[插值](variable-interpolation.md)来设置变量，以及它与容器环境的关系。
- 一些[最佳实践](best-practices.md)。
