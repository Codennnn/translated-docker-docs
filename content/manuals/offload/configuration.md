---
title: 配置 Docker Offload
linktitle: 配置
weight: 20
description: 了解如何配置 Docker Offload 的构建设置。
keywords: 云端, 配置, 设置, 云构建器, GPU, 磁盘分配, 私有资源, 防火墙
---

要使用 Docker Offload，您必须在 Docker Desktop 中启动它。更多详细信息，请参阅 [Docker Offload 快速入门](/offload/quickstart/)。

除了整个组织的设置外，Docker Offload 中的云构建器设置还可以通过 Docker Offload 仪表板中的 **Offload 设置** 进行进一步配置。

> [!NOTE]
>
> 要查看 Docker Offload 的使用情况并配置计费，请参阅 [Docker Offload 使用和计费](/offload/usage/)。

## Offload 设置

Docker 主页中的 **Offload 设置** 页面允许您为组织中的云构建器配置磁盘分配、私有资源访问和防火墙设置。

要查看 **Offload 设置** 页面：

1. 访问 [Docker 主页](https://app.docker.com/)。
2. 选择要管理 Docker Offload 的账户。
3. 选择 **Offload** > **Offload 设置**。

以下部分介绍了可用的设置。

### 磁盘分配

**磁盘分配** 设置允许您控制有多少可用存储专用于构建缓存。较低的分配会增加活动构建的可用存储。

调整 **磁盘分配** 滑块以指定用于构建缓存的存储百分比。

任何更改都会立即生效。

> [!TIP]
>
> 如果您构建非常大的镜像，请考虑为缓存分配较少的存储空间。

### 构建缓存空间

您的订阅包含以下构建缓存空间：

| 订阅类型 | 构建缓存空间 |
|----------|--------------|
| 个人版   | 不适用       |
| 专业版   | 50GB         |
| 团队版   | 100GB        |
| 商业版   | 200GB        |

要获取更多构建缓存空间，请 [升级您的订阅](/manuals/subscription/change.md)。

### 私有资源访问

私有资源访问允许云构建器从私有资源拉取镜像和包。当构建依赖于自托管工件仓库或私有 OCI 注册表时，此功能非常有用。

例如，如果您的组织在私有网络上托管私有 [PyPI](https://pypi.org/) 仓库，Docker Build Cloud 默认将无法访问它，因为云构建器未连接到您的私有网络。

要启用云构建器访问您的私有资源，请输入您私有资源的主机名和端口，然后选择 **添加**。

#### 身份验证

如果您的内部工件需要身份验证，请确保在构建之前或构建期间对仓库进行身份验证。对于 npm 或 PyPI 的内部包仓库，使用 [构建密钥](/manuals/build/building/secrets.md) 在构建期间进行身份验证。对于内部 OCI 注册表，请在构建前使用 `docker login` 进行身份验证。

请注意，如果您使用需要身份验证的私有注册表，您需要在构建前使用 `docker login` 进行两次身份验证。这是因为云构建器需要先向 Docker 进行身份验证以使用云构建器，然后再向私有注册表进行身份验证。

```console
$ echo $DOCKER_PAT | docker login docker.io -u <username> --password-stdin
$ echo $REGISTRY_PASSWORD | docker login registry.example.com -u <username> --password-stdin
$ docker build --builder <cloud-builder> --tag registry.example.com/<image> --push .
```

### 防火墙

防火墙设置允许您将云构建器的出口流量限制为特定的 IP 地址。这通过限制构建器的外部网络出口来帮助增强安全性。

1. 选择 **启用防火墙：将云构建器出口限制为特定的公共 IP 地址**。
2. 输入您要允许的 IP 地址。
3. 选择 **添加** 以应用限制。

