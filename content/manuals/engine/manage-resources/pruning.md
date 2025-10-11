---
description: 使用 prune 命令移除未使用的资源，释放磁盘空间
keywords: pruning, prune, images, volumes, containers, networks, disk, administration,
  garbage collection
title: 清理未使用的 Docker 对象
aliases:
- /engine/admin/pruning/
- /config/pruning/
---

Docker 对清理未使用对象（常称为“垃圾回收”）采取保守策略。诸如镜像、容器、卷和网络等对象，除非你明确要求，Docker 一般不会自动删除。这可能导致磁盘空间被额外占用。针对每种对象类型，Docker 都提供了相应的 `prune` 清理命令。此外，你还可以使用 `docker system prune` 一次性清理多种对象。本文介绍如何使用这些 `prune` 命令。

## 清理镜像

`docker image prune` 用于清理未使用的镜像。默认情况下，它只会清理“悬空镜像”（dangling images）。悬空镜像指未被打标签，且不被任何容器引用的镜像。要移除悬空镜像：

```console
$ docker image prune

WARNING! This will remove all dangling images.
Are you sure you want to continue? [y/N] y
```

若要移除所有未被现有容器使用的镜像，使用 `-a` 参数：

```console
$ docker image prune -a

WARNING! This will remove all images without at least one container associated to them.
Are you sure you want to continue? [y/N] y
```

默认会提示确认。要跳过提示，使用 `-f` 或 `--force` 参数。

你可以通过 `--filter` 指定过滤条件以限定清理范围。例如，仅考虑 24 小时前创建的镜像：

```console
$ docker image prune -a --filter "until=24h"
```

还有更多可用的过滤表达式，参见
[`docker image prune` 参考](/reference/cli/docker/image/prune.md) 了解更多示例。

## 清理容器

当你停止一个容器时，除非在启动时使用了 `--rm`，否则容器不会被自动移除。使用 `docker ps -a` 可以查看主机上的全部容器（包括已停止的）。尤其在开发环境中，容器数量可能会让你吃惊！已停止容器的可写层仍会占用磁盘空间。要清理这些内容，可以使用 `docker container prune`。

```console
$ docker container prune

WARNING! This will remove all stopped containers.
Are you sure you want to continue? [y/N] y
```

默认会提示确认。要跳过提示，使用 `-f` 或 `--force` 参数。

默认会移除所有已停止的容器。你可以通过 `--filter` 限定范围。例如，仅移除 24 小时之前停止的容器：

```console
$ docker container prune --filter "until=24h"
```

还有更多可用的过滤表达式，参见
[`docker container prune` 参考](/reference/cli/docker/container/prune.md) 了解更多示例。

## 清理卷

卷可以被一个或多个容器使用，并会占用主机存储空间。卷从不会被自动删除，因为这样做可能会破坏数据。

```console
$ docker volume prune

WARNING! This will remove all volumes not used by at least one container.
Are you sure you want to continue? [y/N] y
```

默认会提示确认。要跳过提示，使用 `-f` 或 `--force` 参数。

默认会移除所有未使用的卷。你可以通过 `--filter` 限定范围。例如，仅移除未打上 `keep` 标签的卷：

```console
$ docker volume prune --filter "label!=keep"
```

还有更多可用的过滤表达式，参见
[`docker volume prune` 参考](/reference/cli/docker/volume/prune.md) 了解更多示例。

## 清理网络

Docker 网络本身不占用太多磁盘空间，但会创建 `iptables` 规则、网桥设备以及路由表项。要清理这些内容，可以使用 `docker network prune` 移除未被任何容器使用的网络。

```console
$ docker network prune

WARNING! This will remove all networks not used by at least one container.
Are you sure you want to continue? [y/N] y
```

默认会提示确认。要跳过提示，使用 `-f` 或 `--force` 参数。

默认会移除所有未使用的网络。你可以通过 `--filter` 限定范围。例如，仅移除 24 小时之前创建的网络：

```console
$ docker network prune --filter "until=24h"
```

还有更多可用的过滤表达式，参见
[`docker network prune` 参考](/reference/cli/docker/network/prune.md) 了解更多示例。

## 一键清理全部

`docker system prune` 是一个便捷命令，可同时清理镜像、容器与网络。默认不会清理卷，如需同时清理卷，请添加 `--volumes` 参数。

```console
$ docker system prune

WARNING! This will remove:
        - all stopped containers
        - all networks not used by at least one container
        - all dangling images
        - unused build cache

Are you sure you want to continue? [y/N] y
```

若还需清理卷，请添加 `--volumes` 参数：

```console
$ docker system prune --volumes

WARNING! This will remove:
        - all stopped containers
        - all networks not used by at least one container
        - all volumes not used by at least one container
        - all dangling images
        - all build cache

Are you sure you want to continue? [y/N] y
```

默认会提示确认。要跳过提示，使用 `-f` 或 `--force` 参数。

默认会移除所有未使用的容器、网络与镜像。你可以通过 `--filter` 限定范围。例如，移除 24 小时之前的条目：

```console
$ docker system prune --filter "until=24h"
```

还有更多可用的过滤表达式，参见
[`docker system prune` 参考](/reference/cli/docker/system/prune.md) 了解更多示例。
