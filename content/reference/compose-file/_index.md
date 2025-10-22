---
description: 查找用于定义多容器应用程序的 Docker Compose 文件格式的最新推荐版本。
keywords: docker compose 文件, docker compose yml, docker compose 参考, docker
  compose 命令, docker compose 用户, docker compose 镜像, yaml 规范, docker compose
  语法, yaml 规范, docker compose 规范
title: Compose 文件参考
toc_max: 4
toc_min: 1
grid:
- title: 版本和名称顶级元素
  description: 了解 Compose 的版本和名称属性。
  icon: text_snippet
  link: /reference/compose-file/version-and-name/
- title: 服务顶级元素
  description: 探索 Compose 的所有服务属性。
  icon: construction
  link: /reference/compose-file/services/
- title: 网络顶级元素
  description: 查找 Compose 的所有网络属性。
  icon: lan
  link: /reference/compose-file/networks/
- title: 卷顶级元素
  description: 探索 Compose 的所有卷属性。
  icon: database
  link: /reference/compose-file/volumes/
- title: 配置顶级元素
  description: 了解 Compose 中的配置信息。
  icon: settings
  link: /reference/compose-file/configs/
- title: 密钥顶级元素
  description: 了解 Compose 中的密钥管理。
  icon: lock
  link: /reference/compose-file/secrets/
aliases:
 - /compose/yaml/
 - /compose/compose-file/compose-file-v1/
 - /compose/compose-file/
 - /compose/reference/overview/
---

>**Docker Compose 新手？**
>
> 了解 [Docker Compose 的主要功能和用例](/manuals/compose/intro/features-uses.md) 或 [尝试快速入门指南](/manuals/compose/gettingstarted.md)。

Compose 规范是 Compose 文件格式的最新推荐版本。它帮助您定义 [Compose 文件](/manuals/compose/intro/compose-application-model.md)，用于配置 Docker 应用程序的服务、网络、卷等内容。

Compose 文件格式的旧版本 2.x 和 3.x 已合并到 Compose 规范中。它在 Docker Compose CLI 的 1.27.0 及更高版本（也称为 Compose v2）中实现。

Docker 文档上的 Compose 规范是 Docker Compose 的实现。如果您希望实现自己的 Compose 规范版本，请参阅 [Compose 规范仓库](https://github.com/compose-spec/compose-spec)。

使用以下链接导航 Compose 规范的关键部分。

> [!TIP]
>
> 想要在 VS Code 中获得更好的 Compose 文件编辑体验？
> 查看 [Docker VS Code 扩展（测试版）](https://marketplace.visualstudio.com/items?itemName=docker.docker)，获取代码检查、代码导航和漏洞扫描功能。

{{< grid >}}
