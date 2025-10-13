---
title: 构建、打标签并发布镜像
keywords: concepts, build, images, container, docker desktop
description: 本文将介绍如何构建、打标签并将镜像发布到 Docker Hub 或其他任意仓库。
summary: |
  构建、打标签与发布 Docker 镜像是容器化工作流中的关键步骤。
  本指南将带你学习如何创建 Docker 镜像、如何为镜像添加可识别的标签，
  以及如何将镜像发布到公共仓库。
weight: 3
aliases: 
 - /guides/docker-concepts/building-images/build-tag-and-publish-an-image/
---

{{< youtube-embed chiiGLlYRlY >}}

## 概念解析

在本指南中，你将学习：

- 构建镜像：基于 `Dockerfile` 构建镜像的流程
- 为镜像打标签：给镜像一个可读的名称，并决定镜像的分发位置
- 发布镜像：使用容器仓库分发或共享新创建的镜像

### 构建镜像

大多数情况下，镜像通过 Dockerfile 构建。最基础的 `docker build` 命令如下：

```bash
docker build .
```

命令末尾的 `.` 指定了[构建上下文](https://docs.docker.com/build/concepts/context/#what-is-a-build-context) 的路径或 URL。构建器会在该位置找到 `Dockerfile` 以及其他被引用的文件。

运行构建时，构建器会在需要时拉取基础镜像，然后按 Dockerfile 中的指令逐步执行。

使用上面的命令构建出的镜像默认没有名称，但输出会包含镜像的 ID。例如，可能会看到如下输出：

```console
$ docker build .
[+] Building 3.5s (11/11) FINISHED                                              docker:desktop-linux
 => [internal] load build definition from Dockerfile                                            0.0s
 => => transferring dockerfile: 308B                                                            0.0s
 => [internal] load metadata for docker.io/library/python:3.12                                  0.0s
 => [internal] load .dockerignore                                                               0.0s
 => => transferring context: 2B                                                                 0.0s
 => [1/6] FROM docker.io/library/python:3.12                                                    0.0s
 => [internal] load build context                                                               0.0s
 => => transferring context: 123B                                                               0.0s
 => [2/6] WORKDIR /usr/local/app                                                                0.0s
 => [3/6] RUN useradd app                                                                       0.1s
 => [4/6] COPY ./requirements.txt ./requirements.txt                                            0.0s
 => [5/6] RUN pip install --no-cache-dir --upgrade -r requirements.txt                          3.2s
 => [6/6] COPY ./app ./app                                                                      0.0s
 => exporting to image                                                                          0.1s
 => => exporting layers                                                                         0.1s
 => => writing image sha256:9924dfd9350407b3df01d1a0e1033b1e543523ce7d5d5e2c83a724480ebe8f00    0.0s
```

根据上述输出，你可以通过引用镜像 ID 来启动容器：

```console
docker run sha256:9924dfd9350407b3df01d1a0e1033b1e543523ce7d5d5e2c83a724480ebe8f00
```

这个名称显然不利于记忆，这正是“打标签”的用武之地。

### 为镜像打标签

给镜像打标签的方式可以为其赋予一个可记忆的名称。需要注意的是，镜像名称有其结构。完整的镜像名称由以下部分构成：

```text
[HOST[:PORT_NUMBER]/]PATH[:TAG]
```

- `HOST`：可选，镜像所在的仓库主机名。如果未指定，默认使用 Docker 公共仓库 `docker.io`。
- `PORT_NUMBER`：当提供主机名时的仓库端口号。
- `PATH`：镜像路径，由斜杠分隔的多个部分构成。对 Docker Hub 而言，格式为 `[NAMESPACE/]REPOSITORY`，其中 `NAMESPACE` 是用户或组织名。如未指定 `NAMESPACE`，将使用 `library`，即 Docker 官方镜像的命名空间。
- `TAG`：自定义、便于识别的人类可读标识，常用于区分镜像的版本或变体。如未指定，默认使用 `latest`。

示例镜像名：

- `nginx` 等价于 `docker.io/library/nginx:latest`：从 `docker.io` 仓库的 `library` 命名空间拉取 `nginx` 仓库的 `latest` 标签。
- `docker/welcome-to-docker` 等价于 `docker.io/docker/welcome-to-docker:latest`：从 `docker.io` 仓库的 `docker` 命名空间拉取 `welcome-to-docker` 仓库的 `latest` 标签。
- `ghcr.io/dockersamples/example-voting-app-vote:pr-311`：从 GitHub Container Registry 的 `dockersamples` 命名空间拉取 `example-voting-app-vote` 仓库的 `pr-311` 标签。

在构建时为镜像打标签，使用 `-t` 或 `--tag` 参数：

```console
docker build -t my-username/my-image .
```

如果镜像已构建完成，你也可以使用 [`docker image tag`](https://docs.docker.com/engine/reference/commandline/image_tag/) 命令为其添加另一个标签：

```console
docker image tag my-username/my-image another-username/another-image:v1
```

### 发布镜像

当镜像已构建并打好标签，就可以将其推送到仓库。使用 [`docker push`](https://docs.docker.com/engine/reference/commandline/image_push/) 命令：

```console
docker push my-username/my-image
```

几秒钟后，镜像的所有层就会被推送到仓库中。

> **需要认证**
>
> 在将镜像推送到某个仓库之前，需要先完成认证。
> 可使用 [docker login](https://docs.docker.com/engine/reference/commandline/login/) 命令登录。
{ .information }

## 动手试试

本实践将使用给定的 Dockerfile 构建一个简单镜像，并将其推送到 Docker Hub。

### 准备环境

1. 获取示例应用。

   如果你安装了 Git，可以直接克隆示例应用仓库；否则也可以下载示例应用。选择以下任一方式。

   {{< tabs >}}
   {{< tab name="使用 git 克隆" >}}

   在终端中执行以下命令克隆示例应用仓库：

   ```console
   $ git clone https://github.com/docker/getting-started-todo-app
   ```
   {{< /tab >}}
   {{< tab name="下载" >}}

   下载源码并解压。

   {{< button url="https://github.com/docker/getting-started-todo-app/raw/cd61f824da7a614a8298db503eed6630eeee33a3/app.zip" text="下载源码" >}}

   {{< /tab >}}
   {{< /tabs >}}


2. [下载并安装](https://www.docker.com/products/docker-desktop/) Docker Desktop。

3. 如果你还没有 Docker 账号，请先[创建一个](https://hub.docker.com/)。创建后，使用该账号登录 Docker Desktop。

### 构建镜像

现在你已经在 Docker Hub 上拥有一个仓库，可以开始构建镜像并推送到该仓库。

1. 在示例应用仓库根目录打开终端，运行以下命令。将 `YOUR_DOCKER_USERNAME` 替换为你的 Docker Hub 用户名：

    ```console
    $ docker build -t <YOUR_DOCKER_USERNAME>/concepts-build-image-demo .
    ```

    例如，如果你的用户名是 `mobywhale`，则运行：

    ```console
    $ docker build -t mobywhale/concepts-build-image-demo .
    ```

2. 构建完成后，使用以下命令查看镜像：

    ```console
    $ docker image ls
    ```

    该命令会输出类似如下内容：

    ```plaintext
    REPOSITORY                             TAG       IMAGE ID       CREATED          SIZE
    mobywhale/concepts-build-image-demo    latest    746c7e06537f   24 seconds ago   354MB
    ```

3. 你还可以使用 [docker image history](/reference/cli/docker/image/history/) 查看镜像历史（即镜像是如何被构建的）：

    ```console
    $ docker image history mobywhale/concepts-build-image-demo
    ```

    输出类似如下：

    ```plaintext
    IMAGE          CREATED         CREATED BY                                      SIZE      COMMENT
    f279389d5f01   8 seconds ago   CMD ["node" "./src/index.js"]                   0B        buildkit.dockerfile.v0
    <missing>      8 seconds ago   EXPOSE map[3000/tcp:{}]                         0B        buildkit.dockerfile.v0 
    <missing>      8 seconds ago   WORKDIR /app                                    8.19kB    buildkit.dockerfile.v0
    <missing>      4 days ago      /bin/sh -c #(nop)  CMD ["node"]                 0B
    <missing>      4 days ago      /bin/sh -c #(nop)  ENTRYPOINT ["docker-entry…   0B
    <missing>      4 days ago      /bin/sh -c #(nop) COPY file:4d192565a7220e13…   20.5kB
    <missing>      4 days ago      /bin/sh -c apk add --no-cache --virtual .bui…   7.92MB
    <missing>      4 days ago      /bin/sh -c #(nop)  ENV YARN_VERSION=1.22.19     0B
    <missing>      4 days ago      /bin/sh -c addgroup -g 1000 node     && addu…   126MB
    <missing>      4 days ago      /bin/sh -c #(nop)  ENV NODE_VERSION=20.12.0     0B
    <missing>      2 months ago    /bin/sh -c #(nop)  CMD ["/bin/sh"]              0B
    <missing>      2 months ago    /bin/sh -c #(nop) ADD file:d0764a717d1e9d0af…   8.42MB
    ```

    该输出展示了镜像的各个层，既包括你添加的层，也包括从基础镜像继承的层。

### 推送镜像

镜像构建完成后，接下来将镜像推送到仓库。

1. 使用 [docker push](/reference/cli/docker/image/push/) 命令推送镜像：

    ```console
    $ docker push <YOUR_DOCKER_USERNAME>/concepts-build-image-demo
    ```

    如果看到 `requested access to the resource is denied` 错误，请确认你已登录，且镜像标签中的用户名填写正确。

    片刻之后，你的镜像就会被推送到 Docker Hub。

## 延伸阅读

想进一步了解构建、打标签与发布镜像，请参考：

* [什么是构建上下文？](/build/concepts/context/#what-is-a-build-context)
* [docker build 参考](/engine/reference/commandline/image_build/)
* [docker image tag 参考](/engine/reference/commandline/image_tag/)
* [docker push 参考](/engine/reference/commandline/image_push/)
* [什么是仓库？](/get-started/docker-concepts/the-basics/what-is-a-registry/)

## 下一步

现在你已经了解了如何构建并发布镜像，接下来学习如何使用 Docker 构建缓存来加速构建过程。

{{< button text="使用构建缓存" url="using-the-build-cache" >}}


