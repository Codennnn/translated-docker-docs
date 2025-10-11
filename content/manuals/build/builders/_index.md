---
title: 构建器（Builders）
weight: 40
keywords: build, buildx, builders, buildkit, drivers, backend
description: 了解构建器的概念以及如何进行管理
---

构建器是一个可用于运行构建的 BuildKit 守护进程。BuildKit 是构建引擎，用于解析并执行 Dockerfile 中的构建步骤，以产出容器镜像或其他构建产物。

你可以创建与管理构建器、查看其详情，甚至连接到远程运行的构建器。你可以通过 Docker CLI 与构建器交互。

## 默认构建器

Docker Engine 会自动创建一个构建器，作为你进行构建时的默认后端。
该构建器使用与守护进程一起打包的 BuildKit 库，无需额外配置。

默认构建器直接绑定到 Docker 守护进程及其[上下文](/manuals/engine/manage-resources/contexts.md)。
若你切换 Docker 上下文，`default` 构建器也会随之指向新的 Docker 上下文。

## 构建驱动

Buildx 提供了[构建驱动](drivers/_index.md)的概念，用于表示不同的构建器配置。
守护进程创建的默认构建器使用 [`docker` 驱动](drivers/docker.md)。

Buildx 支持以下构建驱动：

- `docker`：使用打包在 Docker 守护进程中的 BuildKit 库
- `docker-container`：通过 Docker 创建独立的 BuildKit 容器
- `kubernetes`：在 Kubernetes 集群中创建 BuildKit Pod
- `remote`：直接连接到手动管理的 BuildKit 守护进程

## 已选构建器（Selected builder）

“已选构建器”是指当你运行构建命令时默认使用的构建器。

当你运行构建或通过 CLI 与构建器交互时，可以使用可选的 `--builder` 标志，或 `BUILDX_BUILDER`
[环境变量](../building/variables.md#buildx_builder)来按名称指定构建器。
如果未显式指定，将使用已选构建器。

使用 `docker buildx ls` 可查看可用的构建器实例。
构建器名称后的星号（`*`）表示它是已选构建器。

```console
$ docker buildx ls
NAME/NODE       DRIVER/ENDPOINT      STATUS   BUILDKIT PLATFORMS
default *       docker
  default       default              running  v0.11.6  linux/amd64, linux/amd64/v2, linux/amd64/v3, linux/386
my_builder      docker-container
  my_builder0   default              running  v0.11.6  linux/amd64, linux/amd64/v2, linux/amd64/v3, linux/386
```

### 切换构建器

要在多个构建器之间切换，使用 `docker buildx use <name>`。

执行该命令后，你指定的构建器会在后续的构建中被自动选用。

### `docker build` 与 `docker buildx build` 的差异

尽管 `docker build` 是 `docker buildx build` 的别名，但两者仍存在一些差异。
在 Buildx 模式下，构建客户端与守护进程（BuildKit）是解耦的，这意味着你可以从同一个客户端使用多个构建器，甚至可以连接远程构建器。

为确保与旧版本 Docker CLI 的兼容性，`docker build` 始终默认使用 Docker Engine 自带的默认构建器。
相比之下，`docker buildx build` 会在将构建发送给 BuildKit 之前，检查你是否设置了其他默认构建器。

若希望 `docker build` 使用非默认构建器，你需要：

- 显式指定构建器：通过 `--builder` 标志或 `BUILDX_BUILDER` 环境变量：

  ```console
  $ BUILDX_BUILDER=my_builder docker build .
  $ docker build --builder my_builder .
  ```

- 将 Buildx 配置为默认客户端，可运行：

  ```console
  $ docker buildx install
  ```

  该命令会更新你的 [Docker CLI 配置文件](/reference/cli/docker/_index.md#configuration-files)，
  确保所有与构建相关的命令都通过 Buildx 路由。

  > [!TIP]
  > 若要撤销该更改，运行 `docker buildx uninstall`。

<!-- vale Docker.We = NO -->

总体而言，当你需要使用自定义构建器时，建议使用 `docker buildx build`。
这可以确保你的[已选构建器](#已选构建器selected-builder)配置被正确识别与应用。

<!-- vale Docker.We = YES -->

## 更多信息

- 了解如何与构建器交互并进行管理，参见：[管理构建器](./manage.md)
- 了解不同类型的构建器，参见：[构建驱动](drivers/_index.md)
