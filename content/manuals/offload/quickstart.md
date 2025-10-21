---
title: Docker Offload 快速入门
linktitle: 快速入门
weight: 10
description: 了解如何使用 Docker Offload 在本地和 CI 环境中更快地构建和运行容器镜像。
keywords: 云, 快速入门, 云模式, Docker Desktop, GPU 支持, 云构建器, 使用
---

{{< summary-bar feature_name="Docker Offload" >}}

本快速入门指南将帮助您开始使用 Docker Offload。Docker Offload 通过将资源密集型任务卸载到云端，让您能够更快地构建和运行容器镜像。它提供了一个与本地 Docker Desktop 体验一致的云环境。

## 步骤 1：注册并订阅 Docker Offload 以获取访问权限

要访问 Docker Offload，您必须[注册](https://www.docker.com/products/docker-offload/)并订阅服务。

## 步骤 2：启动 Docker Offload

> [!NOTE]
>
> 订阅 Docker Offload 后，首次启动 Docker Desktop 并登录时，系统可能会提示您启动 Docker Offload。如果您通过此提示启动 Docker Offload，可以跳过以下步骤。请注意，您可以随时使用以下步骤启动 Docker Offload。


1. 启动 Docker Desktop 并登录。
2. 打开终端并运行以下命令来启动 Docker Offload：

   ```console
   $ docker offload start
   ```

3. 系统提示时，选择用于 Docker Offload 的账户。该账户将消耗 Docker Offload 的使用量。

4. 系统提示时，选择是否启用 GPU 支持。如果选择启用 GPU 支持，Docker Offload 将在配备 NVIDIA L4 GPU 的实例中运行，这对于机器学习或计算密集型工作负载非常有用。

   > [!NOTE]
   >
   > 启用 GPU 支持会消耗更多预算。有关更多详细信息，请参阅 [Docker Offload 使用指南](/offload/usage/)。

当 Docker Offload 启动后，您将在 Docker Desktop 仪表板标题栏中看到云图标 ({{< inline-image
src="./images/cloud-mode.png" alt="Offload 模式图标" >}})，并且 Docker Desktop 仪表板会呈现紫色。
您可以在终端中运行 `docker offload status` 命令来检查 Docker Offload 的状态。

## 步骤 3：使用 Docker Offload 运行容器

启动 Docker Offload 后，Docker Desktop 会连接到一个与本地体验一致的云环境。当您运行构建或容器时，它们会在远程执行，但行为与本地容器完全相同。

要验证 Docker Offload 是否正常工作，请运行一个容器：

```console
$ docker run --rm hello-world
```

如果您启用了 GPU 支持，还可以运行支持 GPU 的容器：

```console
$ docker run --rm --gpus all hello-world
```

如果 Docker Offload 正常工作，您将在终端输出中看到 `Hello from Docker!`。

## 步骤 4：停止 Docker Offload

当您完成 Docker Offload 的使用后，可以停止它。停止后，您将在本地构建镜像和运行容器。

```console
$ docker offload stop
```

要再次启动 Docker Offload，请运行 `docker offload start` 命令。

## 下一步

- [配置 Docker Offload](configuration.md)。
- 尝试使用 [Docker Model Runner](../ai/model-runner/_index.md) 或 [Compose](../ai/compose/models-and-compose.md) 通过 Docker Offload 运行 AI 模型。