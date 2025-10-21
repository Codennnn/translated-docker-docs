---
title: 优化 Docker Offload 使用
linktitle: 优化使用
weight: 40
description: 了解如何优化 Docker Offload 的使用。
keywords: 云端, 优化, 性能, 缓存, 成本效率
---

Docker Offload 在远程运行您的构建，而不是在您调用构建的机器上。这意味着文件必须通过网络从本地系统传输到云端。

与本地传输相比，通过网络传输文件会引入更高的延迟和更低的带宽。为了减少这些影响，Docker Offload 包含了多项性能优化：

- 它使用附加的存储卷进行构建缓存，使缓存读写速度更快。
- 当将构建结果拉回本地机器时，它只传输自上次构建以来发生变化的层。

即使有这些优化，大型项目或较慢的网络连接也可能导致更长的传输时间。以下是几种优化 Docker Offload 构建设置的方法：

- [使用 `.dockerignore` 文件](#dockerignore-文件)
- [选择精简基础镜像](#精简基础镜像)
- [使用多阶段构建](#多阶段构建)
- [在构建过程中获取远程文件](#在构建过程中获取远程文件)
- [利用多线程工具](#利用多线程工具)

有关 Dockerfile 的一般技巧，请参阅[构建最佳实践](/manuals/build/building/best-practices.md)。

## dockerignore 文件

[`.dockerignore` 文件](/manuals/build/concepts/context.md#dockerignore-文件)允许您指定哪些本地文件*不应*包含在构建上下文中。被这些模式排除的文件在构建过程中不会上传到 Docker Offload。

通常应忽略的项目：

- `.git` - 避免传输版本历史记录。（注意：您将无法在构建中运行 `git` 命令。）
- 构建工件或本地生成的二进制文件。
- 依赖文件夹（如 `node_modules`），如果这些文件夹在构建过程中会被恢复。

作为经验法则，您的 `.dockerignore` 应该与您的 `.gitignore` 类似。

## 精简基础镜像

在 `FROM` 指令中使用更小的基础镜像可以减少最终镜像大小并提高构建性能。[`alpine`](https://hub.docker.com/_/alpine) 镜像是最小基础镜像的一个很好的例子。

对于完全静态的二进制文件，您可以使用 [`scratch`](https://hub.docker.com/_/scratch)，它是一个空的基础镜像。

## 多阶段构建

[多阶段构建](/build/building/multi-stage/)允许您在 Dockerfile 中分离构建时和运行时环境。这不仅减少了最终镜像的大小，还允许在构建过程中并行执行各个阶段。

使用 `COPY --from` 从早期阶段或外部镜像复制文件。这种方法有助于最小化不必要的层并减少最终镜像大小。

## 在构建过程中获取远程文件

尽可能在构建过程中从互联网下载大文件，而不是将它们打包在本地上下文中。这样可以避免从客户端到 Docker Offload 的网络传输。

您可以使用以下方式实现：

- Dockerfile [`ADD` 指令](/reference/dockerfile/#add)
- `RUN` 命令，如 `wget`、`curl` 或 `rsync`

### 利用多线程工具

一些构建工具（如 `make`）默认是单线程的。如果工具支持，请将其配置为并行运行。例如，使用 `make --jobs=4` 同时运行四个作业。

充分利用云环境中可用的 CPU 资源可以显著缩短构建时间。
