---
description: 了解如何开始使用 Docker Hub
keywords: Docker Hub, 推送镜像, 拉取镜像, 仓库
title: Docker Hub 快速上手
linkTitle: 快速上手
weight: 10
---

Docker Hub 提供庞大的预构建镜像与资源库，能加速开发流程并减少环境搭建时间。你可以基于 Docker Hub 上的预构建镜像进行扩展，再通过仓库与团队或数以百万计的开发者分享和分发你自己的镜像。

本指南将演示如何查找并运行一个预构建镜像，随后带你创建自定义镜像并通过 Docker Hub 进行分享。

## 前提条件

- [下载并安装 Docker](../../get-started/get-docker.md)
- [创建 Docker 账号](https://app.docker.com/signup)

## Step 1: Find an image in Docker Hub's library

你可以直接在 Docker Hub、Docker Desktop 仪表板中搜索内容，或使用 CLI 进行搜索。

在 Docker Hub 上搜索或浏览内容：

{{< tabs >}}
{{< tab name="Docker Hub" >}}

1. Navigate to the [Docker Hub Explore page](https://hub.docker.com/explore).

   在 **Explore** 页面，你可以按目录或类别浏览，也可以使用搜索快速定位内容。

2. 在 **Categories** 下选择 **Web servers**。

   结果显示后，可使用页面左侧的筛选器进一步过滤结果。

3. 在筛选条件中选择 **Docker Official Image**。

   选择可信内容（Trusted Content）可确保只显示由 Docker 与经过验证的发布伙伴提供的高质量、安全镜像。

4. 在结果中选择 **nginx** 镜像。

   选择镜像后会进入其详情页，你可以了解镜像的使用方式；页面上也提供了用于拉取镜像的 `docker pull` 命令。

{{< /tab >}}
{{< tab name="Docker Desktop" >}}

1. 打开 Docker Desktop 仪表板。
2. 选择 **Docker Hub** 视图。

   在 **Docker Hub** 视图中，你可以按目录或类别浏览，也可以使用搜索快速定位内容。

3. 让搜索框保持为空，然后选择 **Search**。

   搜索结果会显示在界面中，同时搜索框旁会出现额外的筛选条件。

4. 点击搜索筛选图标，选择 **Docker Official Image** 与 **Web Servers**。
5. 在结果中选择 **nginx** 镜像。

{{< /tab >}}
{{< tab name="CLI" >}}

1. 打开一个终端窗口。

   > [!TIP]
   >
   > Docker Desktop 仪表板内置终端。你可以在仪表板底部选择 **>_ Terminal** 打开它。

2. 在终端中运行以下命令：

   ```console
   $ docker search --filter is-official=true nginx
   ```

   与 Docker Hub 和 Docker Desktop 的界面不同，`docker search` 命令不支持按类别浏览。更多用法参见 [docker search](/reference/cli/docker/search/)。

{{< /tab >}}
{{< /tabs >}}

找到镜像后，就可以在本地拉取并运行它了。

## Step 2: Pull and run an image from Docker Hub

你可以使用 CLI 或 Docker Desktop 仪表板运行来自 Docker Hub 的镜像。

{{< tabs >}}
{{< tab name="Docker Desktop" >}}

1. 在 Docker Desktop 仪表板的 **Docker Hub** 视图中选择 **nginx** 镜像。详情参见 [Step 1: Find an image in Docker Hub's library](#step-1-find-an-image-in-docker-hubs-library)。

2. 在 **nginx** 详情页，选择 **Run**。

   如果你的设备上还没有此镜像，将会自动从 Docker Hub 拉取。根据网络情况，拉取可能需要数秒到数分钟。镜像拉取完成后，Docker Desktop 会弹出窗口以便你指定运行选项。

3. 在 **Host port** 选项中填写 `8080`。
4. 选择 **Run**。

   容器启动后会显示容器日志。

5. 点击 **8080:80** 链接打开服务，或在浏览器访问 [http://localhost:8080](http://localhost:8080)。

6. 在 Docker Desktop 仪表板中点击 **Stop** 按钮停止容器。


{{< /tab >}}
{{< tab name="CLI" >}}

1. 打开一个终端窗口。

   > [!TIP]
   >
   > Docker Desktop 仪表板内置终端。你可以在仪表板底部选择 **>_ Terminal** 打开它。

2. 在终端中运行以下命令以拉取并运行 Nginx 镜像：

   ```console
   $ docker run -p 8080:80 --rm nginx
   ```

   `docker run` 会在本地不存在镜像时自动拉取并运行，无需先执行 `docker pull`。更多用法与选项参见 [`docker run` CLI 参考](../../reference/cli/docker/container/run.md)。运行后，你应当看到类似以下的输出：

   ```console {collapse=true}
   Unable to find image 'nginx:latest' locally
   latest: Pulling from library/nginx
   a480a496ba95: Pull complete
   f3ace1b8ce45: Pull complete
   11d6fdd0e8a7: Pull complete
   f1091da6fd5c: Pull complete
   40eea07b53d8: Pull complete
   6476794e50f4: Pull complete
   70850b3ec6b2: Pull complete
   Digest: sha256:28402db69fec7c17e179ea87882667f1e054391138f77ffaf0c3eb388efc3ffb
   Status: Downloaded newer image for nginx:latest
   /docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
   /docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
   /docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
   10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
   10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
   /docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
   /docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
   /docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
   /docker-entrypoint.sh: Configuration complete; ready for start up
   2024/11/07 21:43:41 [notice] 1#1: using the "epoll" event method
   2024/11/07 21:43:41 [notice] 1#1: nginx/1.27.2
   2024/11/07 21:43:41 [notice] 1#1: built by gcc 12.2.0 (Debian 12.2.0-14)
   2024/11/07 21:43:41 [notice] 1#1: OS: Linux 6.10.11-linuxkit
   2024/11/07 21:43:41 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
   2024/11/07 21:43:41 [notice] 1#1: start worker processes
   2024/11/07 21:43:41 [notice] 1#1: start worker process 29
   ...
   ```

3. 访问 [http://localhost:8080](http://localhost:8080) 查看默认的 Nginx 页面，验证容器已正常运行。

4. 在终端中按下 <kdb>Ctrl+C</kbd> 停止容器。

{{< /tab >}}
{{< /tabs >}}

至此，你已经在无需额外配置的情况下运行了一个 Web 服务器。Docker Hub 让你可以即时获取预构建、开箱即用的容器镜像，快速拉取并运行应用，而无需手动安装或配置软件。借助 Docker Hub 丰富的镜像库，你可以轻松试用新工具、搭建开发环境，或在现有软件之上进行构建，从而提升生产效率并简化部署。

你也可以基于 Docker Hub 的镜像进行扩展，快速构建并定制满足特定需求的自定义镜像。


## Step 3: Build and push an image to Docker Hub

1. 创建一个用于描述应用的 [Dockerfile](/reference/dockerfile.md)：

   ```dockerfile
   FROM nginx
   RUN echo "<h1>Hello world from Docker!</h1>" > /usr/share/nginx/html/index.html
   ```

   这个 Dockerfile 基于 Docker Hub 上的 Nginx 镜像扩展，创建了一个简单的网站。只需几行配置，你就能用 Docker 轻松搭建、定制并分享一个静态网站。

2. 运行以下命令构建镜像。将 `<YOUR-USERNAME>` 替换为你的 Docker ID：

   ```console
   $ docker build -t <YOUR-USERNAME>/nginx-custom .
   ```

   此命令会构建镜像并为其打标签，以便 Docker 知道应推送到 Docker Hub 的哪个仓库。更多用法与选项参见 [`docker build` CLI 参考](../../reference/cli/docker/buildx/build.md)。运行后，你应当看到类似以下的输出：

   ```console {collapse=true}
   [+] Building 0.6s (6/6) FINISHED                      docker:desktop-linux
    => [internal] load build definition from Dockerfile                  0.0s
    => => transferring dockerfile: 128B                                  0.0s
    => [internal] load metadata for docker.io/library/nginx:latest       0.0s
    => [internal] load .dockerignore                                     0.0s
    => => transferring context: 2B                                       0.0s
    => [1/2] FROM docker.io/library/nginx:latest                         0.1s
    => [2/2] RUN echo "<h1>Hello world from Docker!</h1>" > /usr/share/  0.2s
    => exporting to image                                                0.1s
    => => exporting layers                                               0.0s
    => => writing image sha256:f85ab68f4987847713e87a95c39009a5c9f4ad78  0.0s
    => => naming to docker.io/mobyismyname/nginx-custom                  0.0s
   ```

3. 运行以下命令测试镜像。将 `<YOUR-USERNAME>` 替换为你的 Docker ID：

   ```console
   $ docker run -p 8080:80 --rm <YOUR-USERNAME>/nginx-custom
   ```

4. 访问 [http://localhost:8080](http://localhost:8080) 查看页面，应能看到 `Hello world from Docker!`。

5. 在终端中按下 CTRL+C 停止容器。

6. 登录 Docker Desktop。推送镜像到 Docker Hub 之前必须先完成登录。

7. 运行以下命令将镜像推送到 Docker Hub。将 `<YOUR-USERNAME>` 替换为你的 Docker ID：

   ```console
   $ docker push <YOUR-USERNAME>/nginx-custom
   ```

    > [!NOTE]
    >
    > 你必须通过 Docker Desktop 或命令行登录 Docker Hub，并按上述步骤正确命名镜像。

   该命令会将镜像推送至 Docker Hub；若仓库尚不存在，将自动创建。更多用法参见 [`docker push` CLI 参考](../../reference/cli/docker/image/push.md)。运行后，你应当看到类似以下的输出：

   ```console {collapse=true}
   Using default tag: latest
   The push refers to repository [docker.io/mobyismyname/nginx-custom]
   d0e011850342: Pushed
   e4e9e9ad93c2: Mounted from library/nginx
   6ac729401225: Mounted from library/nginx
   8ce189049cb5: Mounted from library/nginx
   296af1bd2844: Mounted from library/nginx
   63d7ce983cd5: Mounted from library/nginx
   b33db0c3c3a8: Mounted from library/nginx
   98b5f35ea9d3: Mounted from library/nginx
   latest: digest: sha256:7f5223ae866e725a7f86b856c30edd3b86f60d76694df81d90b08918d8de1e3f size: 1985
   ```

  完成创建仓库并推送镜像后，你可以查看仓库并探索更多可用选项。

## Step 4: View your repository on Docker Hub and explore options

你可以在 Docker Hub 或 Docker Desktop 界面查看你的 Docker Hub 仓库。

{{< tabs >}}
{{< tab name="Docker Hub" >}}

1. 访问 [Docker Hub](https://hub.docker.com) 并登录。

   登录后通常会进入 **Repositories** 页面；如果没有，请前往 [**Repositories**](https://hub.docker.com/repositories/) 页面。

2. 找到 **nginx-custom** 仓库并点击该行。

   选择仓库后，你将看到更多详情与可用选项。

{{< /tab >}}
{{< tab name="Docker Desktop" >}}

1. 登录 Docker Desktop。
2. 选择 **Images** 视图。
3. 切换到 **Hub repositories** 选项卡。

   你会看到自己的 Docker Hub 仓库列表。

4. 找到 **nginx-custom** 仓库，将鼠标悬停在该行上，然后选择 **View in Hub**。

   将会打开 Docker Hub，你可以查看更多关于该镜像的详情。

{{< /tab >}}
{{< /tabs >}}

至此，你已经确认仓库存在于 Docker Hub，并了解了更多可用选项。继续查看后续步骤以进一步了解这些能力。

## 后续步骤

为仓库补充[仓库信息](./repos/manage/information.md)，以帮助其他用户更好地发现并使用你的镜像。
