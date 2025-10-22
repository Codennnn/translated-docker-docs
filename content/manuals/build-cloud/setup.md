---
title: Docker Build Cloud 设置
linkTitle: 设置
weight: 10
description: Docker Build Cloud 入门指南
keywords: 构建, 云构建
aliases:
  - /build/cloud/setup/
---

在开始使用 Docker Build Cloud 之前，必须先将构建器添加到您的本地环境中。

## 先决条件

要开始使用 Docker Build Cloud，您需要：

- 下载并安装 Docker Desktop 4.26.0 或更高版本。
- 在 [Docker Build Cloud 控制台](https://app.docker.com/build/)上创建云构建器。
  - 创建构建器时，为其选择一个名称（例如，`default`）。在下面的 CLI 步骤中，您将使用此名称作为 `BUILDER_NAME`。

### 不使用 Docker Desktop 的情况下使用 Docker Build Cloud

要在不使用 Docker Desktop 的情况下使用 Docker Build Cloud，您必须下载并安装支持 Docker Build Cloud（`cloud` 驱动程序）的 Buildx 版本。您可以在[此仓库](https://github.com/docker/buildx-desktop)的发布页面找到兼容的 Buildx 二进制文件。

如果您计划使用 `docker compose build` 命令通过 Docker Build Cloud 进行构建，您还需要一个支持 Docker Build Cloud 的 Docker Compose 版本。您可以在[此仓库](https://github.com/docker/compose-desktop)的发布页面找到兼容的 Docker Compose 二进制文件。

## 操作步骤

您可以使用 CLI 通过 `docker buildx create` 命令添加云构建器，或者使用 Docker Desktop 设置 GUI。

{{< tabs >}}
{{< tab name="CLI" >}}

1. 登录到您的 Docker 账户。

   ```console
   $ docker login
   ```

2. 添加云构建器端点。

   ```console
   $ docker buildx create --driver cloud <ORG>/<BUILDER_NAME>
   ```

   将 `<ORG>` 替换为您的 Docker 组织的 Docker Hub 命名空间（如果您使用个人账户，则替换为您的用户名），并将 `<BUILDER_NAME>` 替换为您在控制台中创建构建器时选择的名称。

   这将创建一个名为 `cloud-ORG-BUILDER_NAME` 的云构建器本地实例。

   > [!NOTE]
   >
   > 如果您的组织是 `acme` 且您将构建器命名为 `default`，请使用：
   > ```console
   > $ docker buildx create --driver cloud acme/default
   > ```


{{< /tab >}}
{{< tab name="Docker Desktop" >}}

1. 使用 Docker Desktop 中的**登录**按钮登录到您的 Docker 账户。

2. 打开 Docker Desktop 设置并导航到**构建器**选项卡。

3. 在**可用构建器**下，选择**连接到构建器**。

{{< /tab >}}
{{< /tabs >}}

该构建器原生支持 `linux/amd64` 和 `linux/arm64` 架构。这为您提供了一个高性能的构建集群，用于原生构建多平台镜像。

## 防火墙配置

要在防火墙后使用 Docker Build Cloud，请确保您的防火墙允许到以下地址的流量：

- 3.211.38.21
- https://auth.docker.io
- https://build-cloud.docker.com
- https://hub.docker.com

## 下一步

- 查看[使用 Docker Build Cloud 构建](usage.md)了解如何使用 Docker Build Cloud 的示例。
- 查看[在 CI 中使用 Docker Build Cloud](ci.md)了解如何在 CI 系统中使用 Docker Build Cloud 的示例。
