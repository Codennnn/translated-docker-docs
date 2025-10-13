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

Docker Desktop 是一个一体化套件，可用于构建镜像、运行容器等更多操作。
本指南将带你完成安装流程，让你亲身体验 Docker Desktop。


> **Docker Desktop 使用条款**
>
> 若在大型企业中将 Docker Desktop 用于商业用途（员工人数超过 250 人，或年营收超过 1000 万美元），需要购买[付费订阅](https://www.docker.com/pricing/?_gl=1*1nyypal*_ga*MTYxMTUxMzkzOS4xNjgzNTM0MTcw*_ga_XJWPQMJYHQ*MTcxNjk4MzU4Mi4xMjE2LjEuMTcxNjk4MzkzNS4xNy4wLjA.).

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

安装完成后，继续完成初始化设置，你就可以开始运行 Docker 容器了。

## 试一试

在这个动手指南中，你将了解如何使用 Docker Desktop 运行一个 Docker 容器。

按照以下步骤，通过 CLI 启动一个容器。


## 运行第一个容器

打开命令行终端，使用 `docker run` 命令启动一个容器：



```console
$ docker run -d -p 8080:80 docker/welcome-to-docker
```

## 访问前端

对于该容器，前端服务映射到 `8080` 端口。打开浏览器访问 [http://localhost:8080](http://localhost:8080) 即可查看页面。





![来自正在运行容器的 Nginx Web 服务器首页截图](../docker-concepts/the-basics/images/access-the-frontend.webp?border=true)

## 使用 Docker Desktop 管理容器


1. 打开 Docker Desktop，在左侧边栏选择 **Containers**。
2. 你可以查看容器的日志与文件，并可在 **Exec** 选项卡中进入容器终端。

   ![在 Docker Desktop 中进入正在运行容器终端的截图](images/exec-into-docker-container.webp?border=true)


3. 选择 **Inspect** 获取容器的详细信息。你还可以执行暂停、恢复、启动或停止等操作，或查看 **Logs**、**Bind mounts**、**Exec**、**Files**、**Stats** 等选项卡。

![在 Docker Desktop 中检查正在运行容器的截图](images/inspecting-container.webp?border=true)

Docker Desktop 通过简化跨环境的安装、配置与兼容性，为开发者提供一致体验，从而缓解环境不一致与部署困难等痛点。

## 下一步

现在你已安装 Docker Desktop 并运行了第一个容器，是时候开始使用容器进行开发了。

{{< button text="使用容器进行开发" url="develop-with-containers" >}}
