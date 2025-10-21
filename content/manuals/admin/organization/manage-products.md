---
title: 管理 Docker 产品
weight: 45
description: 了解如何为您的组织管理 Docker 产品的访问权限和使用情况
keywords: organization, tools, products, product access, organization management
---

{{< summary-bar feature_name="Admin orgs" >}}

在本节中，您将了解如何为您的组织管理 Docker 产品的访问权限和查看使用情况。有关每个产品的更详细信息，包括如何设置和配置它们，请参阅以下手册：

- [Docker Desktop](../../desktop/_index.md)
- [Docker Hub](../../docker-hub/_index.md)
- [Docker Build Cloud](../../build-cloud/_index.md)
- [Docker Scout](../../scout/_index.md)
- [Testcontainers Cloud](https://testcontainers.com/cloud/docs/#getting-started)

## 管理组织的产品访问权限

默认情况下，您订阅中包含的 Docker 产品对所有用户开放访问。有关订阅中包含产品的概述，请参阅 [Docker 订阅和功能](/manuals/subscription/details.md)。

{{< tabs >}}
{{< tab name="Docker Desktop" >}}

### 管理 Docker Desktop 访问权限

要管理 Docker Desktop 访问权限：

1. [强制登录](/manuals/enterprise/security/enforce-sign-in/_index.md)。
1. [手动管理](./members.md)成员或使用[自动配置](/manuals/enterprise/security/provisioning/_index.md)。

强制登录后，只有属于您组织的用户在登录后才能使用 Docker Desktop。

{{< /tab >}}
{{< tab name="Docker Hub" >}}

### 管理 Docker Hub 访问权限

要管理 Docker Hub 访问权限，请登录 [Docker Home](https://app.docker.com/) 并配置[仓库访问管理](/manuals/enterprise/security/hardened-desktop/registry-access-management.md)或[镜像访问管理](/manuals/enterprise/security/hardened-desktop/image-access-management.md)。

{{< /tab >}}
{{< tab name="Docker Build Cloud" >}}

### 管理 Docker Build Cloud 访问权限

要初始设置和配置 Docker Build Cloud，请登录 [Docker Build Cloud](https://app.docker.com/build) 并按照屏幕上的说明操作。

要管理 Docker Build Cloud 访问权限：

1. 以组织所有者身份登录 [Docker Build Cloud](http://app.docker.com/build)。
1. 选择 **Account settings**（账户设置）。
1. 选择 **Lock access to Docker Build Account**（锁定对 Docker Build 账户的访问）。

{{< /tab >}}
{{< tab name="Docker Scout" >}}

### 管理 Docker Scout 访问权限

要初始设置和配置 Docker Scout，请登录 [Docker Scout](https://scout.docker.com/) 并按照屏幕上的说明操作。

要管理 Docker Scout 访问权限：

1. 以组织所有者身份登录 [Docker Scout](https://scout.docker.com/)。
1. 选择您的组织，然后选择 **Settings**（设置）。
1. 要管理哪些仓库启用了 Docker Scout 分析，请选择 **Repository settings**（仓库设置）。有关更多信息，请参阅[仓库设置](../../scout/explore/dashboard.md#repository-settings)。
1. 要管理 Docker Scout 在 Docker Desktop 上用于本地镜像的访问权限，请使用[设置管理](/manuals/enterprise/security/hardened-desktop/settings-management/_index.md)，并将 `sbomIndexing` 设置为 `false` 以禁用，或设置为 `true` 以启用。

{{< /tab >}}
{{< tab name="Testcontainers Cloud" >}}

### 管理 Testcontainers Cloud 访问权限

要初始设置和配置 Testcontainers Cloud，请登录 [Testcontainers Cloud](https://app.testcontainers.cloud/) 并按照屏幕上的说明操作。

要管理 Testcontainers Cloud 访问权限：

1. 登录 [Testcontainers Cloud](https://app.testcontainers.cloud/) 并选择 **Account**（账户）。
1. 选择 **Settings**（设置），然后选择 **Lock access to Testcontainers Cloud**（锁定对 Testcontainers Cloud 的访问）。

{{< /tab >}}
{{< /tabs >}}

## 监控组织的产品使用情况

要查看 Docker 产品的使用情况：

- Docker Desktop：在 [Docker Home](https://app.docker.com/) 中查看 **Insights**（洞察）页面。有关更多详细信息，请参阅[洞察](./insights.md)。
- Docker Hub：在 Docker Hub 中查看[**使用情况**页面](https://hub.docker.com/usage)。
- Docker Build Cloud：在 [Docker Build Cloud](http://app.docker.com/build) 中查看 **Build minutes**（构建分钟数）页面。
- Docker Scout：在 Docker Scout 中查看[**仓库设置**页面](https://scout.docker.com/settings/repos)。
- Testcontainers Cloud：在 Testcontainers Cloud 中查看[**账单**页面](https://app.testcontainers.cloud/dashboard/billing)。

如果您的使用量或许可证数量超过订阅限额，您可以[扩展订阅](../../subscription/scale.md)以满足您的需求。
