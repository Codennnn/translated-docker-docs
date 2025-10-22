---
title: 调试 Docker 加固镜像容器
linkTitle: 调试容器
weight: 60
keywords: 调试, 加固镜像, DHI, 故障排除, 临时容器, docker 调试
description: 学习如何在本地或生产环境中使用 Docker 调试功能来排查 Docker 加固镜像 (DHI) 问题。
keywords: docker 调试, 临时容器, 非 root 容器, 加固容器镜像, 调试安全容器
---

{{< summary-bar feature_name="Docker Hardened Images" >}}

Docker 加固镜像 (DHI) 优先考虑极简性和安全性，这意味着它们有意省略了许多常见的调试工具（如 shell 或包管理器）。这使得直接故障排除变得困难，而且容易引入风险。为了解决这个问题，您可以使用 [Docker 调试](../../../reference/cli/docker/debug.md)，这是一个安全的工作流程，可以临时将工具丰富的调试容器附加到正在运行的服务或镜像，而无需修改原始镜像。

本指南展示了如何在开发期间本地调试 Docker 加固镜像。您也可以使用 `--host` 选项远程调试容器。

以下示例使用了镜像化的 `dhi-python:3.13` 镜像，但同样的步骤适用于任何镜像。

## 步骤 1：从加固镜像运行容器

从一个基于 DHI 的容器开始，该容器模拟了一个问题：

```console
$ docker run -d --name myapp <YOUR_ORG>/dhi-python:3.13 python -c "import time; time.sleep(300)"
```

此容器不包含 shell 或 `ps`、`top`、`cat` 等工具。

如果您尝试：

```console
$ docker exec -it myapp sh
```

您会看到：

```console
exec: "sh": executable file not found in $PATH
```

## 步骤 2：使用 Docker 调试检查容器

使用 `docker debug` 命令将临时、工具丰富的调试容器附加到正在运行的实例。

```console
$ docker debug myapp
```

从这里，您可以检查正在运行的进程、网络状态或挂载的文件。

例如，要检查正在运行的进程：

```console
$ ps aux
```

使用以下命令退出调试会话：

```console
$ exit
```

## 下一步

Docker 调试可帮助您排查加固容器问题，而不会影响原始镜像的完整性。由于调试容器是临时的且独立的，因此可以避免在生产环境中引入安全风险。

如果您遇到与权限、端口、缺少 shell 或包管理器相关的问题，请参阅[排查 Docker 加固镜像问题](../troubleshoot.md)以获取推荐的解决方案和解决方法。
