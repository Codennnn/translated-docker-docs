---
title: 下一步 
keywords: concepts, build, images, container, docker desktop
description: 浏览循序渐进的指南，帮助你理解 Docker 核心概念、镜像构建与容器运行。
aliases:
 - /guides/getting-started/whats-next/
summary: |
  你已经完成 Docker Desktop 的安装、使用容器进行开发，
  并构建并推送了第一个镜像。接下来，你可以更进一步，
  深入理解容器是什么以及它如何工作。
notoc: true
weight: 4

the-basics:
- title: 什么是容器？
  description: 了解如何运行你的第一个容器。
  link: /get-started/docker-concepts/the-basics/what-is-a-container/
- title: 什么是镜像？
  description: 了解镜像分层的基础知识。 
  link: /get-started/docker-concepts/the-basics/what-is-an-image/
- title: 什么是仓库？
  description: 了解容器仓库，探索其互操作性，并学习与仓库交互。 
  link: /get-started/docker-concepts/the-basics/what-is-a-registry/
- title: 什么是 Docker Compose？
  description: 更系统地理解 Docker Compose。
  link: /get-started/docker-concepts/the-basics/what-is-docker-compose/

building-images:
- title: 理解镜像分层 
  description: 了解容器镜像的分层结构。
  link: /get-started/docker-concepts/building-images/understanding-image-layers/
- title: 编写 Dockerfile
  description: 学习如何使用 Dockerfile 创建镜像。
  link: /get-started/docker-concepts/building-images/writing-a-dockerfile/
- title: 构建、标记并发布镜像
  description: 学习如何构建、打标签并将镜像发布到 Docker Hub 或其他仓库。
  link: /get-started/docker-concepts/building-images/build-tag-and-publish-an-image/
- title: 使用构建缓存
  description: 了解构建缓存、哪些变更会使缓存失效，以及如何高效利用缓存。
  link: /get-started/docker-concepts/building-images/using-the-build-cache/
- title: 多阶段构建
  description: 更深入理解多阶段构建及其优势。
  link: /get-started/docker-concepts/building-images/multi-stage-builds/

running-containers:
- title: 发布端口
  description: 理解在 Docker 中发布与暴露端口的意义。
  link: /get-started/docker-concepts/running-containers/publishing-ports/
- title: 覆盖容器默认设置
  description: 学习如何通过 `docker run` 覆盖容器默认配置。
  link: /get-started/docker-concepts/running-containers/overriding-container-defaults/
- title: 持久化容器数据
  description: 了解在 Docker 中进行数据持久化的重要性。
  link: /get-started/docker-concepts/running-containers/persisting-container-data/
- title: 与容器共享本地文件
  description: 探索 Docker 提供的多种存储选项及其常见用法。
  link: /get-started/docker-concepts/running-containers/sharing-local-files/
- title: 多容器应用
  description: 了解多容器应用的价值，以及它与单容器应用的差异。
  link: /get-started/docker-concepts/running-containers/multi-container-applications/
---

以下章节提供循序渐进的指南，帮助你理解 Docker 核心概念、镜像构建与容器运行。

## 基础入门

从容器、镜像、仓库与 Docker Compose 的核心概念开始学习。

{{< grid items="the-basics" >}}

## 构建镜像

借助 Dockerfile、构建缓存与多阶段构建，打造更优的容器镜像。

{{< grid items="building-images" >}}

## 运行容器

掌握端口暴露、覆盖默认配置、数据持久化、文件共享与多容器应用管理等关键技巧。

{{< grid items="running-containers" >}}
