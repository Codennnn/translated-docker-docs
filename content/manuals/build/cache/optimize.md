---
title: 优化构建中的缓存使用
description: 概述如何在 Docker 构建中优化缓存利用。
keywords: build, buildkit, buildx, guide, tutorial, mounts, cache mounts, bind mounts
aliases:
  - /build/guide/mounts/
---

在使用 Docker 进行构建时，如果某条指令及其依赖的文件自上次构建以来没有发生变化，则会复用构建缓存中的对应层。
复用缓存层能够显著加速构建流程，因为 Docker 无需再次重建该层。

以下技巧可帮助你优化缓存并加速构建：

- [按顺序组织层](#order-your-layers)：将 Dockerfile 中的命令按合理顺序编排，避免不必要的缓存失效。
- [保持上下文精简](#keep-the-context-small)：上下文是发送给构建器以处理构建指令的文件与目录集合。上下文越小，传输的数据越少，缓存失效的概率也越低。
- [使用绑定挂载](#use-bind-mounts)：将宿主机的文件或目录挂载到构建容器中，可避免在镜像中产生不必要的层，从而减少构建开销。
- [使用缓存挂载](#use-cache-mounts)：为构建阶段指定持久的包缓存，尤其在使用包管理器安装依赖时效果明显。即便重建某层，也只需下载新增或变更的包。
- [使用外部缓存](#use-an-external-cache)：将构建缓存存放到远端位置，便于在多个构建与不同环境间共享。

## Order your layers（按顺序组织层）

将 Dockerfile 中的命令按逻辑顺序组织，是优化缓存的起点。由于某一步的变化会导致后续步骤重建，
尽量把开销大的步骤放在前面，把经常变化的步骤放在后面，以避免无谓地重建未变化的层。

看一个例子：下面的 Dockerfile 片段会基于当前目录的源代码执行一次 JavaScript 构建：

```dockerfile
# syntax=docker/dockerfile:1
FROM node
WORKDIR /app
COPY . .          # Copy over all files in the current directory
RUN npm install   # Install dependencies
RUN npm build     # Run build
```

这个 Dockerfile 效率较低：只要更新任意文件，每次构建镜像都会重新安装所有依赖，即使这些依赖并未变化。

更好的做法是将 `COPY` 分两步：先复制包管理文件（例如 `package.json` 与 `yarn.lock`），再安装依赖；
最后再复制经常变动的项目源码。

```dockerfile
# syntax=docker/dockerfile:1
FROM node
WORKDIR /app
COPY package.json yarn.lock .    # Copy package management files
RUN npm install                  # Install dependencies
COPY . .                         # Copy over project files
RUN npm build                    # Run build
```

将依赖安装放到较早的层中，项目文件更改时就无需重建这些层。

## Keep the context small（保持上下文精简）

确保上下文不包含无关文件的最简单方式，是在构建上下文根目录创建 `.dockerignore` 文件。
它的工作方式类似 `.gitignore`，可将不需要的文件和目录排除在构建上下文之外。

下面示例的 `.dockerignore` 会排除 `node_modules` 目录，以及所有以 `tmp` 开头的文件或目录：

```plaintext {title=".dockerignore"}
node_modules
tmp*
```

`.dockerignore` 中的忽略规则对整个构建上下文（含子目录）生效，属于一种相对粗粒度的机制。
但它非常适合排除明确不需要参与构建的内容，例如临时文件、日志文件与构建产物等。

## Use bind mounts（使用绑定挂载）

在使用 `docker run` 或 Docker Compose 运行容器时，你或许已经熟悉绑定挂载。
绑定挂载允许将宿主机的文件或目录挂载进容器。

```bash
# bind mount using the -v flag
docker run -v $(pwd):/path/in/container image-name
# bind mount using the --mount flag
docker run --mount=type=bind,src=.,dst=/path/in/container image-name
```

在构建中使用绑定挂载，可在 Dockerfile 的 `RUN` 指令中配合 `--mount` 标志：

```dockerfile
FROM golang:latest
WORKDIR /app
RUN --mount=type=bind,target=. go build -o /app/hello
```

在该示例中，`go build` 执行前会将当前目录挂载到构建容器中。源代码仅在该 `RUN` 指令期间可用；
指令执行结束后，被挂载的文件不会出现在最终镜像或构建缓存中，只有 `go build` 的产物会保留下来。

`COPY` 和 `ADD` 会把上下文中的文件复制到构建容器中。
而绑定挂载有利于优化缓存，因为不会给镜像增加不必要的层。
如果你的构建上下文较大且仅用于生成某个工件（artifact），推荐使用绑定挂载把必要源码临时挂载到构建中。
若改用 `COPY`，即使这些文件不会进入最终镜像，BuildKit 仍会把它们计入缓存。

在构建中使用绑定挂载时，需要注意：

- 绑定挂载默认只读；若需写入，请加 `rw` 选项。但即使加了 `rw`，改动也不会持久化到最终镜像或构建缓存中。
  文件写入仅在该 `RUN` 指令执行期间有效，指令结束后会被丢弃。
- 被挂载的文件不会出现在最终镜像中。最终镜像仅包含该 `RUN` 指令的产物。
  如果需要将上下文中的文件打包进最终镜像，应使用 `COPY` 或 `ADD`。
- 若目标目录非空，挂载后的文件会“遮蔽”目标目录的原有内容；
  当 `RUN` 执行结束，原有内容将被还原。

  {{< accordion title="Example" >}}

  例如，假设构建上下文中只有一个 `Dockerfile`：

  ```plaintext
  .
  └── Dockerfile
  ```

  以及一个将当前目录挂载进构建容器的 Dockerfile：

  ```dockerfile
  FROM alpine:latest
  WORKDIR /work
  RUN touch foo.txt
  RUN --mount=type=bind,target=. ls
  RUN ls
  ```

  第一次带绑定挂载的 `ls` 展示的是被挂载目录的内容；
  第二次 `ls` 列出的则是原始构建上下文中的内容。

  ```plaintext {title="Build log"}
  #8 [stage-0 3/5] RUN touch foo.txt
  #8 DONE 0.1s
  
  #9 [stage-0 4/5] RUN --mount=target=. ls -1
  #9 0.040 Dockerfile
  #9 DONE 0.0s
  
  #10 [stage-0 5/5] RUN ls -1
  #10 0.046 foo.txt
  #10 DONE 0.1s
  ```

  {{< /accordion >}}

## Use cache mounts（使用缓存挂载）

Docker 中的常规缓存层要求“指令与其依赖文件”完全匹配。
只要任一发生变化，该层就会失效并被重建。

缓存挂载用于在构建期间指定一个持久化的缓存位置。该缓存可跨构建累积读写。
这意味着即使某层被重建，也只会下载新增或变更的包，未变化的包将直接从缓存挂载中复用。

在构建中使用缓存挂载，可在 Dockerfile 的 `RUN` 指令中配合 `--mount` 标志：

```dockerfile
FROM node:latest
WORKDIR /app
RUN --mount=type=cache,target=/root/.npm npm install
```

在该示例中，`npm install` 为 `/root/.npm` 目录（npm 的默认缓存位置）使用了缓存挂载。
该缓存在不同构建之间保持持久化，即便重建该层也只会下载新增或变更的包；
缓存的变更会跨构建保留，并可在多个构建之间共享。

如何指定缓存挂载，取决于所用的构建工具。如不确定，请参考对应工具的文档。如下是一些示例：

{{< tabs >}}
{{< tab name="Go" >}}

```dockerfile
RUN --mount=type=cache,target=/go/pkg/mod \
    go build -o /app/hello
```

{{< /tab >}}
{{< tab name="Apt" >}}

```dockerfile
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
  --mount=type=cache,target=/var/lib/apt,sharing=locked \
  apt update && apt-get --no-install-recommends install -y gcc
```

{{< /tab >}}
{{< tab name="Python" >}}

```dockerfile
RUN --mount=type=cache,target=/root/.cache/pip \
    pip install -r requirements.txt
```

{{< /tab >}}
{{< tab name="Ruby" >}}

```dockerfile
RUN --mount=type=cache,target=/root/.gem \
    bundle install
```

{{< /tab >}}
{{< tab name="Rust" >}}

```dockerfile
RUN --mount=type=cache,target=/app/target/ \
    --mount=type=cache,target=/usr/local/cargo/git/db \
    --mount=type=cache,target=/usr/local/cargo/registry/ \
    cargo build
```

{{< /tab >}}
{{< tab name=".NET" >}}

```dockerfile
RUN --mount=type=cache,target=/root/.nuget/packages \
    dotnet restore
```

{{< /tab >}}
{{< tab name="PHP" >}}  

```dockerfile
RUN --mount=type=cache,target=/tmp/cache \
    composer install
```

{{< /tab >}}
{{< /tabs >}}

务必阅读你所使用构建工具的文档，确保采用正确的缓存挂载参数。不同包管理器对缓存的要求不同，
错误的参数会导致意外行为。比如 Apt 需要对其数据的独占访问，
因此示例中使用 `sharing=locked`，以保证并行构建在使用相同缓存挂载时互相等待，避免同时访问同一缓存文件。

## Use an external cache（使用外部缓存）

构建默认的缓存存储位于当前使用的构建器（BuildKit 实例）内部，每个构建器使用其独立的缓存存储。
在不同构建器之间切换时，缓存不会共享。外部缓存允许你将缓存数据推送/拉取到远端位置。

外部缓存对 CI/CD 流水线尤为有用：构建器通常是短暂存在的，构建时间也十分宝贵。
在构建之间复用缓存可以显著加速并降低成本；本地开发环境同样可以使用同一份缓存。

要使用外部缓存，可在 `docker buildx build` 命令中指定 `--cache-to` 与 `--cache-from`：

- `--cache-to`：将构建缓存导出到指定位置。
- `--cache-from`：指定构建所使用的远程缓存。

下面示例展示了如何在 GitHub Actions 中使用 `docker/build-push-action`，
并将构建缓存层推送到 OCI 仓库镜像：

```yaml {title=".github/workflows/ci.yml"}
name: ci

on:
  push:

jobs:
  docker:
    runs-on: ubuntu-latest
    steps:
      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ vars.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Build and push
        uses: docker/build-push-action@v6
        with:
          push: true
          tags: user/app:latest
          cache-from: type=registry,ref=user/app:buildcache
          cache-to: type=registry,ref=user/app:buildcache,mode=max
```

此配置会让 BuildKit 从镜像 `user/app:buildcache` 拉取缓存；
构建完成后，会将新的构建缓存推送回该镜像，覆盖旧缓存。

该缓存也可在本地构建中复用。你可以通过 `docker buildx build` 的 `--cache-from` 拉取缓存：

```console
$ docker buildx build --cache-from type=registry,ref=user/app:buildcache .
```

## Summary（总结）

优化缓存使用可以显著加速构建。
保持上下文精简、使用绑定挂载与缓存挂载，以及引入外部缓存，都是充分利用构建缓存、提升构建速度的有效手段。

想进一步了解本文提到的概念，请参阅：

- [.dockerignore files](/manuals/build/concepts/context.md#dockerignore-files)
- [Cache invalidation](/manuals/build/cache/invalidation.md)
- [Cache mounts](/reference/dockerfile.md#run---mounttypecache)
- [Cache backend types](/manuals/build/cache/backends/_index.md)
- [Building best practices](/manuals/build/building/best-practices.md)
