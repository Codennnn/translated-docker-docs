---
title: Docker 构建缓存
linkTitle: 缓存
weight: 60
description: 合理利用构建缓存，加速你的构建
keywords: build, buildx, buildkit, dockerfile, image layers, build instructions, build context
aliases:
  - /build/building/cache/
  - /build/guide/layers/
---

当你多次构建同一个 Docker 镜像时，掌握如何优化构建缓存，是确保构建高速稳定的关键手段。

## 构建缓存的工作原理

理解 Docker 的构建缓存机制，有助于你编写更高效的 Dockerfile，从而获得更快的构建速度。

下面是一个用于 C 程序的精简 Dockerfile 示例：

```dockerfile
# syntax=docker/dockerfile:1
FROM ubuntu:latest

RUN apt-get update && apt-get install -y build-essentials
COPY main.c Makefile /src/
WORKDIR /src/
RUN make build
```

Dockerfile 中的每条指令，都会对应到最终镜像中的一个层（layer）。
你可以把镜像层理解为一个堆栈：每一层都在其之前的层之上累加内容：

![Image layer diagram](../images/cache-stack.png)

只要某一层发生变化，该层就需要重新构建。比如，你改动了 `main.c`，那么 `COPY` 指令需要再次执行，
这些变更才会体现在镜像中。换言之，这一层的缓存会被判定失效。

当某一层被修改，它之后的所有层都会受到影响。当包含 `COPY` 指令的层缓存失效时，后续所有层也都需要重新运行：

![Image layer diagram, showing cache invalidation](../images/cache-stack-invalidated.png)

这就是 Docker 构建缓存的核心要点：一旦某层发生变化，其后所有层都需要重新构建。
即使这些层本身没有任何变化，也必须重新执行。

## 相关资源

想要进一步了解如何利用缓存进行高效构建，请参阅：

- [Cache invalidation](invalidation.md)
- [Optimize build cache](optimization.md)
- [Garbage collection](garbage-collection.md)
- [Cache storage backends](./backends/_index.md)
