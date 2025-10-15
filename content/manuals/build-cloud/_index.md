---
title: Docker Build Cloud
weight: 20
description: 了解 Docker Build Cloud 文档,帮助你在本地和 CI 环境中更快地构建容器镜像
keywords: build, cloud, cloud build, remote builder
params:
  sidebar:
    group: Products
aliases:
  - /build/cloud/faq/
  - /build/cloud/
---

{{< summary-bar feature_name="Docker Build Cloud" >}}

Docker Build Cloud 是一项服务,让你能够在本地和 CI 环境中更快地构建容器镜像。
构建任务运行在为你的工作负载优化配置的云基础设施上,无需额外配置。
该服务使用远程构建缓存,确保在任何地方、对所有团队成员都能实现快速构建。

## Docker Build Cloud 工作原理

使用 Docker Build Cloud 与运行常规构建没有什么不同。你仍然像往常一样通过 `docker buildx build` 命令调用构建。
不同之处在于构建的执行位置和方式。

默认情况下,当你执行构建命令时,构建会在本地的 BuildKit 实例上运行(它与 Docker 守护进程打包在一起)。
而使用 Docker Build Cloud 时,你会将构建请求发送到云端远程运行的 BuildKit 实例。
所有数据在传输过程中都经过加密。

远程构建器会执行构建步骤,并将生成的构建产物发送到你指定的目标位置。
例如,发送回你本地的 Docker Engine 镜像存储,或者推送到镜像仓库。

相比本地构建,Docker Build Cloud 提供了以下优势:

- 更快的构建速度
- 共享的构建缓存
- 原生的多平台构建

最棒的是:你无需操心构建器或基础设施的管理。只需连接到你的构建器,即可开始构建。
为组织配置的每个云构建器都完全隔离在单独的 Amazon EC2 实例上,配有专用的 EBS 卷用于构建缓存,
并且传输过程加密。这意味着不同云构建器之间不会共享进程或数据。

> [!NOTE]
>
> Docker Build Cloud 目前仅在美国东部地区可用。欧洲和亚洲的用户可能会比北美用户体验到更高的延迟。
>
> 多区域构建器支持已在规划中。

## 获取 Docker Build Cloud

要开始使用 Docker Build Cloud,请先[创建 Docker 账户](/accounts/create-account/)。
有两种方式可以访问 Docker Build Cloud:

- 拥有免费个人账户的用户可以选择 7 天免费试用,试用结束后可订阅继续使用。
要开始免费试用,请登录 [Docker Build Cloud 控制台](https://app.docker.com/build/)并按照屏幕指引操作。
- 所有付费 Docker 订阅的用户都可以使用 Docker Build Cloud,它包含在 Docker 产品套件中。
详情请参阅 [Docker 订阅与功能](/manuals/subscription/details.md)。

完成注册并创建构建器后,请继续[在本地环境中设置构建器](./setup.md)。

有关 Docker Build Cloud 相关的角色和权限信息,请参阅
[角色与权限](/manuals/enterprise/security/roles-and-permissions.md#docker-build-cloud-permissions)。
