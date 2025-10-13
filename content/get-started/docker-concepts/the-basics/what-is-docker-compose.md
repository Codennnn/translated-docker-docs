---
title: 什么是 Docker Compose？
weight: 40
keywords: concepts, build, images, container, docker desktop
description: 什么是 Docker Compose？
aliases:
 - /guides/walkthroughs/multi-container-apps/
 - /guides/docker-concepts/the-basics/what-is-docker-compose/
---

{{< youtube-embed xhcUIK4fGtY >}}

## 说明

如果你一直跟着指南学习，那么此前你主要在处理“单容器”应用。接下来，当你想运行数据库、消息队列、缓存或其他多个服务时，是把所有内容装进一个容器，还是运行多个容器？如果是多个容器，又该如何将它们连接在一起？

关于容器的最佳实践之一：每个容器只做一件事并把它做好。虽然也有例外，但应尽量避免“一个容器做很多事”。

你当然可以通过多次执行 `docker run` 来启动多个容器。但很快你会发现，需要管理自定义网络、为连接到网络的容器传递各种参数等；而在结束后，清理也会变得更复杂。

借助 Docker Compose，你可以在一个 YAML 文件中定义所有容器及其配置。只要将该文件随代码一起提交，任何克隆仓库的人都可以通过一条命令启动整个应用。

需要强调的是，Compose 是一个声明式工具：你只需定义想要的状态即可，无需每次从零开始。当你修改配置后，再次运行 `docker compose up`，Compose 会“对账”并智能应用差异。

> **Dockerfile 与 Compose 文件**
>
> Dockerfile 用于描述如何构建容器镜像；而 Compose 文件用于定义要运行的容器。很多时候，Compose 文件会引用某个 Dockerfile 来构建特定服务所需的镜像。


## 试一试 

在本示例中，你将学习如何使用 Docker Compose 运行一个多容器应用。我们将以一个使用 Node.js 与 MySQL 的待办应用为例。

### 启动应用

按照以下步骤在本机运行该待办应用。

1. [下载并安装](https://www.docker.com/products/docker-desktop/) Docker Desktop。
2. 打开终端，[克隆示例应用](https://github.com/dockersamples/todo-list-app)。

    ```console
    git clone https://github.com/dockersamples/todo-list-app 
    ```

3. 进入 `todo-list-app` 目录：

    ```console
    cd todo-list-app
    ```

    在该目录下，你会看到名为 `compose.yaml` 的文件。这个 YAML 文件定义了组成应用的各个服务及其配置，包括镜像、端口、卷、网络等。建议花点时间浏览该文件，熟悉其结构。

4. 使用 [`docker compose up`](/reference/cli/docker/compose/up/) 命令启动应用：

    ```console
    docker compose up -d --build
    ```

    运行该命令后，你应能看到类似如下的输出：

    ```console
    [+] Running 5/5
    ✔ app 3 layers [⣿⣿⣿]      0B/0B            Pulled          7.1s
      ✔ e6f4e57cc59e Download complete                          0.9s
      ✔ df998480d81d Download complete                          1.0s
      ✔ 31e174fedd23 Download complete                          2.5s
      ✔ 43c47a581c29 Download complete                          2.0s
    [+] Running 4/4
      ⠸ Network todo-list-app_default           Created         0.3s
      ⠸ Volume "todo-list-app_todo-mysql-data"  Created         0.3s
      ✔ Container todo-list-app-app-1           Started         0.3s
      ✔ Container todo-list-app-mysql-1         Started         0.3s
    ```

    这里发生了很多事情！重点包括：

    - 从 Docker Hub 拉取了两个镜像：node 与 MySQL
    - 为你的应用创建了一个网络
    - 创建了一个卷，用于在容器重启之间持久化数据库文件
    - 启动了两个容器，并应用了相应配置

    如果这些信息一时有些多，不必担心！你很快就会熟悉起来！

5. 一切就绪后，在浏览器打开 [http://localhost:3000](http://localhost:3000) 查看站点。可随意添加、勾选或删除待办项。

    ![A screenshot of a webpage showing the todo-list application running on port 3000](images/todo-list-app.webp?border=true&w=950&h=400)

6. 在 Docker Desktop 图形界面中也能看到这些容器，并深入查看其配置。

    ![A screenshot of Docker Desktop dashboard showing the list of containers running todo-list app](images/todo-list-containers.webp?border=true&w=950&h=400)


### 清理环境

由于该应用是通过 Docker Compose 启动的，结束后清理也很简单。

1. 在命令行中运行 [`docker compose down`](/reference/cli/docker/compose/down/) 命令移除相关资源：

    ```console
    docker compose down
    ```

    你将看到类似如下输出：

    ```console
    [+] Running 3/3
    ✔ Container todo-list-app-mysql-1  Removed        2.9s
    ✔ Container todo-list-app-app-1    Removed        0.1s
    ✔ Network todo-list-app_default    Removed        0.1s
    ```

    > **卷的持久化**
    >
    > 默认情况下，当你清理 Compose 栈时，卷不会被自动删除。这样做是为了在下次重新启动时仍能保留数据。
    >
    > 如果确实需要删除卷，请在运行 `docker compose down` 时添加 `--volumes` 参数：
    >
    > ```console
    > docker compose down --volumes
    > [+] Running 1/0
    > ✔ Volume todo-list-app_todo-mysql-data  Removed
    > ```

2. 你也可以在 Docker Desktop GUI 中选择该应用栈并点击 **Delete** 来移除容器。

    ![A screenshot of the Docker Desktop GUI showing the containers view with an arrow pointing to the "Delete" button](images/todo-list-delete.webp?w=930&h=400)

    > **在 GUI 中管理 Compose 栈**
    >
    > 注意：在 GUI 中移除 Compose 应用的容器只会删除容器本身。如需删除网络或卷，需要手动操作。

在本次演练中，你学习了如何使用 Docker Compose 启动与停止一个多容器应用。


## 进一步阅读

本页是对 Compose 的简要介绍。可参考以下资源深入了解 Compose 及其文件编写方式。


* [Docker Compose 概览](/compose/)
* [Docker Compose CLI 概览](/compose/reference/)
* [Compose 工作原理](/compose/intro/compose-application-model/)
