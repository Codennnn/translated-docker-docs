---
title: 通过策略强制使用 Docker 加固镜像
linktitle: 强制镜像使用
description: 了解如何为 Docker 加固镜像使用 Docker Scout 的镜像策略。
weight: 50
keywords: docker scout 策略, 强制镜像合规, 容器安全策略, 镜像来源, 漏洞策略检查
---

{{< summary-bar feature_name="Docker Hardened Images" >}}

镜像化 Docker 加固镜像（DHI）仓库会自动启用 [Docker Scout](/scout/)，让您无需额外配置即可开始强制实施镜像安全与合规策略。借助 Docker Scout 策略，您可以定义并应用规则，确保环境中仅使用经过批准且安全的镜像（例如基于 DHI 的镜像）。

Docker Scout 内置策略评估功能，可实时监控镜像合规性，将检查集成到 CI/CD 工作流中，并持续维护镜像安全与来源的统一标准。

## 查看现有策略

要查看已镜像的 DHI 仓库当前应用的策略：

1. 在 [Docker Hub](https://hub.docker.com) 中进入已镜像的 DHI 仓库。
2. 选择 **View on Scout**。

   这会打开 [Docker Scout 控制台](https://scout.docker.com)，您可以在其中查看当前激活的策略以及镜像是否满足策略条件。

Docker Scout 会在新镜像推送时自动评估策略合规性。每条策略都包含合规结果以及指向受影响镜像和层的链接。

## 为基于 DHI 的镜像创建策略

为确保使用 Docker 加固镜像构建的镜像保持安全，您可以为自己的仓库量身定制 Docker Scout 策略。这些策略有助于强制执行安全标准，例如防止高危漏洞、要求使用最新的基础镜像，或验证关键元数据的存在。

策略会在镜像推送到仓库时进行评估，让您能够跟踪合规性、接收偏差通知，并将策略检查集成到 CI/CD 流水线中。

### 示例：为基于 DHI 的镜像创建策略

本示例演示如何创建一条策略，要求组织中的所有镜像都必须以 Docker 加固镜像作为基础。这样可以确保您的应用构建在安全、精简且可用于生产环境的镜像之上。

#### 步骤 1：在 Dockerfile 中使用 DHI 基础镜像

创建一个 Dockerfile，使用已镜像的 Docker 加固镜像仓库作为基础。例如：

```dockerfile
# Dockerfile
FROM ORG_NAME/dhi-python:3.13-alpine3.21

ENTRYPOINT ["python", "-c", "print('Hello from a DHI-based image')"]
```

#### 步骤 2：构建并推送镜像

打开终端，进入包含 Dockerfile 的目录。然后构建镜像并推送到您的 Docker Hub 仓库：

```console
$ docker build \
  --push \
  -t YOUR_ORG/my-dhi-app:v1 .
```

#### 步骤 3：启用 Docker Scout

要为您的组织和仓库启用 Docker Scout，请在终端运行以下命令：

```console
$ docker login
$ docker scout enroll YOUR_ORG
$ docker scout repo enable --org YOUR_ORG YOUR_ORG/my-dhi-app
```

#### 步骤 4：创建策略

1. 进入 [Docker Scout 控制台](https://scout.docker.com)。
2. 选择您的组织，然后进入 **策略（Policies）**。
3. 选择 **添加策略（Add policy）**。
4. 在 **Approved Base Images Policy** 处选择 **配置（Configure）**。
5. 为策略起一个合规的名称，例如 **Approved DHI Base Images**。
6. 在 **Approved base image sources** 中删除默认项。
7. 在 **Approved base image sources** 中添加已批准的基础镜像源。本示例中，使用通配符（`*`）允许所有已镜像的 DHI 仓库，格式为 `docker.io/ORG_NAME/dhi-*`。请将 `ORG_NAME` 替换为您的组织名称。
8. 选择 **保存策略（Save policy）**。

#### 步骤 5：评估策略合规性

1. 进入 [Docker Scout 控制台](https://scout.docker.com)。
2. 选择您的组织，然后进入 **镜像（Images）**。
3. 找到您的镜像 `YOUR_ORG/my-dhi-app:v1`，在 **合规性（Compliance）** 列中选择链接。

这里会显示该镜像的策略合规结果，包括是否满足 **Approved DHI Base Images** 策略的要求。

您现在可以在 CI 中[评估策略合规性](/scout/policy/ci/)了。