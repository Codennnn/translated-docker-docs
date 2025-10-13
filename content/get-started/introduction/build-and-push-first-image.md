---
title: 构建并推送你的第一个镜像
keywords: concepts, container, docker desktop
description: 本页将教你如何构建并推送你的第一个镜像。
summary: |
  学习如何构建你的第一个 Docker 镜像，这是让应用走向容器化的关键一步。
  我们将引导你创建镜像仓库，并将镜像构建并推送到 Docker Hub，
  让你可以在团队内轻松共享镜像。
weight: 3
aliases: 
 - /guides/getting-started/build-and-push-first-image/
---

{{< youtube-embed 7ge1s5nAa34 >}}

## 说明

现在你已经更新了[待办应用](develop-with-containers.md)，可以为该应用创建一个容器镜像并将其分享至 Docker Hub。为此，你需要完成以下步骤：

1. 使用你的 Docker 账号登录
2. 在 Docker Hub 上创建镜像仓库
3. 构建容器镜像
4. 将镜像推送到 Docker Hub

在开始动手之前，先了解几个核心概念。

### 容器镜像

如果你刚接触容器镜像，可以把它看作一个标准化的软件包，包含运行应用所需的一切：文件、配置与依赖。镜像可以被分发并与他人共享。

### Docker Hub

要分享你的 Docker 镜像，你需要一个存放它们的地方，即“仓库”（registry）。虽然有许多可用的仓库，但 Docker Hub 是默认且最常用的镜像仓库。Docker Hub 既是你存放自有镜像的地方，也可以在其中查找他人的镜像，用来直接运行或作为你的镜像基础。

在[使用容器进行开发](develop-with-containers.md)中，你使用了来自 Docker Hub 的以下镜像，它们均为[Docker 官方镜像](/manuals/docker-hub/image-library/trusted-content.md#docker-official-images)：

- [node](https://hub.docker.com/_/node) - 提供 Node 运行环境，作为开发阶段的基础镜像，同时也作为最终应用镜像的基础。
- [mysql](https://hub.docker.com/_/mysql) - 提供用于存放待办事项的 MySQL 数据库
- [phpmyadmin](https://hub.docker.com/_/phpmyadmin) - 提供 phpMyAdmin，一个基于 Web 的 MySQL 数据库管理界面
- [traefik](https://hub.docker.com/_/traefik) - 提供 Traefik，一款现代化的 HTTP 反向代理与负载均衡器，可基于路由规则将请求转发至目标容器

前往查看完整目录：[Docker 官方镜像](https://hub.docker.com/search?image_filter=official&q=)、[Docker 验证发布者](https://hub.docker.com/search?q=&image_filter=store)与 [Docker 赞助的开源软件](https://hub.docker.com/search?q=&image_filter=open_source)，探索更多可运行与可构建的内容。

## 试一试

在这个动手指南中，你将学习如何登录 Docker Hub，并将镜像推送到 Docker Hub 仓库。

## 使用 Docker 账号登录

要将镜像推送到 Docker Hub，你需要先使用 Docker 账号登录。

1. 打开 Docker Dashboard。

2. 点击右上角的 **Sign in**。

3. 如有需要，先创建账号，再完成登录流程。

完成后，你将看到 **Sign in** 按钮变为头像。

## 创建镜像仓库

现在你已有账号，可以创建镜像仓库。就像 Git 仓库存放源代码一样，镜像仓库存放容器镜像。

1. 访问 [Docker Hub](https://hub.docker.com)。

2. 点击 **Create repository**。

3. 在 **Create repository** 页面填写以下信息：

    - **Repository name** - `getting-started-todo-app`
    - **Short description** - 可按需填写描述
    - **Visibility** - 选择 **Public** 允许他人拉取你定制的待办应用镜像

4. 点击 **Create** 创建仓库。


## 构建并推送镜像

有了仓库之后，就可以构建并推送镜像了。需要注意的是，你要构建的镜像基于 Node 镜像，这意味着你无需手动安装或配置 Node、Yarn 等依赖，可将精力集中在应用本身。

> **什么是镜像 / Dockerfile？**
>
> 简单来说，一个容器镜像就是包含运行某个进程所需一切内容的单一软件包。
> 在此示例中，它包含 Node 运行环境、后端代码以及编译后的 React 代码。
>
> 任何使用该镜像运行容器的机器，都可以按构建时的方式运行应用，
> 而无需在宿主机上预先安装其他组件。
>
> `Dockerfile` 是一个文本脚本，提供构建镜像所需的指令集合。
> 在本快速入门示例中，仓库已包含该 Dockerfile。


{{< tabs group="cli-or-vs-code" persist=true >}}
{{< tab name="CLI" >}}

1. 首先，将项目克隆到本地，或[下载 ZIP 包](https://github.com/docker/getting-started-todo-app/archive/refs/heads/main.zip)。

   ```console
   $ git clone https://github.com/docker/getting-started-todo-app
   ```

   克隆完成后，进入项目目录：

   ```console
   $ cd getting-started-todo-app
   ```

2. 运行如下命令构建项目，将 `DOCKER_USERNAME` 替换为你的用户名。

    ```console
    $ docker build -t <DOCKER_USERNAME>/getting-started-todo-app .
    ```

    例如，若你的 Docker 用户名为 `mobydock`，则运行：

    ```console
    $ docker build -t mobydock/getting-started-todo-app .
    ```

3. 使用 `docker image ls` 命令验证镜像是否已存在于本地：

    ```console
    $ docker image ls
    ```

    你将看到类似如下输出：

    ```console
    REPOSITORY                          TAG       IMAGE ID       CREATED          SIZE
    mobydock/getting-started-todo-app   latest    1543656c9290   2 minutes ago    1.12GB
    ...
    ```

4. 使用 `docker push` 推送镜像。请将 `DOCKER_USERNAME` 替换为你的用户名：

    ```console
    $ docker push <DOCKER_USERNAME>/getting-started-todo-app
    ```

    推送时间取决于你的上传带宽，可能需要片刻。

{{< /tab >}}
{{< tab name="VS Code" >}}

1. 打开 Visual Studio Code。请从[扩展市场](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-docker)安装 **Docker 扩展**。

   ![Screenshot of VS code extension marketplace](images/install-docker-extension.webp)

2. 在 **File** 菜单中选择 **Open Folder**。选择 **Clone Git Repository** 并粘贴以下 URL： [https://github.com/docker/getting-started-todo-app](https://github.com/docker/getting-started-todo-app)

    ![Screenshot of VS code showing how to clone a repository](images/clone-the-repo.webp?border=true)



3. 右键点击 `Dockerfile`，选择 **Build Image...**。


    ![Screenshot of VS Code showing the right-click menu and "Build Image" menu item](images/build-vscode-menu-item.webp?border=true)

4. 在弹出的对话框中，输入名称 `DOCKER_USERNAME/getting-started-todo-app`，将 `DOCKER_USERNAME` 替换为你的 Docker 用户名。 

5. 按下 **Enter** 后，将出现一个终端用于执行构建。构建完成后可关闭该终端。

6. 点击左侧导航栏中的 Docker 图标，打开 VS Code 的 Docker 扩展。

7. 找到你刚构建的镜像，名称为 `docker.io/DOCKER_USERNAME/getting-started-todo-app`。 

8. 展开该镜像以查看标签（不同版本）。你应能看到名为 `latest` 的标签，它是镜像的默认标签。

9. 右键点击 **latest** 条目，选择 **Push...**。

    ![Screenshot of the Docker Extension and the right-click menu to push an image](images/build-vscode-push-image.webp)

10. 按 **Enter** 确认，观察镜像被推送至 Docker Hub。推送时间取决于你的上传带宽。

    上传完成后，可关闭终端。

{{< /tab >}}
{{< /tabs >}}


## 回顾

在继续之前，花点时间回顾一下你完成了什么：短时间内，你已经构建了打包应用的容器镜像，并将其推送到了 Docker Hub。

接下来，请记住：

- Docker Hub 是查找可信内容的首选仓库。Docker 提供了由 Docker 官方镜像、Docker 验证发布者以及 Docker 赞助开源软件构成的可信内容集合，可直接使用，或作为你自有镜像的基础。

- Docker Hub 也可作为分发你自有应用的“应用市场”。任何人都可以创建账号并分发镜像。除了公开分发之外，你也可以使用私有仓库来确保只有授权用户可以访问你的镜像。

> **关于使用其他仓库**
>
> 虽然 Docker Hub 是默认仓库，但在 [Open Container Initiative](https://opencontainers.org/) 的推动下，
> 各类仓库已实现标准化与互操作性。这使企业和组织可以运行各自的私有仓库。
> 在很多情况下，可信内容会从 Docker Hub 镜像（或复制）到这些私有仓库中。
>



## 下一步

现在你已经构建了一个镜像，接下来我们来讨论，作为开发者为何应当进一步学习 Docker，以及它如何帮助你完成日常工作。

{{< button text="下一步" url="whats-next" >}}


