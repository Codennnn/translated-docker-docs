---
title: 什么是镜像？
weight: 20
keywords: concepts, build, images, container, docker desktop
description: 镜像是什么
aliases:
  - /guides/docker-concepts/the-basics/what-is-an-image/
  - /get-started/run-docker-hub-images/
---

{{< youtube-embed NyvT9REqLe4 >}}

## 概念解析

既然[容器](./what-is-a-container.md)本质上是一个隔离的进程，它的文件与配置从哪里来？又该如何在不同环境之间共享这些运行环境？

答案就是容器镜像。容器镜像是一个标准化的软件包，包含运行容器所需的全部文件、二进制、库以及配置。

以 [PostgreSQL](https://hub.docker.com/_/postgres) 镜像为例，它会打包数据库二进制、配置文件和其他依赖。对于一个 Python Web 应用，镜像会包含 Python 运行时、你的应用代码以及其所有依赖。

镜像有两个重要原则：

1. 镜像是不可变的。一旦镜像创建完成，就不能被修改；你只能基于它创建一个新的镜像，或在其之上追加更改。
2. 容器镜像由多层组成。每一层代表一组对文件系统的变更，用于新增、删除或修改文件。

基于这两个原则，你可以方便地扩展或叠加现有镜像。例如构建一个 Python 应用时，可以从[Python 镜像](https://hub.docker.com/_/python)起步，添加额外的层来安装应用依赖并加入你的代码。这样你就能把精力集中在应用本身，而不是底层的 Python 运行时。

### 查找镜像

[Docker Hub](https://hub.docker.com) 是默认的全球镜像存储与分发平台，拥有十万余个由开发者创建、可直接本地运行的镜像。你可以在 Docker Desktop 中搜索 Docker Hub 上的镜像并直接运行它们。

Docker Hub 提供一系列由 Docker 支持或背书的镜像，统称为 Docker Trusted Content。它们要么提供完整托管服务，要么是构建自定义镜像的优质起点，包括：

- [Docker Official Images](https://hub.docker.com/search?q=&type=image&image_filter=official) —— 经精心维护的官方镜像集合，常作为大多数用户的起点，也是 Docker Hub 上安全性较高的一类镜像。
- [Docker Verified Publishers](https://hub.docker.com/search?q=&image_filter=store) —— 由 Docker 验证的商业发布者提供的高质量镜像。
- [Docker-Sponsored Open Source](https://hub.docker.com/search?q=&image_filter=open_source) —— 由 Docker 开源赞助计划支持的开源项目发布并维护的镜像。

例如，[Redis](https://hub.docker.com/_/redis) 与 [Memcached](https://hub.docker.com/_/memcached) 都是流行、可即开即用的 Docker Official Images。你可以下载这些镜像，在数秒内启动服务。还有一些基础镜像，如 [Node.js](https://hub.docker.com/_/node) Docker 镜像，可作为起点在其之上添加你的文件与配置。

## 动手试试

{{< tabs group=concept-usage persist=true >}}
{{< tab name="使用 GUI" >}}

本实践将演示如何使用 Docker Desktop GUI 搜索并拉取一个容器镜像。

### 搜索并下载镜像

1. 打开 Docker Desktop 控制面板，在左侧导航中选择 **Images** 视图。

   ![Docker Desktop 控制面板截图，左侧栏选中 Images 视图](images/click-image.webp?border=true&w=1050&h=400)

2. 选择 **Search images to run** 按钮；如果未看到该按钮，请点击顶部的_全局搜索栏_。

   ![Docker Desktop 控制面板截图，展示搜索标签页](images/search-image.webp?border)

3. 在 **Search** 输入框中输入 "welcome-to-docker"。搜索完成后，选择 `docker/welcome-to-docker` 镜像。

   ![Docker Desktop 控制面板截图，显示 docker/welcome-to-docker 的搜索结果](images/select-image.webp?border=true&w=1050&h=400)

4. 选择 **Pull** 下载该镜像。

### 了解该镜像

镜像下载完成后，你可以通过 GUI 或 CLI 查看关于该镜像的诸多细节。

1. 在 Docker Desktop 控制面板中，进入 **Images** 视图。

2. 选择 **docker/welcome-to-docker** 镜像以打开其详情。

   ![镜像视图截图，箭头指向 docker/welcome-to-docker 镜像](images/pulled-image.webp?border=true&w=1050&h=400)

3. 镜像详情页会展示镜像的各层、镜像中安装的软件包和库，以及任何被发现的漏洞信息。

   ![docker/welcome-to-docker 的镜像详情视图截图](images/image-layers.webp?border=true&w=1050&h=400)

{{< /tab >}}

{{< tab name="使用 CLI" >}}

按以下步骤使用 CLI 搜索并拉取 Docker 镜像，并查看其分层信息。

### 搜索并下载镜像

1. 打开终端，使用 [`docker search`](/reference/cli/docker/search.md) 命令搜索镜像：

   ```console
   docker search docker/welcome-to-docker
   ```

   你将看到类似如下输出：

   ```console
   NAME                       DESCRIPTION                                     STARS     OFFICIAL
   docker/welcome-to-docker   Docker image for new users getting started w…   20
   ```

   上述输出展示了 Docker Hub 上可用的相关镜像信息。

2. 使用 [`docker pull`](/reference/cli/docker/image/pull.md) 命令拉取镜像：

   ```console
   docker pull docker/welcome-to-docker
   ```

   你将看到类似如下输出：

   ```console
   Using default tag: latest
   latest: Pulling from docker/welcome-to-docker
   579b34f0a95b: Download complete
   d11a451e6399: Download complete
   1c2214f9937c: Download complete
   b42a2f288f4d: Download complete
   54b19e12c655: Download complete
   1fb28e078240: Download complete
   94be7e780731: Download complete
   89578ce72c35: Download complete
   Digest: sha256:eedaff45e3c78538087bdd9dc7afafac7e110061bbdd836af4104b10f10ab693
   Status: Downloaded newer image for docker/welcome-to-docker:latest
   docker.io/docker/welcome-to-docker:latest
   ```

   以上每一行对应镜像中的一个已下载层。请记住，每一层都是一组文件系统变更，为镜像提供相应功能。

### 了解该镜像

1. 使用 [`docker image ls`](/reference/cli/docker/image/ls.md) 命令列出本地镜像：

   ```console
   docker image ls
   ```

   你将看到类似如下输出：

   ```console
   REPOSITORY                 TAG       IMAGE ID       CREATED        SIZE
   docker/welcome-to-docker   latest    eedaff45e3c7   4 months ago   29.7MB
   ```

   该命令会显示当前系统中可用的 Docker 镜像列表。`docker/welcome-to-docker` 的总大小约为 29.7MB。

   > **关于镜像大小**
   >
   > 这里展示的是镜像的未压缩大小，并非各层的下载大小。

2. 使用 [`docker image history`](/reference/cli/docker/image/history.md) 命令查看镜像的分层历史：

   ```console
   docker image history docker/welcome-to-docker
   ```

   你将看到类似如下输出：

   ```console
   IMAGE          CREATED        CREATED BY                                      SIZE      COMMENT
   648f93a1ba7d   4 months ago   COPY /app/build /usr/share/nginx/html # buil…   1.6MB     buildkit.dockerfile.v0
   <missing>      5 months ago   /bin/sh -c #(nop)  CMD ["nginx" "-g" "daemon…   0B
   <missing>      5 months ago   /bin/sh -c #(nop)  STOPSIGNAL SIGQUIT           0B
   <missing>      5 months ago   /bin/sh -c #(nop)  EXPOSE 80                    0B
   <missing>      5 months ago   /bin/sh -c #(nop)  ENTRYPOINT ["/docker-entr…   0B
   <missing>      5 months ago   /bin/sh -c #(nop) COPY file:9e3b2b63db9f8fc7…   4.62kB
   <missing>      5 months ago   /bin/sh -c #(nop) COPY file:57846632accc8975…   3.02kB
   <missing>      5 months ago   /bin/sh -c #(nop) COPY file:3b1b9915b7dd898a…   298B
   <missing>      5 months ago   /bin/sh -c #(nop) COPY file:caec368f5a54f70a…   2.12kB
   <missing>      5 months ago   /bin/sh -c #(nop) COPY file:01e75c6dd0ce317d…   1.62kB
   <missing>      5 months ago   /bin/sh -c set -x     && addgroup -g 101 -S …   9.7MB
   <missing>      5 months ago   /bin/sh -c #(nop)  ENV PKG_RELEASE=1            0B
   <missing>      5 months ago   /bin/sh -c #(nop)  ENV NGINX_VERSION=1.25.3     0B
   <missing>      5 months ago   /bin/sh -c #(nop)  LABEL maintainer=NGINX Do…   0B
   <missing>      5 months ago   /bin/sh -c #(nop)  CMD ["/bin/sh"]              0B
   <missing>      5 months ago   /bin/sh -c #(nop) ADD file:ff3112828967e8004…   7.66MB
   ```

   该输出展示了所有分层、各自的大小，以及创建每一层时所执行的命令。

   > **查看完整命令**
   >
   > 添加 `--no-trunc` 参数可以显示完整命令。需要注意的是，输出为表格样式，命令过长时会降低可读性。

{{< /tab >}}
{{< /tabs >}}

通过本次演练，你已完成 Docker 镜像的搜索与拉取，并了解了镜像的分层结构。

## 延伸阅读

以下资源可帮助你进一步了解镜像的探索、查找与构建：

- [Docker 可信内容](/manuals/docker-hub/image-library/trusted-content.md)
- [探索 Docker Desktop 的镜像视图](/manuals/desktop/use-desktop/images.md)
- [Docker 构建概览](/manuals/build/concepts/overview.md)
- [Docker Hub](https://hub.docker.com)

## 下一步

既然你已经掌握了镜像的基础知识，接下来可以学习如何通过仓库分发镜像。

{{< button text="什么是仓库？" url="what-is-a-registry" >}}
