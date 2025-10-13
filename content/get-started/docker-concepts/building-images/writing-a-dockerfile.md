---
title: 编写 Dockerfile
keywords: concepts, build, images, container, docker desktop
description: 本文将介绍如何使用 Dockerfile 创建镜像。
summary: |
  掌握 Dockerfile 的编写实践对高效利用容器技术至关重要，
  它能提升应用可靠性，并更好地支持 DevOps 与 CI/CD 流程。
  在本指南中，你将学习如何编写一个 Dockerfile，如何选择基础镜像，
  以及如何编写安装软件与复制必要文件等构建指令。
weight: 2
aliases: 
 - /guides/docker-concepts/building-images/writing-a-dockerfile/
---

{{< youtube-embed Jx8zoIhiP4c >}}

## 概念解析

Dockerfile 是一个基于文本的文档，用于创建容器镜像。它向镜像构建器提供一系列指令，说明需要运行哪些命令、复制哪些文件、设置何种启动命令等。

例如，下面这个 Dockerfile 可以构建一个开箱即用的 Python 应用：

```dockerfile
FROM python:3.13
WORKDIR /usr/local/app

# 安装应用依赖
COPY requirements.txt ./
RUN pip install --no-cache-dir -r requirements.txt

# 拷贝源代码
COPY src ./src
EXPOSE 8080

# 创建应用用户，避免以 root 身份运行
RUN useradd app
USER app

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8080"]
```

### 常用指令

`Dockerfile` 中最常见的指令包括：

- `FROM <image>` —— 指定要扩展的基础镜像。
- `WORKDIR <path>` —— 指定工作目录，即镜像内后续复制文件与执行命令的路径。
- `COPY <host-path> <image-path>` —— 从宿主机复制文件到容器镜像中。
- `RUN <command>` —— 在构建期间执行命令。
- `ENV <name> <value>` —— 设置环境变量，供运行中的容器使用。
- `EXPOSE <port-number>` —— 为镜像声明一个希望暴露的端口（元数据）。
- `USER <user-or-uid>` —— 设置后续指令的默认用户。
- `CMD ["<command>", "<arg1>"]` —— 设置基于该镜像启动的容器默认执行的命令。

若需了解全部指令与更深入的说明，请参阅 [Dockerfile 参考](https://docs.docker.com/engine/reference/builder/)。

## 动手试试

与前述示例类似，一个 Dockerfile 通常遵循以下流程：

1. 选择基础镜像
2. 安装应用依赖
3. 复制相关源码与/或二进制文件
4. 配置最终镜像

接下来，我们用一个简单的 Node.js 应用来演示如何编写 Dockerfile。即使你不熟悉 JavaScript，也不影响跟随本指南操作。

### 准备环境

[下载此 ZIP 包](https://github.com/docker/getting-started-todo-app/archive/refs/heads/build-image-from-scratch.zip) 并将内容解压到本地目录。

如果不想下载 ZIP，可以克隆仓库 https://github.com/docker/getting-started-todo-app 并切换到 `build-image-from-scratch` 分支。

### 创建 Dockerfile

现在你已获取项目代码，可以开始创建 `Dockerfile`。

1. [下载并安装](https://www.docker.com/products/docker-desktop/) Docker Desktop。

2. 浏览项目结构。

   打开 `getting-started-todo-app/app/` 目录。你会注意到已有一个 `Dockerfile`。
   它只是一个纯文本文件，可用任意文本或代码编辑器打开。

3. 删除现有的 `Dockerfile`。

   在本练习中，我们假设你从零开始，并会手动创建一个新的 `Dockerfile`。

4. 在 `getting-started-todo-app/app/` 目录中新建名为 `Dockerfile` 的文件。

    > **关于 Dockerfile 的扩展名**
    >
    > 请注意，`Dockerfile` 没有文件扩展名。有些编辑器可能会自动添加扩展名
    > （或对没有扩展名发出警告）。

5. 在 `Dockerfile` 中添加以下内容以指定基础镜像：

    ```dockerfile
    FROM node:22-alpine
    ```

6. 使用 `WORKDIR` 指令设置工作目录。这会指定后续命令的执行位置，以及在容器镜像内复制文件的目标目录。

    ```dockerfile
    WORKDIR /app
    ```

7. 使用 `COPY` 指令将你机器上的项目文件全部复制到容器镜像中：

    ```dockerfile
    COPY . .
    ```

8. 使用 `yarn` 包管理器安装应用依赖。通过 `RUN` 指令执行以下命令：

    ```dockerfile
    RUN yarn install --production
    ```

9. 使用 `CMD` 指令指定容器的默认启动命令：

    ```dockerfile
    CMD ["node", "./src/index.js"]
    ```
    至此，你应得到如下完整的 Dockerfile：

    ```dockerfile
    FROM node:22-alpine
    WORKDIR /app
    COPY . .
    RUN yarn install --production
    CMD ["node", "./src/index.js"]
    ```

> **该 Dockerfile 还未达到生产可用水准**
>
> 需要说明的是，这个 Dockerfile（刻意）尚未遵循全部最佳实践。
> 它可以完成构建，但构建速度与镜像安全性还有提升空间。
>
> 继续阅读，了解如何最大化构建缓存命中率、以非 root 用户运行、以及多阶段构建等实践。

> **用 `docker init` 快速容器化新项目**
>
> `docker init` 会分析你的项目并快速生成 Dockerfile、`compose.yaml` 与 `.dockerignore`，
> 帮助你快速上手。由于本文聚焦 Dockerfile 的学习，这里不直接使用它。
> 但你可以在[此处了解更多](/engine/reference/commandline/init/)。

## 延伸阅读

想进一步了解如何编写 Dockerfile，可参考：

* [Dockerfile 参考](/reference/dockerfile/)
* [Dockerfile 最佳实践](/develop/develop-images/dockerfile_best-practices/)
* [基础镜像](/build/building/base-images/)
* [Docker Init 入门](/reference/cli/docker/init/)

## 下一步

现在你已经创建了一个 Dockerfile 并掌握了基础知识，下一步是学习如何构建、打标签并推送镜像。

{{< button text="构建、打标签并发布镜像" url="build-tag-and-publish-an-image" >}}

