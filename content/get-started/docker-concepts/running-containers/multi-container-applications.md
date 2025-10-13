---
title: 多容器应用
weight: 5
keywords: concepts, build, images, container, docker desktop
description: 本文将介绍多容器应用的重要性，以及它与单容器应用的差异。
aliases: 
 - /guides/docker-concepts/running-containers/multi-container-applications/
---

{{< youtube-embed 1jUwR6F9hvM >}}

## 概念解析

启动一个单容器应用很容易。比如，一个用于特定数据处理任务的 Python 脚本，可以在包含其依赖的容器中运行；又如，一个提供静态站点与少量 API 的 Node.js 应用，也可以将所需库与依赖一并打包到容器中。然而，随着应用规模增长，仅用单个容器来承载与管理所有职责会越来越困难。

设想这个 Python 脚本需要连接数据库。你不再只需要管理脚本，还要在同一容器里管理数据库服务器。如果脚本还需要用户登录，你还得加入认证机制，容器会愈发臃肿。

容器的一条最佳实践是：一个容器只做好一件事。虽然并非绝对，但应尽量避免让单个容器承担多种职责。

这时你可能会问：“那我需要把它们拆成多个容器分别运行吗？如果分开运行，彼此如何连通？”

尽管 `docker run` 启动容器很方便，但用它来管理不断扩张的应用栈会变得吃力，原因包括：

- 需要在开发、测试、生产等环境下分别运行多个 `docker run` 命令（前端、后端、数据库），配置差异多且易出错、耗时长。
- 应用之间存在依赖关系。手动按顺序启动容器并维护网络连接，随着栈的扩展将愈发复杂。
- 每个应用都需要独立的 `docker run` 命令，难以对单个服务进行弹性伸缩；要扩容整个应用栈又可能造成不必要的资源浪费。
- 每个应用都要单独配置卷挂载或数据持久化，数据管理分散且不易维护。
- 需要为每个应用单独设置环境变量，繁琐且容易出错。

这正是 Docker Compose 大显身手的地方。

Docker Compose 用一个名为 `compose.yml` 的 YAML 文件来定义整个多容器应用。该文件集中描述所有容器、它们的依赖、环境变量，以及卷与网络等。使用 Docker Compose：

- 无需运行多个 `docker run` 命令，只需在一个 YAML 中定义整套应用，配置集中、管理更简单。
- 可以按特定顺序启动容器，网络连接配置也更直观。
- 可针对单个服务进行横向扩缩，实现按需分配资源。
- 易于配置持久化卷。
- 可以在 Compose 文件中统一设置环境变量。

借助 Docker Compose，你可以以模块化、可伸缩且一致性的方式构建复杂应用。

## 动手试试

本实践先使用 `docker run` 基于 Node.js 计数服务、Nginx 反向代理与 Redis 数据库搭建一个 Web 应用，然后演示如何用 Docker Compose 简化整个部署过程。

### 准备

1. 获取示例应用。若已安装 Git，可直接克隆示例仓库；否则可下载示例源码。任选其一：

   {{< tabs >}}
   {{< tab name="使用 git 克隆" >}}

   在终端执行以下命令克隆示例仓库：

   ```console
   $ git clone https://github.com/dockersamples/nginx-node-redis
   ```
   
   进入 `nginx-node-redis` 目录：

   ```console
   $ cd nginx-node-redis
   ```

   该目录下包含两个子目录：`nginx` 与 `web`。
   

   {{< /tab >}}
   {{< tab name="下载" >}}

   下载源码并解压。

   {{< button url="https://github.com/dockersamples/nginx-node-redis/archive/refs/heads/main.zip" text="下载源码" >}}


   进入 `nginx-node-redis-main` 目录：

   ```console
   $ cd nginx-node-redis-main
   ```

   该目录下包含两个子目录：`nginx` 与 `web`。

   {{< /tab >}}
   {{< /tabs >}}


2. [下载并安装](/get-started/get-docker.md) Docker Desktop。

### 构建镜像

1. 进入 `nginx` 目录并构建镜像：

    ```console
    $ docker build -t nginx .
    ```

2. 进入 `web` 目录并构建第一个 web 镜像：
    
    ```console
    $ docker build -t web .
    ```

### 运行容器

1. 在运行多容器应用前，先创建一个供各容器通信的网络，使用 `docker network create`：

    ```console
    $ docker network create sample-app
    ```

2. 启动 Redis 容器，将其连接到上述网络并设置网络别名（用于 DNS 解析）：

    ```console
    $ docker run -d  --name redis --network sample-app --network-alias redis redis
    ```

3. 启动第一个 web 容器：

    ```console
    $ docker run -d --name web1 -h web1 --network sample-app --network-alias web1 web
    ```

4. 启动第二个 web 容器：

    ```console
    $ docker run -d --name web2 -h web2 --network sample-app --network-alias web2 web
    ```
    
5. 启动 Nginx 容器：

    ```console
    $ docker run -d --name nginx --network sample-app  -p 80:80 nginx
    ```

     > [!NOTE]
     >
     > Nginx 常作为 Web 应用的反向代理，用于把流量路由到后端服务。在本例中，它会转发到 Node.js 后端容器（web1 或 web2）。

6. 使用以下命令查看容器是否已启动：

    ```console
    $ docker ps
    ```

    你将看到类似输出： 

    ```text
    CONTAINER ID   IMAGE     COMMAND                  CREATED              STATUS              PORTS                NAMES
    2cf7c484c144   nginx     "/docker-entrypoint.…"   9 seconds ago        Up 8 seconds        0.0.0.0:80->80/tcp   nginx
    7a070c9ffeaa   web       "docker-entrypoint.s…"   19 seconds ago       Up 18 seconds                            web2
    6dc6d4e60aaf   web       "docker-entrypoint.s…"   34 seconds ago       Up 33 seconds                            web1
    008e0ecf4f36   redis     "docker-entrypoint.s…"   About a minute ago   Up About a minute   6379/tcp             redis
    ```

7. 打开 Docker Desktop 控制面板，可以看到这些容器，并进一步查看它们的配置。

   ![Docker Desktop Dashboard 截图，展示多容器应用](images/multi-container-apps.webp?w=5000&border=true)

8. 一切启动就绪后，打开浏览器访问 [http://localhost](http://localhost) 查看站点。多次刷新页面，可看到处理请求的主机名和累计访问次数：

    ```console
    web2: Number of visits is: 9
    web1: Number of visits is: 10
    web2: Number of visits is: 11
    web1: Number of visits is: 12
    ```

    > [!NOTE]
    >
    > 你可能注意到，作为反向代理的 Nginx 很可能以轮询（round-robin）的方式在两个后端容器之间分发请求。因此，每次请求可能被转发到不同的容器（web1 与 web2）。输出显示 web1 与 web2 的计数交替增长，而 Redis 中的实际计数是在响应返回客户端后才更新的。

9. 你可以在 Docker Desktop 控制面板中选中这些容器并点击 **Delete** 来删除它们。

   ![Docker Desktop Dashboard 截图，展示如何删除多容器应用](images/delete-multi-container-apps.webp?border=true)
 
## 使用 Docker Compose 简化部署

Docker Compose 为多容器部署提供了结构化、流程化的管理方式。正如上文所述，借助 Compose 无需运行多个 `docker run` 命令，只需在名为 `compose.yml` 的 YAML 文件中定义整个多容器应用。其工作方式如下。

进入项目根目录。你会看到一个 `compose.yml` 文件。这个 YAML 文件定义了构成应用的所有服务及其配置：每个服务的镜像、端口、卷、网络以及运行所需的其他设置。

1. 使用 `docker compose up` 启动应用：

    ```console
    $ docker compose up -d --build
    ```

    你将看到类似如下输出：

    ```console
    Running 5/5
    ✔ Network nginx-nodejs-redis_default    Created                                                0.0s
    ✔ Container nginx-nodejs-redis-web1-1   Started                                                0.1s
    ✔ Container nginx-nodejs-redis-redis-1  Started                                                0.1s
    ✔ Container nginx-nodejs-redis-web2-1   Started                                                0.1s
    ✔ Container nginx-nodejs-redis-nginx-1  Started
    ```

2. 打开 Docker Desktop 控制面板，可以查看到该应用栈的各个容器并深入其配置。

    ![Docker Desktop Dashboard 截图，展示通过 Docker Compose 部署的应用容器列表](images/list-containers.webp?border=true)

3. 也可以在 Docker Desktop 控制面板中选中该应用栈并点击 **Delete** 来移除通过 Docker Compose 部署的容器。

   ![Docker Desktop Dashboard 截图，展示如何移除通过 Docker Compose 部署的容器](images/delete-containers.webp?border=true)

在本指南中，你学习了为何相较于易错且难以管理的 `docker run`，使用 Docker Compose 启停多容器应用更加轻松高效。

## 延伸阅读

* [`docker container run` 命令参考](reference/cli/docker/container/run/)
* [什么是 Docker Compose](/get-started/docker-concepts/the-basics/what-is-docker-compose/)

