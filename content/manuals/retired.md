---
title: Docker 已弃用和退役的产品和功能
linkTitle: 已弃用的产品和功能
description: |
  探索 Docker 已弃用和退役的功能、产品以及开源项目，包括已转移的工具和已归档的倡议的详细信息。
params:
  sidebar:
    group: Products
aliases:
  - /cloud/
  - /cloud/aci-compose-features/
  - /cloud/aci-container-features/
  - /cloud/aci-integration/
  - /cloud/ecs-architecture/
  - /cloud/ecs-compose-examples/
  - /cloud/ecs-compose-features/
  - /cloud/ecs-integration/
  - /engine/context/aci-integration/
  - /engine/context/ecs-integration/
  - /machine/
  - /machine/drivers/hyper-v/
  - /machine/get-started/
  - /machine/install-machine/
  - /machine/overview/
  - /registry/
  - /registry/compatibility/
  - /registry/configuration/
  - /registry/deploying/
  - /registry/deprecated/
  - /registry/garbage-collection/
  - /registry/help/
  - /registry/insecure/
  - /registry/introduction/
  - /registry/notifications/
  - /registry/recipes/
  - /registry/recipes/apache/
  - /registry/recipes/nginx/
  - /registry/recipes/osx-setup-guide/
  - /registry/spec/
  - /registry/spec/api/
  - /registry/spec/auth/
  - /registry/spec/auth/jwt/
  - /registry/spec/auth/oauth/
  - /registry/spec/auth/scope/
  - /registry/spec/auth/token/
  - /registry/spec/deprecated-schema-v1/
  - /registry/spec/implementations/
  - /registry/spec/json/
  - /registry/spec/manifest-v2-1/
  - /registry/spec/manifest-v2-2/
  - /registry/spec/menu/
  - /registry/storage-drivers/
  - /registry/storage-drivers/azure/
  - /registry/storage-drivers/filesystem/
  - /registry/storage-drivers/gcs/
  - /registry/storage-drivers/inmemory/
  - /registry/storage-drivers/oss/
  - /registry/storage-drivers/s3/
  - /registry/storage-drivers/swift/
  - /toolbox/
  - /toolbox/overview/
  - /toolbox/toolbox_install_mac/
  - /toolbox/toolbox_install_windows/
  - /desktop/features/dev-environments/
  - /desktop/features/dev-environments/create-dev-env/
  - /desktop/features/dev-environments/set-up/
  - /desktop/features/dev-environments/share/
  - /desktop/features/dev-environments/dev-cli/
  - /desktop/dev-environments/
---

本文档概述了 Docker 已弃用、退役或转移的功能、产品和开源项目。

> [!NOTE]
>
> 本页面不涵盖已弃用和移除的 Docker Engine 功能。
> 如需查看已弃用 Docker Engine 功能的详细列表，请参考
> [Docker Engine 已弃用功能文档](/manuals/engine/deprecated.md)。

## 产品和功能

Docker, Inc. 不再为这些已弃用或退役的功能提供支持。已转移给第三方的项目将继续从其新的维护者那里获得更新。

### Docker Machine

Docker Machine 是一个用于在各种平台（包括虚拟机和云提供商）上配置和管理 Docker 主机的工具。它已不再维护，建议用户直接在支持的平台上使用 [Docker Desktop](/manuals/desktop/_index.md) 或 [Docker Engine](/manuals/engine/_index.md)。Machine 创建和配置主机的方法已被与 Docker Desktop 更紧密集成的现代化工作流所取代。

### Docker Toolbox

Docker Toolbox 用于无法运行 Docker Desktop 的旧系统。它将 Docker Machine、Docker Engine 和 Docker Compose 打包到一个安装程序中。Toolbox 已不再维护，在当前系统中已被 [Docker Desktop](/manuals/desktop/_index.md) 有效替代。在旧文档或社区教程中偶尔会出现对 Docker Toolbox 的引用，但不建议用于新安装。

### Docker Cloud 集成

Docker 之前为 Amazon 的弹性容器服务（ECS）和 Azure 容器实例（ACI）提供集成，以简化容器工作流。这些集成已被弃用，用户现在应该依赖原生云工具或第三方解决方案来管理其工作负载。向平台特定或通用编排工具的转变减少了对专门 Docker Cloud 集成的需求。

您仍可以在 [Compose CLI 仓库](https://github.com/docker-archive/compose-cli/tree/main/docs) 中查看这些集成的相关文档。

### Docker Enterprise Edition

Docker Enterprise Edition (EE) 是 Docker 用于部署和管理大规模容器环境的商业平台。它于 2019 年被 Mirantis 收购，寻求企业级功能的用户现在可以探索 Mirantis Kubernetes Engine 或 Mirantis 提供的其他产品。Docker EE 中的许多技术和功能已被吸收到 Mirantis 产品线中。

> [!NOTE]  
> 有关 Docker 当前提供的企业级功能的信息，
> 请参阅 [Docker Business 订阅](/manuals/subscription/details.md#docker-business)。

### Docker Data Center 和 Docker Trusted Registry

Docker Data Center (DDC) 是一个总称，涵盖了 Docker Universal Control Plane (UCP) 和 Docker Trusted Registry (DTR)。这些组件为企业环境中的容器、安全和仓库服务管理提供了全栈解决方案。在 Docker Enterprise 收购后，它们现在属于 Mirantis 产品组合。仍遇到 DDC、UCP 或 DTR 引用的用户应参考 Mirantis 的文档以获取现代等效产品的指导。

### Dev Environments

Dev Environments 是 Docker Desktop 中引入的一项功能，允许开发者快速启动开发环境。它已被弃用并从 Docker Desktop 4.42 及更高版本中移除。可以通过 Docker Compose 或创建针对特定项目需求的自定义配置来实现类似的工作流。

## 开源项目

Docker 最初维护的几个开源项目已被归档、停用或转移给其他维护者或组织。

### Registry（现为 CNCF Distribution）

Docker Registry 是容器镜像仓库的开源实现。它于 2019 年捐赠给云原生计算基金会（CNCF），并以"Distribution"的名称进行维护。它仍然是管理和分发容器镜像的基石。

[CNCF Distribution](https://github.com/distribution/distribution)

### Docker Compose v1（已被 Compose v2 替代）

Docker Compose v1（`docker-compose`）是一个基于 Python 的工具，用于定义多容器应用程序，现已被 Compose v2（`docker compose`）取代，后者用 Go 编写并与 Docker CLI 集成。Compose v1 已不再维护，用户应迁移到 Compose v2。

[Compose v2 文档](/manuals/compose/_index.md)

### InfraKit

InfraKit 是一个开源工具包，旨在管理声明式基础设施并自动化容器部署。它已被归档，建议用户探索 Terraform 等工具进行基础设施配置和编排。

[InfraKit GitHub 仓库](https://github.com/docker/infrakit)

### Docker Notary（现为 CNCF Notary）

Docker Notary 是一个用于签名和验证容器内容真实性的系统。它于 2017 年捐赠给 CNCF，并继续以"Notary"的名称进行开发。寻求安全内容验证的用户应参考 CNCF Notary 项目。

[CNCF Notary](https://github.com/notaryproject/notary)

### SwarmKit

SwarmKit 通过为容器部署提供编排功能来驱动 Docker Swarm 模式。虽然 Swarm 模式仍然可用，但开发已放缓，转而支持基于 Kubernetes 的解决方案。评估容器编排选项的个人应调查 SwarmKit 是否满足现代工作负载需求。

[SwarmKit GitHub 仓库](https://github.com/docker/swarmkit)
