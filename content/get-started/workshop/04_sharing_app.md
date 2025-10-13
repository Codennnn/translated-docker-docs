---
title: 分享应用
weight: 40
linkTitle: "第 3 部分：分享应用"
keywords: 入门, 安装, 上手, 快速开始, 概念, 容器,
  Docker Desktop, Docker Hub, 分享
description: 将为示例应用构建的镜像分享出去，便于在其他环境运行并供其他开发者使用
aliases:
 - /get-started/part3/
 - /get-started/04_sharing_app/
 - /guides/workshop/04_sharing_app/
---

现在你已经构建了镜像，可以将其分享出去。要分享 Docker 镜像，需要使用 Docker 仓库（registry）。默认的仓库是 Docker Hub，你此前使用的镜像都来自这里。

> **Docker ID**
>
> Docker ID 可用于访问 Docker Hub——全球最大的容器镜像库与社区。如果你还没有账号，可以免费创建一个 [Docker ID](https://hub.docker.com/signup)。

## 创建仓库

要推送镜像，你需要先在 Docker Hub 上创建一个仓库。

1. 在 [Docker Hub](https://hub.docker.com) 上 [注册](https://www.docker.com/pricing?utm_source=docker&utm_medium=webreferral&utm_campaign=docs_driven_upgrade) 或登录。

2. 选择 **Create Repository** 按钮。

3. 仓库名称填写 `getting-started`，并确保 **Visibility** 为 **Public**。

4. 点击 **Create** 创建仓库。

如下图所示，Docker Hub 会给出一个示例 Docker 命令，用于推送到该仓库。

![Docker command with push example](images/push-command.webp)


## 推送镜像

现在试着将镜像推送到 Docker Hub。

1. 在命令行中运行：

   ```console
   docker push docker/getting-started
   ```

   你会看到类似如下的错误：

   ```console
   $ docker push docker/getting-started
   The push refers to repository [docker.io/docker/getting-started]
   An image does not exist locally with the tag: docker/getting-started
   ```

   这是预期之中的失败，因为镜像的标签尚未正确设置。Docker 在查找名为 `docker/getting-started` 的镜像，但你本地的镜像仍然叫做 `getting-started`。

   你可以通过以下命令确认：

   ```console
   docker image ls
   ```

2. 解决方法：先使用你的 Docker ID 登录 Docker Hub：`docker login YOUR-USER-NAME`。
3. 使用 `docker tag` 命令为 `getting-started` 镜像添加新名称。将 `YOUR-USER-NAME` 替换为你的 Docker ID：

   ```console
   $ docker tag getting-started YOUR-USER-NAME/getting-started
   ```

4. 现在再次运行 `docker push` 命令。如果你从 Docker Hub 复制的名称包含 `tagname`，可以省略它，因为我们并未为镜像添加额外的标签。未指定标签时，Docker 会默认使用 `latest`。

   ```console
   $ docker push YOUR-USER-NAME/getting-started
   ```

## 在全新实例上运行镜像

现在镜像已构建并推送到仓库，试着在一台从未拉取过该镜像的全新实例上运行应用。这里我们使用 Play with Docker。

> [!NOTE]
>
> Play with Docker 使用 amd64 平台。如果你使用的是基于 ARM 的 Apple 芯片 Mac，需要为 amd64 平台重新构建镜像并推送到你的仓库。
>
> 要为 amd64 平台构建镜像，可使用 `--platform` 参数：
> ```console
> $ docker build --platform linux/amd64 -t YOUR-USER-NAME/getting-started .
> ```
>
> Docker buildx 也支持构建多平台镜像。了解更多，请参阅 [Multi-platform images](/manuals/build/building/multi-platform.md)。


1. 在浏览器打开 [Play with Docker](https://labs.play-with-docker.com/)。

2. 选择 **Login**，在下拉列表中选择 **docker**。

3. 使用 Docker Hub 账号登录，然后点击 **Start**。

4. 在左侧边栏选择 **ADD NEW INSTANCE**。如果没有看到该选项，可以适当加宽浏览器窗口。几秒钟后，浏览器中会打开一个终端窗口。

    ![Play with Docker add new instance](images/pwd-add-new-instance.webp)

5. 在终端中启动你刚刚推送的应用：

   ```console
   $ docker run -dp 0.0.0.0:3000:3000 YOUR-USER-NAME/getting-started
   ```

    你应当会看到镜像被拉取并最终启动。

    > [!TIP]
    >
    > 你可能注意到该命令绑定端口映射到不同的 IP。之前的 `docker run` 命令将端口发布到宿主机的 `127.0.0.1:3000`，这次我们使用的是 `0.0.0.0`。
    >
    > 绑定到 `127.0.0.1` 仅将容器端口暴露给回环接口；而绑定到 `0.0.0.0` 会在宿主机的所有网络接口上暴露容器端口，使其能被外部访问。
    >
    > 关于端口映射的更多信息，请参阅 [Networking](/manuals/engine/network/_index.md#published-ports)。

6. 当界面出现 3000 的徽章（badge）时点击它。

   如果没有出现 3000 徽章，可以选择 **Open Port** 并指定 `3000`。

## 小结

本节你学习了如何通过推送到镜像仓库来分享镜像。随后你在一台全新实例上运行了刚刚推送的镜像。这与 CI 流水线中的常见做法类似：流水线创建镜像并将其推送到仓库，生产环境再使用该镜像的最新版本。

相关信息：

 - [docker CLI reference](/reference/cli/docker/)
 - [Multi-platform images](/manuals/build/building/multi-platform.md)
 - [Docker Hub overview](/manuals/docker-hub/_index.md)

## 下一步

下一节你将学习如何在容器化应用中持久化数据。

{{< button text="持久化数据库" url="05_persisting_data.md" >}}
