---
title: Docker 容器驱动
description: Docker 容器驱动在容器镜像中运行 BuildKit。
keywords: build, buildx, driver, builder, docker-container
aliases:
  - /build/buildx/drivers/docker-container/
  - /build/building/drivers/docker-container/
  - /build/drivers/docker-container/
---

Docker 容器驱动（docker-container）允许你在专用的 Docker 容器中创建一个可托管、可自定义的 BuildKit 运行环境。

与默认的 Docker 驱动相比，使用 Docker 容器驱动具有一些优势，例如：

- 可以指定自定义的 BuildKit 版本。
- 构建多架构镜像，参见 [QEMU](#qemu)
- 提供更高级的[缓存导入与导出](/manuals/build/cache/backends/_index.md) 选项。

## 概要

运行以下命令，创建一个名为 `container` 且使用 Docker 容器驱动的构建器：

```console
$ docker buildx create \
  --name container \
  --driver=docker-container \
  --driver-opt=[key=value,...]
container
```

下表说明了可通过 `--driver-opt` 传入的驱动特定选项：

| 参数             | 类型    | 默认值           | 描述                                                                                                                   |
| ---------------- | ------- | ---------------- | ---------------------------------------------------------------------------------------------------------------------- |
| `image`          | String  |                  | 指定容器使用的 BuildKit 镜像。                                                                                          |
| `memory`         | String  |                  | 设置容器可使用的内存上限。                                                                                             |
| `memory-swap`    | String  |                  | 设置容器的 swap 内存上限。                                                                                              |
| `cpu-quota`      | String  |                  | 对容器施加 CPU CFS 配额。                                                                                               |
| `cpu-period`     | String  |                  | 设置容器的 CPU CFS 调度周期。                                                                                           |
| `cpu-shares`     | String  |                  | 配置容器的 CPU 份额（相对权重）。                                                                                       |
| `cpuset-cpus`    | String  |                  | 限制容器可使用的 CPU 核心集合。                                                                                        |
| `cpuset-mems`    | String  |                  | 限制容器可使用的 CPU 内存节点集合。                                                                                    |
| `default-load`   | Boolean | `false`          | 构建完成后自动将镜像加载到 Docker Engine 的镜像存储。                                                                  |
| `network`        | String  |                  | 设置容器的网络模式。                                                                                                   |
| `cgroup-parent`  | String  | `/docker/buildx` | 当 Docker 使用 "cgroupfs" 驱动时，设置容器的 cgroup 父级。                                                              |
| `restart-policy` | String  | `unless-stopped` | 设置容器的[重启策略](/manuals/engine/containers/start-containers-automatically.md#use-a-restart-policy)。                |
| `env.<key>`      | String  |                  | 在容器内将环境变量 `key` 设置为指定的 `value`。                                                                         |

在为容器配置资源限制之前，建议先阅读[配置容器运行时资源约束](/engine/containers/resource_constraints/)。

## 用法

当你运行构建时，Buildx 会拉取指定的 `image`（默认为 [`moby/buildkit`](https://hub.docker.com/r/moby/buildkit)）。
容器启动后，Buildx 会把构建任务提交给该容器化的构建服务器。

```console
$ docker buildx build -t <image> --builder=container .
WARNING: No output specified with docker-container driver. Build result will only remain in the build cache. To push result image into registry use --push or to load image into docker use --load
#1 [internal] booting buildkit
#1 pulling image moby/buildkit:buildx-stable-1
#1 pulling image moby/buildkit:buildx-stable-1 1.9s done
#1 creating container buildx_buildkit_container0
#1 creating container buildx_buildkit_container0 0.5s done
#1 DONE 2.4s
...
```

## 缓存持久化

`docker-container` 驱动支持缓存持久化：它会将所有 BuildKit 状态与相关缓存存放到专用的 Docker 卷中。

即便通过 `docker buildx rm` 和 `docker buildx create` 重建驱动，也可以保留 `docker-container` 驱动的缓存。删除构建器时使用 `--keep-state` 标志即可：

例如，创建一个名为 `container` 的构建器，并在移除时保留其状态：

```console
# 设置一个构建器
$ docker buildx create --name=container --driver=docker-container --use --bootstrap
container
$ docker buildx ls
NAME/NODE       DRIVER/ENDPOINT              STATUS   BUILDKIT PLATFORMS
container *     docker-container
  container0    desktop-linux                running  v0.10.5  linux/amd64
$ docker volume ls
DRIVER    VOLUME NAME
local     buildx_buildkit_container0_state

# 在保留状态的情况下移除该构建器
$ docker buildx rm --keep-state container
$ docker volume ls
DRIVER    VOLUME NAME
local     buildx_buildkit_container0_state

# 使用相同名称新建的驱动将拥有之前的全部状态！
$ docker buildx create --name=container --driver=docker-container --use --bootstrap
container
```

## QEMU {#qemu}

`docker-container` 驱动支持使用 [QEMU](https://www.qemu.org/)（用户态）来构建非原生平台。使用 `--platform` 标志指定要构建的目标架构。

例如，为 `amd64` 与 `arm64` 构建 Linux 镜像：

```console
$ docker buildx build \
  --builder=container \
  --platform=linux/amd64,linux/arm64 \
  -t <registry>/<image> \
  --push .
```

> [!NOTE]
>
> 基于 QEMU 的仿真通常比原生构建慢得多，尤其是在编译、压缩/解压缩等计算密集型任务上。

## 自定义网络

你可以自定义构建器容器所使用的网络。这在需要为构建使用特定网络时非常有用。

例如，我们先[创建一个名为 `foonet` 的网络](/reference/cli/docker/network/create.md)：

```console
$ docker network create foonet
```

然后创建一个将使用该网络的 [`docker-container` 构建器](/reference/cli/docker/buildx/create.md)：

```console
$ docker buildx create --use \
  --name mybuilder \
  --driver docker-container \
  --driver-opt "network=foonet"
```

启动并[检查 `mybuilder`](/reference/cli/docker/buildx/inspect.md)：

```console
$ docker buildx inspect --bootstrap
```

[检查构建器容器](/reference/cli/docker/inspect.md)，查看其使用的网络：

```console
$ docker inspect buildx_buildkit_mybuilder0 --format={{.NetworkSettings.Networks}}
map[foonet:0xc00018c0c0]
```

## 延伸阅读

关于 Docker 容器驱动的更多信息，参见
[buildx 参考](/reference/cli/docker/buildx/create.md#driver)。
