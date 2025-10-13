---
title: 发布与暴露端口
keywords: concepts, build, images, container, docker desktop
description: 本文将介绍在 Docker 中发布（publish）与暴露（expose）端口的重要性。
weight: 1
aliases: 
 - /guides/docker-concepts/running-containers/publishing-ports/
---

{{< youtube-embed 9JnqOmJ96ds >}}

## 概念解析

如果你一直跟随本系列进行练习，你已经了解容器会为应用的每个组件提供隔离的进程环境。每个组件——例如 React 前端、Python API 与 Postgres 数据库——都在各自的沙箱环境中运行，与宿主机上的其他一切完全隔离。这种隔离有利于安全与依赖治理，但也意味着你无法直接访问它们，例如无法直接在浏览器里访问 Web 应用。

这时就需要“端口发布”。

### 发布端口（Publishing ports）

发布端口允许你在一定程度上突破网络隔离，通过建立转发规则，将宿主机收到的请求转发到容器内部。例如，你可以指定把宿主机的 `8080` 端口转发到容器的 `80` 端口。发布端口发生在创建容器时，通过 `docker run` 的 `-p`（或 `--publish`）参数实现。其语法为：

```console
$ docker run -d -p HOST_PORT:CONTAINER_PORT nginx
```

- `HOST_PORT`：宿主机上用于接收流量的端口号
- `CONTAINER_PORT`：容器内部用于监听连接的端口号

例如，将容器的 `80` 端口发布到宿主机的 `8080`：

```console
$ docker run -d -p 8080:80 nginx
```

现在，发往宿主机 `8080` 端口的任何流量都会被转发到容器内的 `80` 端口。

> [!IMPORTANT]
>
> 端口一旦发布，默认会绑定到所有网络接口。这意味着任何能访问到你机器的流量都可以访问已发布的应用。请谨慎发布数据库等敏感服务。[在此了解更多已发布端口的细节](/engine/network/#published-ports)。

### 发布到临时端口（Ephemeral ports）

有时你只想发布端口，但并不关心具体映射到宿主机的哪个端口。这种情况下，可以让 Docker 自动选择端口。只需省略 `HOST_PORT` 即可。

例如，以下命令会把容器的 `80` 端口发布到宿主机的一个临时端口：

```console
$ docker run -p 80 nginx
```
 
容器启动后，使用 `docker ps` 可查看被分配的端口：

```console
docker ps
CONTAINER ID   IMAGE         COMMAND                  CREATED          STATUS          PORTS                    NAMES
a527355c9c53   nginx         "/docker-entrypoint.…"   4 seconds ago    Up 3 seconds    0.0.0.0:54772->80/tcp    romantic_williamson
```

在这个示例中，应用通过宿主机的 `54772` 端口对外提供服务。

### 发布所有暴露的端口

创建镜像时，可以使用 `EXPOSE` 指令声明镜像内的应用将使用某些端口。这些端口默认不会被发布。

使用 `-P` 或 `--publish-all` 参数，可以将镜像中所有通过 `EXPOSE` 声明的端口，自动发布到宿主机的临时端口。这对于在开发或测试环境中避免端口冲突非常有用。

例如，以下命令会发布镜像配置中声明的所有端口：

```console
$ docker run -P nginx
```

## 动手试试

本实践将演示如何使用 CLI 与 Docker Compose 发布容器端口来部署一个 Web 应用。

### 使用 Docker CLI

本步骤中，你将使用 Docker CLI 运行一个容器并发布其端口。

1. [下载并安装](/get-started/get-docker/) Docker Desktop。

2. 在终端中运行以下命令启动一个新容器：

    ```console
    $ docker run -d -p 8080:80 docker/welcome-to-docker
    ```

    其中，第一个 `8080` 指宿主机端口，即你本机用于访问容器内应用的端口；第二个 `80` 指容器端口，即容器内应用对外监听的端口。因此，该命令将宿主机的 `8080` 绑定到容器的 `80`。

3. 打开 Docker Desktop 控制面板的 **Containers** 视图，验证端口是否已发布。

   ![Docker Desktop Dashboard 截图，显示已发布端口](images/published-ports.webp?w=5000&border=true)

4. 通过点击容器 **Port(s)** 列的链接，或在浏览器访问 [http://localhost:8080](http://localhost:8080) 打开站点。

   ![在容器中运行的 Nginx 首页截图](/get-started/docker-concepts/the-basics/images/access-the-frontend.webp?border=true)

### 使用 Docker Compose

下面用 Docker Compose 启动同一个应用：

1. 新建一个目录，并在其中创建 `compose.yaml` 文件，内容如下：

    ```yaml
    services:
      app:
        image: docker/welcome-to-docker
        ports:
          - 8080:80
    ```

    `ports` 支持多种语法形式，这里使用和 `docker run` 一致的 `HOST_PORT:CONTAINER_PORT` 格式。

2. 打开终端，进入上一步创建的目录。

3. 使用 `docker compose up` 启动应用。

4. 打开浏览器访问 [http://localhost:8080](http://localhost:8080)。

## 延伸阅读

如果你想更深入了解相关主题，请参考：

* [`docker container port` 命令参考](/reference/cli/docker/container/port/)
* [已发布端口](/engine/network/#published-ports)

## 下一步

现在你已经理解如何发布与暴露端口，接下来学习如何使用 `docker run` 覆盖容器默认设置。

{{< button text="覆盖容器默认值" url="overriding-container-defaults" >}}

