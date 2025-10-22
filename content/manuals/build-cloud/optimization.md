---
title: 优化云构建
linkTitle: 优化
weight: 40
description: 远程构建与本地构建有所不同。以下是如何优化远程构建器的方法。
keywords: 构建, 云构建, 优化, 远程, 本地, 云
aliases:
  - /build/cloud/optimization/
---

Docker Build Cloud 在远程运行您的构建，而不是在您调用构建的机器上。这意味着客户端和构建器之间的文件传输通过网络进行。

通过网络传输文件比本地传输具有更高的延迟和更低的带宽。Docker Build Cloud 有几个功能来缓解这种情况：

- 它使用附加的存储卷进行构建缓存，这使得读取和写入缓存非常快。
- 将构建结果加载回客户端时，只拉取与之前构建相比已更改的层。

尽管有这些优化，但对于大型项目或网络连接较慢的情况，远程构建仍可能导致上下文传输和镜像加载缓慢。以下是一些您可以优化构建以使传输更高效的方法：

- [Dockerignore 文件](#dockerignore-文件)
- [精简基础镜像](#精简基础镜像)
- [多阶段构建](#多阶段构建)
- [在构建中获取远程文件](#在构建中获取远程文件)
- [多线程工具](#多线程工具)

有关如何优化构建的更多信息，请参阅[构建最佳实践](/manuals/build/building/best-practices.md)。

### Dockerignore 文件

使用 [`.dockerignore` 文件](/manuals/build/concepts/context.md#dockerignore-files)，您可以明确指定不想包含在构建上下文中的本地文件。被您在忽略文件中指定的 glob 模式捕获的文件不会被传输到远程构建器。

您可能想要添加到 `.dockerignore` 文件中的一些示例如下：

- `.git` — 跳过在构建上下文中发送版本控制历史记录。请注意，这意味着您将无法在构建步骤中运行 Git 命令，例如 `git rev-parse`。
- 包含构建产物的目录，例如二进制文件。在开发过程中本地创建的构建产物。
- 包管理器的供应商目录，例如 `node_modules`。

一般来说，您的 `.dockerignore` 文件内容应该与您的 `.gitignore` 文件内容相似。

### 精简基础镜像

在 Dockerfile 中为 `FROM` 指令选择较小的镜像可以帮助减小最终镜像的大小。[Alpine 镜像](https://hub.docker.com/_/alpine)是一个最小化 Docker 镜像的好例子，它提供了您对 Linux 容器所期望的所有操作系统实用程序。

还有[特殊的 `scratch` 镜像](https://hub.docker.com/_/scratch)，它完全不包含任何内容。例如，这对于创建静态链接二进制文件的镜像很有用。

### 多阶段构建

[多阶段构建](/build/building/multi-stage/)可以使您的构建运行更快，因为阶段可以并行运行。它也可以使您的最终结果更小。以这样的方式编写 Dockerfile：最终运行时阶段使用尽可能小的基础镜像，只包含程序运行所需的资源。

也可以使用 Dockerfile `COPY --from` 指令[从其他镜像或阶段复制资源](/build/building/multi-stage/#name-your-build-stages)。这种技术可以减少最终阶段中的层数以及这些层的大小。

### 在构建中获取远程文件

如果可能，您应该在构建中从远程位置获取文件，而不是将文件捆绑到构建上下文中。在 Docker Build Cloud 服务器上直接下载文件更好，因为这可能比使用构建上下文传输文件更快。

您可以在构建过程中使用 [Dockerfile `ADD` 指令](/reference/dockerfile/#add)或在 `RUN` 指令中使用 `wget` 和 `rsync` 等工具来获取远程文件。

### 多线程工具

您在构建指令中使用的一些工具可能默认不利用多核。一个这样的例子是 `make`，默认情况下使用单线程，除非您指定 `make --jobs=<n>` 选项。对于涉及此类工具的构建步骤，请尝试检查是否可以通过并行化优化执行。
