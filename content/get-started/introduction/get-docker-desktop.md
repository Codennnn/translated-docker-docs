---
title: 获取 Docker Desktop
keywords: concepts, container, docker desktop
description: 本页将指导你在 Windows、Mac 和 Linux 上下载并安装 Docker Desktop。
summary: |
  让 Docker Desktop 运行起来，是开发者迈向容器化的关键第一步。
  它提供直观、易用的一体化界面，帮助你管理 Docker 容器。
  Docker Desktop 简化了在容器中构建、共享与运行应用的流程，
  并确保在不同环境之间保持一致性。
weight: 1
aliases:
  - /getting-started/get-docker-desktop/
---

{{< youtube-embed C2bPVhiNU-0 >}}

## 说明

Docker Desktop 是一个功能完整的一体化应用平台，让你能够轻松构建镜像、运行容器，并完成更多容器化开发任务。
本指南将带你一步步完成安装，让你快速上手 Docker Desktop 的强大功能。


> **Docker Desktop 使用条款**
>
> 如果你在大型企业中（员工人数超过 250 人，或年营收超过 1000 万美元）将 Docker Desktop 用于商业用途，需要购买[付费订阅](https://www.docker.com/pricing/?_gl=1*1nyypal*_ga*MTYxMTUxMzkzOS4xNjgzNTM0MTcw*_ga_XJWPQMJYHQ*MTcxNjk4MzU4Mi4xMjE2LjEuMTcxNjk4MzkzNS4xNy4wLjA.)。

<div class="not-prose">
{{< card
  title="适用于 Mac 的 Docker Desktop"
  description="[下载（Apple Silicon）](https://desktop.docker.com/mac/main/arm64/Docker.dmg?utm_source=docker&utm_medium=webreferral&utm_campaign=docs-driven-download-mac-arm64) | [下载（Intel）](https://desktop.docker.com/mac/main/amd64/Docker.dmg?utm_source=docker&utm_medium=webreferral&utm_campaign=docs-driven-download-mac-amd64) | [安装指南](/desktop/setup/install/mac-install)"
  icon="/icons/AppleMac.svg" >}}

{{< card
  title="适用于 Windows 的 Docker Desktop"
  description="[下载](https://desktop.docker.com/win/main/amd64/Docker%20Desktop%20Installer.exe?utm_source=docker&utm_medium=webreferral&utm_campaign=docs-driven-download-windows) | [安装指南](/desktop/setup/install/windows-install)"
  icon="/icons/Windows.svg" >}}

{{< card
  title="适用于 Linux 的 Docker Desktop"
  description="[安装指南](/desktop/setup/install/linux/)"
  icon="/icons/Linux.svg" >}}
</div>

安装完成后，只需完成简单的初始化设置，你就可以立即开始运行 Docker 容器了。

## 动手实践

在这个实践指南中，你将学习如何使用 Docker Desktop 来运行 Docker 容器。

请按照以下步骤，通过命令行界面启动你的第一个容器。

## 运行你的第一个容器

打开命令行终端，使用 `docker run` 命令来启动容器：

```console
$ docker run -d -p 8080:80 docker/welcome-to-docker
```

## 访问前端界面

容器启动后，前端服务会映射到主机的 `8080` 端口。打开浏览器并访问 [http://localhost:8080](http://localhost:8080)，你就能看到欢迎页面了。

![来自正在运行容器的 Nginx Web 服务器首页截图](../docker-concepts/the-basics/images/access-the-frontend.webp?border=true)

## 使用 Docker Desktop 管理容器

1. 打开 Docker Desktop，在左侧边栏点击 **Containers** 选项。
2. 在这里，你可以实时查看容器的运行日志和文件系统，还能通过 **Exec** 选项卡直接进入容器的命令行终端。

![在 Docker Desktop 中进入正在运行容器终端的截图](images/exec-into-docker-container.webp?border=true)

3. 点击 **Inspect** 可以查看容器的详细配置信息。你还可以对容器执行暂停、恢复、启动或停止等操作，同时查看 **Logs**（日志）、**Bind mounts**（挂载点）、**Exec**（终端）、**Files**（文件）、**Stats**（统计）等多个选项卡。

![在 Docker Desktop 中检查正在运行容器的截图](images/inspecting-container.webp?border=true)

Docker Desktop 通过简化跨平台的安装流程、统一配置管理和兼容性处理，为开发者提供一致的容器化体验，有效解决了开发环境不一致和部署复杂等常见痛点。

## 下一步

现在你已经成功安装 Docker Desktop 并运行了第一个容器，是时候深入学习如何使用容器进行应用开发了。

{{< button text="使用容器进行开发" url="develop-with-containers" >}}
