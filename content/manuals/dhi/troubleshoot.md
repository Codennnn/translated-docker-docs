---
title: 故障排除
description: 解决构建、运行或调试 Docker 加固镜像时的常见问题，如非 root 行为、缺失 shell 和端口访问等。
weight: 40
tags: [Troubleshooting]
keywords: 故障排除加固镜像, docker 调试容器, 非 root 权限问题, 缺失 shell 错误, 无包管理器
---

以下是在迁移到或使用 Docker 加固镜像（DHIs）时可能遇到的常见问题，以及推荐的解决方案。

## 常规调试

Docker 加固镜像针对安全性和运行时性能进行了优化。因此，它们通常不包含 shell 或标准调试工具。对基于 DHI 的容器进行故障排除的推荐方法是使用 [Docker Debug](./how-to/debug.md)。

Docker Debug 允许您：

- 将临时调试容器附加到现有容器。
- 使用 shell 和熟悉的工具，如 `curl`、`ps`、`netstat` 和 `strace`。
- 在可写的临时层中根据需要安装其他工具，该层在会话结束后消失。

## 权限

DHIs 默认以非 root 用户身份运行以增强安全性。这可能导致访问文件或目录时出现权限问题。确保您的应用程序文件和运行时目录由预期的 UID/GID 拥有或具有适当的权限。

要查找 DHI 以哪个用户身份运行，请检查镜像的文档或使用 `docker inspect` 命令查看镜像配置。

## 特权端口

非 root 容器默认无法绑定到低于 1024 的端口。这是由容器运行时和内核强制执行的（特别是在 Kubernetes 和 Docker Engine < 20.10 中）。

在容器内，将您的应用程序配置为监听非特权端口（1025 或更高）。例如，`docker run -p 80:8080 my-image` 将容器中的端口 8080 映射到主机上的端口 80，允许您无需 root 权限即可访问它。

## 无 shell

运行时 DHIs 省略了交互式 shell，如 `sh` 或 `bash`。如果您的构建或工具假设 shell 存在（例如，对于 `RUN` 指令），请在较早的构建阶段使用镜像的 `dev` 变体，并将最终构件复制到运行时镜像中。

要查找 DHI 具有哪个 shell（如果有），请在 Docker Hub 上检查该镜像的仓库页面。有关更多信息，请参阅[查看镜像变体详细信息](./how-to/explore.md#view-image-variant-details)。

当您需要访问正在运行的容器的 shell 时，还可以使用 [Docker Debug](./how-to/debug.md)。

## 入口点差异

与 Docker 官方镜像（DOIs）或其他社区镜像相比，DHIs 可能定义了不同的入口点。

要查找 DHI 的 ENTRYPOINT 或 CMD，请在 Docker Hub 上检查该镜像的仓库页面。有关更多信息，请参阅[查看镜像变体详细信息](./how-to/explore.md#view-image-variant-details)。

## 无包管理器

运行时 Docker 加固镜像为了安全性和最小攻击面进行了精简。因此，它们不包含包管理器，如 `apk` 或 `apt`。这意味着您无法直接在运行时镜像中安装额外软件。

如果您的构建或应用程序设置需要安装包（例如，编译代码、安装运行时依赖项或添加诊断工具），请在构建阶段使用镜像的 `dev` 变体。然后，仅将必要的构件复制到最终的运行时镜像中。
