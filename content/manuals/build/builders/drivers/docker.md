---
title: Docker 驱动
description: |
  Docker 驱动是默认驱动。
  它使用打包在 Docker Engine 中的 BuildKit。
keywords: build, buildx, driver, builder, docker
aliases:
  - /build/buildx/drivers/docker/
  - /build/building/drivers/docker/
  - /build/drivers/docker/
---

Buildx 的 Docker 驱动是默认驱动。它使用直接内置于 Docker Engine 的 BuildKit 服务端组件。
Docker 驱动无需任何配置。

与其他驱动不同，使用 Docker 驱动的构建器无法手动创建；
它们只会基于 Docker 上下文自动创建。

使用 Docker 驱动构建的镜像会自动加载到本地镜像存储。

## 概要

```console
# buildx 默认使用 Docker 驱动
docker buildx build .
```

在 Docker 驱动下，无法自定义所使用的 BuildKit 版本，也不能向构建器传递额外的 BuildKit 参数。
BuildKit 的版本与参数由 Docker Engine 在内部预设。

如果你需要更多配置与灵活性，建议使用[Docker 容器驱动](./docker-container.md)。

## 延伸阅读

关于 Docker 驱动的更多信息，参见
[buildx 参考](/reference/cli/docker/buildx/create.md#driver)。
