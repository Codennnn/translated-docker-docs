---
title: Docker Compose 的历史与演进
linkTitle: 历史与演进
description: 回顾 Docker Compose 从 v1 到 v2 的演进，包括 CLI 变更、YAML 版本演进与 Compose 规范。
keywords: compose, compose yaml, swarm, migration, compatibility, docker compose vs docker-compose
weight: 30
aliases:
- /compose/history/
---

本页将为你概览：
 - Docker Compose CLI 的发展简史
 - 组成 Compose v1 与 v2 的主要版本与文件格式说明
 - Compose v1 与 v2 的主要差异 

## 简介

![展示 Compose v1 与 v2 主要差异的示意图](../images/v1-versus-v2.png)

上图显示，目前受支持的 Docker Compose CLI 版本为 Compose v2，其由[Compose 规范](/reference/compose-file/_index.md)定义。

图中也快速对比了文件格式、命令行语法与顶层元素的差异。下文将进一步展开。

### Docker Compose CLI 版本沿革

Docker Compose 命令行二进制的 v1 首发于 2014 年，采用 Python 编写，通过 `docker-compose` 调用。
典型的 Compose v1 项目会在 `compose.yaml` 文件中包含顶层的 `version` 字段，取值范围从 `2.0` 到 `3.8`，对应特定的[文件格式](#compose-file-format-versioning)。

Docker Compose 命令行二进制的 v2 于 2020 年发布，采用 Go 编写，通过 `docker compose` 调用。
Compose v2 不再使用 `compose.yaml` 顶层的 `version` 字段。

### Compose 文件格式版本 {#compose-file-format-versioning}

Docker Compose CLI 的行为由特定的文件格式所定义。

围绕 Compose v1，先后发布了三代主要文件格式：
- 文件格式 1（随 Compose 1.0.0 于 2014 年发布）
- 文件格式 2.x（随 Compose 1.6.0 于 2016 年发布）
- 文件格式 3.x（随 Compose 1.10.0 于 2017 年发布）

文件格式 1 与后续格式差异很大，因为它缺少顶层 `services` 键。
该格式现已历史化，无法在 Compose v2 中运行。

文件格式 2.x 与 3.x 彼此非常相似，但后者针对 Swarm 部署引入了许多新选项。

为解决关于 CLI 版本、文件格式版本，以及是否启用 Swarm 模式导致的能力差异等混淆，2.x 与 3.x 文件格式被合并进[Compose 规范](/reference/compose-file/_index.md)。 

Compose v2 基于 Compose 规范来定义项目。不同于过去的文件格式，Compose 规范是“滚动演进”的，并将顶层 `version` 字段设为可选。Compose v2 还引入了可选规范模块——[部署](/reference/compose-file/deploy.md)、[开发](/reference/compose-file/develop.md)与[构建](/reference/compose-file/build.md)。

为简化[迁移](/manuals/compose/releases/migrate.md)，在 2.x/3.x 文件格式与 Compose 规范之间，Compose v2 对部分已废弃或变更的元素提供后向兼容。

## 下一步

- [Compose 的工作原理](compose-application-model.md)
- [Compose 规范参考](/reference/compose-file/_index.md)
- [从 Compose v1 迁移到 v2](/manuals/compose/releases/migrate.md)
