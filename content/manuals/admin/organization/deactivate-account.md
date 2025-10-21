---
title: 停用组织
description: 了解如何停用 Docker 组织以及所需的前提步骤。
keywords: 删除, 停用组织, 账户, 组织管理, 管理控制台, 取消订阅
weight: 42
aliases:
- /docker-hub/deactivate-account/
---

{{< summary-bar feature_name="General admin" >}}

了解如何停用 Docker 组织，包括所需的前提步骤。有关停用用户账户的信息，请参阅[停用用户账户](../../accounts/deactivate-user-account.md)。

> [!WARNING]
>
> 停用账户后，所有使用您的 Docker 账户或组织账户的 Docker 产品和服务将无法访问。

## 先决条件

在停用组织之前，您必须完成以下所有步骤：

- 下载您要保留的任何镜像和标签：
  `docker pull -a <image>:<tag>`。
- 如果您有活跃的 Docker 订阅，请[将其降级为免费订阅](../../subscription/change.md)。
- 移除组织内的所有其他成员。
- 取消关联您的 [GitHub 和 Bitbucket 账户](../../docker-hub/repos/manage/builds/link-source.md#unlink-a-github-user-account)。
- 对于 Business 组织，请[移除您的 SSO 连接](/manuals/enterprise/security/single-sign-on/manage.md#remove-an-organization)。

## 停用

您可以使用管理控制台或 Docker Hub 停用您的组织。

> [!WARNING]
>
> 此操作无法撤销。在停用组织之前，请确保您已收集了需要的所有数据。

1. 登录 [Docker Home](https://app.docker.com) 并选择您要停用的组织。
1. 选择 **管理控制台**，然后选择 **停用**。如果 **停用** 按钮不可用，请确认您已完成所有[先决条件](#prerequisites)。
1. 输入组织名称以确认停用。
1. 选择 **停用组织**。
