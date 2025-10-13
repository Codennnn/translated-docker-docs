---
title: 什么是容器？
weight: 10
keywords: concepts, build, images, container, docker desktop
description: 什么是容器？本页将介绍容器的核心概念，并通过一个快速上手示例带你运行第一个容器。
aliases:
- /guides/walkthroughs/what-is-a-container/
- /guides/walkthroughs/run-a-container/
- /guides/walkthroughs/
- /get-started/run-your-own-container/
- /get-started/what-is-a-container/
- /guides/docker-concepts/the-basics/what-is-a-container/
---

{{< youtube-embed W1kWqFkiu7k >}}

## 说明

想象你要开发一个出色的 Web 应用，它包含三个主要组件：React 前端、Python API 与 PostgreSQL 数据库。要在本机上开展该项目，你通常需要安装 Node、Python 与 PostgreSQL。

如何确保你与团队其他开发者、CI/CD 系统以及生产环境使用的是相同版本？

又如何保证应用所需的 Python（或 Node、数据库）版本不受本机既有环境影响？如何管理潜在冲突？

容器应运而生！

什么是容器？简单来说，容器是为应用各组件提供隔离的进程。每个组件——React 前端、Python API 引擎、数据库——都在独立环境中运行，与宿主机上的其他内容完全隔离。

容器之所以强大，在于它们具备以下特性：

- 自包含：每个容器都包含运行所需的一切，不依赖宿主机预装依赖。
- 隔离性：容器彼此隔离，对宿主机与其他容器的影响最小，有助于提升应用安全性。
- 独立性：每个容器独立管理，删除一个容器不会影响其他容器。
- 可移植：容器可在任意环境运行！在你开发机上能运行的容器，在数据中心或云端也能以相同方式运行。

### 容器与虚拟机（VM）的对比

不做过多展开，虚拟机是一个完整的操作系统，包含自有内核、硬件驱动、程序与应用。仅为隔离单个应用而启动虚拟机，开销较大。

容器本质上是一个带有运行所需文件的隔离进程。即使运行多个容器，它们也共享同一内核，从而在更少的基础设施上运行更多应用。

> **同时使用 VM 与容器**
>
> 在许多场景中，容器与 VM 会结合使用。例如，在云环境中，所提供的计算实例通常是 VM。
> 相比“一机一应用”，在 VM 中运行容器运行时（runtime）可以同时运行多个容器化应用，提升资源利用率并降低成本。


## 试一试

在本示例中，你将通过 Docker Desktop 图形界面运行一个 Docker 容器。

{{< tabs group=concept-usage persist=true >}}
{{< tab name="使用图形界面（GUI）" >}}

按照以下步骤运行一个容器：

1. 打开 Docker Desktop，在顶部导航栏选择 **Search**。

2. 在搜索框输入 `welcome-to-docker`，然后点击 **Pull**。

    ![A screenshot of the Docker Desktop Dashboard showing the search result for welcome-to-docker Docker image ](images/search-the-docker-image.webp?border=true&w=1000&h=700)

3. 镜像拉取成功后，点击 **Run**。

4. 展开 **Optional settings**。

5. 在 **Container name** 中填写 `welcome-to-docker`。

6. 在 **Host port** 中填写 `8080`。

    ![A screenshot of Docker Desktop Dashboard showing the container run dialog with welcome-to-docker typed in as the container name and 8080 specified as the port number](images/run-a-new-container.webp?border=true&w=550&h=400)

7. 点击 **Run** 启动容器。

恭喜你！你已经运行了第一个容器！🎉
 
### 查看你的容器

在 Docker Desktop 的 **Containers** 视图中，你可以查看所有容器。

![Screenshot of the container view of the Docker Desktop GUI showing the welcome-to-docker container running on the host port 8080](images/view-your-containers.webp?border=true&w=750&h=600)

该容器运行了一个 Web 服务器，用于展示一个简单的网站。在更复杂的项目中，你会将不同部分分别运行在不同的容器中，例如前端、后端与数据库使用不同容器。

### 访问前端

在启动容器时，你将容器的某个端口映射到本机。可以把它理解为一项连通配置，让你能够穿透容器的隔离环境进行访问。

对于该容器，前端通过 `8080` 端口对外提供访问。打开方式：在容器的 **Port(s)** 列中点击链接，或在浏览器访问 [http://localhost:8080](http://localhost:8080)。

![Screenshot of the landing page coming from the running container](images/access-the-frontend.webp?border)

### 探索你的容器

Docker Desktop 允许你探索并与容器的不同方面进行交互。你可以试试： 

1. 在 Docker Desktop 中打开 **Containers** 视图。

2. 选择你的容器。

3. 选择 **Files** 选项卡，浏览容器的隔离文件系统。

    ![Screenshot of the Docker Desktop Dashboard showing the files and directories inside a running container](images/explore-your-container.webp?border)

### 停止容器

`docker/welcome-to-docker` 容器会一直运行，直到你将其停止。 

1. Go to the **Containers** view in the Docker Desktop Dashboard.

2. 找到你要停止的容器。

3. 在 **Actions** 列选择 **Stop** 操作。

    ![Screenshot of the Docker Desktop Dashboard with the welcome container selected and being prepared to stop](images/stop-your-container.webp?border)

{{< /tab >}}
{{< tab name="使用命令行（CLI）" >}}

按照以下步骤使用 CLI 运行容器：

1. 打开命令行终端，使用 [`docker run`](/reference/cli/docker/container/run/) 命令启动容器：

    ```console
    $ docker run -d -p 8080:80 docker/welcome-to-docker
    ```

    命令输出的是完整的容器 ID。 

恭喜你！你已经启动了第一个容器！🎉

### 查看正在运行的容器

使用 [`docker ps`](/reference/cli/docker/container/ls/) 命令验证容器是否正在运行：

```console
docker ps
```

你将看到类似如下输出：

```console
 CONTAINER ID   IMAGE                      COMMAND                  CREATED          STATUS          PORTS                      NAMES
 a1f7a4bb3a27   docker/welcome-to-docker   "/docker-entrypoint.…"   11 seconds ago   Up 11 seconds   0.0.0.0:8080->80/tcp       gracious_keldysh
```

该容器运行了一个 Web 服务器，展示一个简单的网站。在更复杂的项目中，你会将不同部分分别运行在不同的容器中，例如 `frontend`、`backend` 与 `database` 各用一个容器。

> [!TIP]
>
> `docker ps` 只会显示正在运行的容器。若需查看已停止的容器，请添加 `-a` 参数列出全部容器：`docker ps -a`


### 访问前端

在启动容器时，你将容器的某个端口映射到本机。可以把它理解为一项连通配置，让你能够穿透容器的隔离环境进行访问。

对于该容器，前端通过 `8080` 端口对外提供访问。打开方式：在容器的 **Port(s)** 列中点击链接，或在浏览器访问 [http://localhost:8080](http://localhost:8080)。

![Screenshot of the landing page of the Nginx web server, coming from the running container](images/access-the-frontend.webp?border)

### 停止容器

`docker/welcome-to-docker` 容器会一直运行，直到你停止它。你可以使用 `docker stop` 命令停止容器。

1. 运行 `docker ps` 获取容器 ID

2. 将容器 ID 或名称传给 [`docker stop`](/reference/cli/docker/container/stop/) 命令：

    ```console
    docker stop <the-container-id>
    ```

> [!TIP]
>
> 通过 ID 引用容器时，无需提供完整 ID，只需提供足以唯一标识的前缀。例如，上述容器可以通过以下命令停止：
>
> ```console
> docker stop a1f
> ```

{{< /tab >}}
{{< /tabs >}}

## 进一步阅读

以下链接提供关于容器的更多参考资料：

- [运行容器](/engine/containers/run/)
- [容器概览](https://www.docker.com/resources/what-container/)
- [为什么选择 Docker？](https://www.docker.com/why-docker/)

## 下一步

既然你已经了解了 Docker 容器的基础知识，接下来是学习 Docker 镜像的时候了。

{{< button text="什么是镜像？" url="what-is-an-image" >}}
