---
title: 过滤命令
weight: 30
description: |
  使用 CLI 的过滤功能，仅包含你定义条件所匹配的资源。
keywords: cli, filter, commands, output, include, exclude
aliases:
  - /config/filter/
---

你可以使用 `--filter` 标志限定命令的作用范围。启用过滤后，命令仅返回与给定模式匹配的条目。

## 使用过滤器

`--filter` 接受由运算符分隔的键值对。

```console
$ docker COMMAND --filter "KEY=VALUE"
```

键表示要过滤的字段；值是该字段要匹配的模式；运算符可以是等于（`=`）或不等于（`!=`）。

例如，`docker images --filter reference=alpine` 只会输出 `alpine` 镜像：

```console
$ docker images
REPOSITORY   TAG       IMAGE ID       CREATED          SIZE
ubuntu       24.04     33a5cc25d22c   36 minutes ago   101MB
ubuntu       22.04     152dc042452c   36 minutes ago   88.1MB
alpine       3.21      a8cbb8c69ee7   40 minutes ago   8.67MB
alpine       latest    7144f7bab3d4   40 minutes ago   11.7MB
busybox      uclibc    3e516f71d880   48 minutes ago   2.4MB
busybox      glibc     7338d0c72c65   48 minutes ago   6.09MB
$ docker images --filter reference=alpine
REPOSITORY   TAG       IMAGE ID       CREATED          SIZE
alpine       3.21      a8cbb8c69ee7   40 minutes ago   8.67MB
alpine       latest    7144f7bab3d4   40 minutes ago   11.7MB
```

可用字段（本例为 `reference`）取决于具体命令。有些过滤器要求精确匹配，有些支持前缀/部分匹配，还有些允许使用正则表达式。

各命令支持的过滤能力，见下方的[参考列表](#reference)。

## 组合过滤条件

可以多次传入 `--filter` 来组合条件。下面的示例展示如何打印同时满足 `alpine:latest` 或 `busybox` 的镜像（逻辑 “或”）：

```console
$ docker images
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
ubuntu       24.04     33a5cc25d22c   2 hours ago   101MB
ubuntu       22.04     152dc042452c   2 hours ago   88.1MB
alpine       3.21      a8cbb8c69ee7   2 hours ago   8.67MB
alpine       latest    7144f7bab3d4   2 hours ago   11.7MB
busybox      uclibc    3e516f71d880   2 hours ago   2.4MB
busybox      glibc     7338d0c72c65   2 hours ago   6.09MB
$ docker images --filter reference=alpine:latest --filter=reference=busybox
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
alpine       latest    7144f7bab3d4   2 hours ago   11.7MB
busybox      uclibc    3e516f71d880   2 hours ago   2.4MB
busybox      glibc     7338d0c72c65   2 hours ago   6.09MB
```

### 多个取反过滤器

部分命令支持对[标签](/manuals/engine/manage-resources/labels.md)使用取反过滤器。取反过滤器仅保留不匹配给定模式的结果。如下命令会清理所有没有 `foo` 标签的容器：

```console
$ docker container prune --filter "label!=foo"
```

注意：组合多个标签的取反过滤器时，效果是合并为一个整体的负约束，即逻辑 “与”。下面的命令会清理除同时拥有 `foo` 与 `bar` 标签之外的所有容器；仅有其一（`foo` 或 `bar`）的容器也会被清理。

```console
$ docker container prune --filter "label!=foo" --filter "label!=bar"
```

## 参考

关于支持 `--filter` 的命令及其过滤能力，详见各命令的 CLI 参考：

- [`docker config ls`](/reference/cli/docker/config/ls.md)
- [`docker container prune`](/reference/cli/docker/container/prune.md)
- [`docker image prune`](/reference/cli/docker/image/prune.md)
- [`docker image ls`](/reference/cli/docker/image/ls.md)
- [`docker network ls`](/reference/cli/docker/network/ls.md)
- [`docker network prune`](/reference/cli/docker/network/prune.md)
- [`docker node ls`](/reference/cli/docker/node/ls.md)
- [`docker node ps`](/reference/cli/docker/node/ps.md)
- [`docker plugin ls`](/reference/cli/docker/plugin/ls.md)
- [`docker container ls`](/reference/cli/docker/container/ls.md)
- [`docker search`](/reference/cli/docker/search.md)
- [`docker secret ls`](/reference/cli/docker/secret/ls.md)
- [`docker service ls`](/reference/cli/docker/service/ls.md)
- [`docker service ps`](/reference/cli/docker/service/ps.md)
- [`docker stack ps`](/reference/cli/docker/stack/ps.md)
- [`docker system prune`](/reference/cli/docker/system/prune.md)
- [`docker volume ls`](/reference/cli/docker/volume/ls.md)
- [`docker volume prune`](/reference/cli/docker/volume/prune.md)
