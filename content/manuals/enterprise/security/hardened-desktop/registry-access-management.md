---
title: 仓库访问管理
description: 使用仓库访问管理控制对已批准容器仓库的访问，确保 Docker Desktop 安全使用
keywords: 仓库访问管理, 容器仓库, 安全控制, docker 商业版, 管理员控制
tags: [admin]
aliases:
 - /desktop/hardened-desktop/registry-access-management/
 - /admin/organization/registry-access/
 - /docker-hub/registry-access-management/
 - /security/for-admins/registry-access-management/
 - /security/for-admins/hardened-desktop/registry-access-management/
weight: 30
---

{{< summary-bar feature_name="Registry access management" >}}

仓库访问管理（RAM）允许管理员控制开发人员通过 Docker Desktop 可以访问哪些容器仓库。这种 DNS 级别的过滤确保开发人员只能从已批准的仓库拉取和推送镜像，从而提高供应链安全性。

RAM 适用于所有类型的仓库，包括云服务、本地仓库和仓库镜像。您可以允许任何主机名或域名，但必须在白名单中包含重定向域名（如某些仓库的 `s3.amazonaws.com`）。

## 支持的仓库

仓库访问管理适用于任何容器仓库，包括：

- Docker Hub（默认允许）
- 云仓库：Amazon ECR、Google Container Registry、Azure Container Registry
- 基于 Git 的仓库：GitHub Container Registry、GitLab Container Registry
- 本地解决方案：Nexus、Artifactory、Harbor
- 仓库镜像：包括 Docker Hub 镜像

## 先决条件

在配置仓库访问管理之前，您必须：

- [强制登录](/manuals/enterprise/security/enforce-sign-in/_index.md)以确保用户使用您的组织身份进行身份验证
- 使用[个人访问令牌 (PAT)](/manuals/security/access-tokens.md)进行身份验证（不支持组织访问令牌）
- 拥有 Docker Business 订阅

> [!IMPORTANT]
>
> 仓库访问管理仅在用户使用组织凭据登录 Docker Desktop 时生效。

## 配置仓库权限

要配置仓库权限：

1. 登录 [Docker Home](https://app.docker.com) 并从左上角的账户下拉菜单中选择您的组织。
1. 选择 **管理控制台**，然后选择 **仓库访问**。
1. 使用**切换开关**启用仓库访问。默认情况下，Docker Hub 在仓库列表中已启用。
1. 要添加其他仓库，选择 **添加仓库** 并提供**仓库地址**和**仓库昵称**。
1. 选择 **创建**。您最多可以添加 100 个仓库。
1. 确认您的仓库出现在仓库列表中，然后选择 **保存更改**。

更改可能需要最多 24 小时才能生效。要更快应用更改，请让开发人员退出并重新登录 Docker Desktop。

> [!IMPORTANT]
>
> 从 Docker Desktop 4.36 开始，如果开发人员属于多个具有不同 RAM 策略的组织，则仅强制执行配置文件中第一个组织的策略。

> [!TIP]
>
> RAM 限制也适用于通过 URL 获取内容的 Dockerfile `ADD` 指令。使用带有 URL 的 `ADD` 时，请将受信任的仓库域包含在您的白名单中。
><br><br>
> RAM 专为容器仓库设计，不适用于像包镜像或存储服务这样的通用 URL。添加过多域可能会导致错误或达到系统限制。

## 验证限制是否生效

用户使用其组织凭据登录 Docker Desktop 后，仓库访问管理立即生效。

当用户尝试从被阻止的仓库拉取时：

```console
$ docker pull blocked-registry.com/image:tag
Error response from daemon: registry access to blocked-registry.com is not allowed
```

允许的仓库访问正常工作：

```console
$ docker pull allowed-registry.com/image:tag
# 拉取成功
```

仓库限制适用于所有 Docker 操作，包括拉取、推送以及引用外部仓库的构建。

## 仓库限制和平台约束

仓库访问管理具有以下限制和平台特定行为：

- 最大白名单大小：每个组织 100 个仓库或域
- 基于 DNS 的过滤：限制在主机名级别工作，而非 IP 地址
- 需要重定向域：必须包含仓库重定向到的所有域（CDN 端点、存储服务）
- Windows 容器：默认情况下，Windows 镜像操作不受限制。在 Docker Desktop 设置中打开 **为 Windows Docker 守护进程使用代理** 以应用限制
- WSL 2 要求：需要 Linux 内核 5.4 或更高版本，限制适用于所有 WSL 2 发行版

## 构建和部署限制

以下场景不受仓库访问管理限制：

- 使用 Kubernetes 驱动程序的 Docker buildx
- 使用自定义 docker-container 驱动程序的 Docker buildx
- 某些 Docker Debug 和 Kubernetes 镜像拉取（即使 Docker Hub 被阻止）
- 如果源仓库受限，之前由仓库镜像缓存的镜像可能仍被阻止

## 安全绕过注意事项

用户可能通过以下方式绕过仓库访问管理：

- 本地代理或 DNS 操作
- 退出 Docker Desktop（除非强制登录）
- Docker Desktop 控制之外的网络级修改

为了最大化安全效果：

- [强制登录](/manuals/enterprise/security/enforce-sign-in/_index.md)以防止通过退出登录绕过
- 实施额外的网络级控制以获得全面保护
- 将仓库访问管理作为更广泛安全策略的一部分使用

## 仓库白名单最佳实践

- 包含所有仓库域：某些仓库会重定向到多个域。对于 AWS ECR，包括：

    ```text
    your-account.dkr.ecr.us-west-2.amazonaws.com
    amazonaws.com
    s3.amazonaws.com
    ```

- 定期维护白名单：
    - 定期删除未使用的仓库
    - 根据需要添加新批准的仓库
    - 更新可能已更改的域名
    - 通过 Docker Desktop 分析监控仓库使用情况
- 测试配置更改：
    - 在白名单更新后验证仓库访问
    - 检查是否包含所有必要的重定向域
    - 确保开发工作流程不被中断
    - 与[增强容器隔离](/manuals/enterprise/security/hardened-desktop/enhanced-container-isolation/_index.md)结合使用以获得全面保护
    