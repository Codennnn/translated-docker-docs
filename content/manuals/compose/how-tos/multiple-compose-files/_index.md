---
description: 在 Docker Compose 中使用多个 Compose 文件的不同方式概览
keywords: compose, compose file, merge, extends, include, docker compose, -f flag
linkTitle: 使用多个 Compose 文件
title: 使用多个 Compose 文件
weight: 80
aliases:
- /compose/multiple-compose-files/
---

本节介绍在项目中如何以多种方式使用多个 Compose 文件。

使用多个 Compose 文件可以针对不同环境或工作流定制 Compose 应用。这对包含数十个容器、由多个团队共同维护的大型应用尤为有用。例如，如果你的组织使用单一代码库（monorepo），每个团队可能都有自己的“本地” Compose 文件来运行其负责的子集；同时，他们还需要依赖其他团队提供参考的 Compose 文件，以定义这些子集应如何按预期方式运行。随着这种模式发展，复杂性会从代码本身转移到基础设施与配置文件上。

处理多个 Compose 文件的最快方式，是在命令行使用 `-f` 标志列出目标 Compose 文件并进行[合并](merge.md)。但受[合并规则](merge.md#merging-rules)影响，这很快会变得复杂。

为应对多文件场景带来的复杂性，Docker Compose 还提供另外两种选项。根据项目需要，你可以：

- 通过[扩展 Compose 文件](extends.md)：引用另一份 Compose 文件，仅复用你需要的部分，并可覆盖部分属性。
- 在你的 Compose 文件中直接[包含其他 Compose 文件](include.md)。
