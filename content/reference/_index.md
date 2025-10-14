---
title: 参考文档
linkTitle: 参考
layout: wide
description: 查阅 Docker 平台各类 API、CLI 以及文件格式的参考文档
params:
  icon: terminal
  notoc: true
  grid_files:
  - title: Dockerfile
    description: 定义单个容器的内容与启动行为。
    icon: edit_document
    link: /reference/dockerfile/
  - title: Compose 文件
    description: 定义一个多容器应用。
    icon: polyline
    link: /reference/compose-file/
  grid_clis:
  - title: Docker CLI
    description: Docker 的主要命令行工具，包含所有 `docker` 命令。
    icon: terminal
    link: /reference/cli/docker/
  - title: Compose CLI
    description: 用于 Docker Compose 的命令行工具，用于构建与运行多容器应用。
    icon: subtitles
    link: /reference/cli/docker/compose/
  - title: 守护进程 CLI（dockerd）
    description: 管理容器的持久化后台进程。
    icon: developer_board
    link: /reference/cli/dockerd/
  grid_apis:
  - title: 引擎 API
    description: Docker 的主要 API，提供对守护进程的编程访问接口。
    icon: api
    link: /reference/api/engine/
  - title: Docker Hub API
    description: 用于与 Docker Hub 交互的 API。
    icon: communities
    link: /reference/api/hub/latest/
  - title: DVP 数据 API
    description: 为 Docker 验证发布者（Docker Verified Publishers）提供分析数据访问的 API。
    icon: area_chart
    link: /reference/api/dvp/latest/
  - title: Registry API
    description: Docker 镜像仓库的 API。
    icon: database
    link: /reference/api/registry/latest/
---

本节包含 Docker 平台各类 API、CLI、驱动规范以及文件格式的参考文档。

## 文件格式

{{< grid items="grid_files" >}}

## 命令行接口（CLI）

{{< grid items="grid_clis" >}}

## 应用程序编程接口（API）

{{< grid items="grid_apis" >}}