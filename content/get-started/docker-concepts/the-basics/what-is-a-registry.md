---
title: 什么是仓库？
weight: 30
keywords: concepts, build, images, container, docker desktop
description: 什么是仓库？本文将解释容器镜像仓库的概念、互操作性，并带你与仓库进行实际交互。
aliases:
- /guides/walkthroughs/run-hub-images/
- /guides/walkthroughs/publish-your-image/
- /guides/docker-concepts/the-basics/what-is-a-registry/
---

{{< youtube-embed 2WDl10Wv5rs >}}

## 说明

既然你已经了解了容器镜像及其工作方式，接下来问题来了：这些镜像应该存在哪里？ 

当然，你可以把容器镜像存放在本机。但如果你想与他人共享，或在另一台机器上使用呢？这就需要“镜像仓库”（registry）。

镜像仓库是一个集中化的位置，用于存储与共享容器镜像。它可以是公开的，也可以是私有的。[Docker Hub](https://hub.docker.com) 是人人可用的公共仓库，同时也是默认仓库。

除了 Docker Hub，如今还有许多可用的容器镜像仓库，例如 [Amazon Elastic Container Registry (ECR)](https://aws.amazon.com/ecr/)、[Azure Container Registry (ACR)](https://azure.microsoft.com/en-in/products/container-registry)、[Google Container Registry (GCR)](https://cloud.google.com/artifact-registry) 等。你也可以在本地或组织内部部署私有仓库，比如 Harbor、JFrog Artifactory、GitLab Container Registry 等。

### 仓库（registry）与镜像仓库（repository）

在使用相关术语时，常会把 “registry” 与 “repository” 混用。二者相关，但并不相同。

“registry” 是存放与管理容器镜像的集中位置；而“repository” 则是位于 registry 内、按项目组织的一组相关镜像集合。可将其理解为“按项目分类的文件夹”，一个 repository 内包含一个或多个容器镜像。

下图展示了 registry、repository 与镜像之间的关系。

```goat {class="text-sm"}
+---------------------------------------+
|               Registry                |
|---------------------------------------|
|                                       |
|    +-----------------------------+    |
|    |        Repository A         |    |
|    |-----------------------------|    |
|    |   Image: project-a:v1.0     |    |
|    |   Image: project-a:v2.0     |    |
|    +-----------------------------+    |
|                                       |
|    +-----------------------------+    |
|    |        Repository B         |    |
|    |-----------------------------|    |
|    |   Image: project-b:v1.0     |    |
|    |   Image: project-b:v1.1     |    |
|    |   Image: project-b:v2.0     |    |
|    +-----------------------------+    |
|                                       |
+---------------------------------------+
```

> [!NOTE]
>
> 使用 Docker Hub 免费版，你可以创建 1 个私有仓库与不限数量的公共仓库。更多信息参见 [Docker Hub 订阅页面](https://www.docker.com/pricing/)。

## 试一试

在本示例中，你将学习如何构建并将 Docker 镜像推送到 Docker Hub 上的镜像仓库。

### 注册免费的 Docker 账号

1. 如果你还没有账号，请前往 [Docker Hub](https://hub.docker.com) 注册。

    ![Docker Hub 注册页面截图](images/dockerhub-signup.webp?border)

    你可以使用 Google 或 GitHub 账户进行身份验证。

### 创建你的第一个镜像仓库

1. 登录 [Docker Hub](https://hub.docker.com)。
2. 点击右上角 **Create repository**。
3. 选择你的命名空间（通常为你的用户名），将仓库名设置为 `docker-quickstart`。

    ![在 Docker Hub 上创建公共仓库的页面截图](images/create-hub-repository.webp?border)

4. 将可见性设置为 **Public**。 
5. 点击 **Create** 创建仓库。

完成！你已经成功创建了第一个镜像仓库。🎉

目前该仓库还是空的。接下来我们将向其推送一个镜像。

### 使用 Docker Desktop 登录

1. 如果尚未安装，请先[下载并安装](https://www.docker.com/products/docker-desktop/) Docker Desktop。
2. 在 Docker Desktop 图形界面右上角点击 **Sign in**。

### 克隆示例 Node.js 代码

要创建镜像，首先需要一个项目。为快速上手，我们将使用位于 [github.com/dockersamples/helloworld-demo-node](https://github.com/dockersamples/helloworld-demo-node) 的示例 Node.js 项目。该仓库包含构建 Docker 镜像所需的 Dockerfile。

无需关心 Dockerfile 的细节，后续章节会详细讲解。

1. 使用以下命令克隆该 GitHub 仓库：

    ```console
    git clone https://github.com/dockersamples/helloworld-demo-node
    ```

2. 进入新创建的目录。

    ```console
    cd helloworld-demo-node
    ```

3. 运行以下命令构建 Docker 镜像，将 `YOUR_DOCKER_USERNAME` 替换为你的用户名。

    ```console
    docker build -t <YOUR_DOCKER_USERNAME>/docker-quickstart .
    ```

    > [!NOTE]
    >
    > 请确保在 `docker build` 命令末尾包含点号（.），用于指示 Docker 在当前目录查找 Dockerfile。

4. 运行以下命令列出新创建的 Docker 镜像：

    ```console
    docker images
    ```

    你将看到类似如下输出：

    ```console
    REPOSITORY                                 TAG       IMAGE ID       CREATED         SIZE
    <YOUR_DOCKER_USERNAME>/docker-quickstart   latest    476de364f70e   2 minutes ago   170MB
    ```

5. 运行以下命令启动容器以测试镜像（将用户名替换为你自己的用户名）：

    ```console
    docker run -d -p 8080:8080 <YOUR_DOCKER_USERNAME>/docker-quickstart 
    ```

    在浏览器访问 [http://localhost:8080](http://localhost:8080) 验证容器是否运行正常。

6. 使用 [`docker tag`](/reference/cli/docker/image/tag/) 命令为镜像打标签。标签用于为镜像做标识与版本化。 

    ```console 
    docker tag <YOUR_DOCKER_USERNAME>/docker-quickstart <YOUR_DOCKER_USERNAME>/docker-quickstart:1.0 
    ```

7. 最后，使用 [`docker push`](/reference/cli/docker/image/push/) 命令将新构建的镜像推送到你的 Docker Hub 仓库：

    ```console 
    docker push <YOUR_DOCKER_USERNAME>/docker-quickstart:1.0
    ```

8. 打开 [Docker Hub](https://hub.docker.com) 并进入你的仓库，切换到 **Tags** 标签页查看刚推送的镜像。

    ![Docker Hub 显示新推送的镜像标签的页面截图](images/dockerhub-tags.webp?border=true) 

在本次演练中，你注册了 Docker 账号、创建了第一个 Docker Hub 仓库，并完成了镜像的构建、打标签与推送。

## 进一步阅读

- [Docker Hub 快速入门](/docker-hub/quickstart/)
- [管理 Docker Hub 仓库](/docker-hub/repos/)

## 下一步

既然你已了解容器与镜像的基础知识，现在可以继续学习 Docker Compose 了。

{{< button text="什么是 Docker Compose？" url="what-is-Docker-Compose" >}}
