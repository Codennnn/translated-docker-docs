---
title: 组织入门指南
weight: 20
description: 开始为您的 Docker Team 或 Business 组织进行入门设置。
keywords: 商业版, 团队版, 组织, 入门, 入门指南, 管理控制台, 组织管理,
toc_min: 1
toc_max: 3
aliases:
- /docker-hub/onboard/
- /docker-hub/onboard-team/
- /docker-hub/onboard-business/
---

{{< summary-bar feature_name="Admin orgs" >}}

了解如何使用管理控制台或 Docker Hub 为您的组织进行入门设置。

组织入门设置包括：

- 识别用户，帮助您分配订阅席位
- 邀请成员和所有者加入您的组织
- 为您的组织提供安全的身份验证和授权
- 强制 Docker Desktop 登录以确保安全最佳实践

这些操作帮助管理员获得用户活动的可见性并
强制执行安全设置。组织成员登录后也会获得更高的拉取
限制和其他权益。

## 先决条件

在开始组织入门设置之前，请确保您：

- 拥有 Docker Team 或 Business 订阅。有关更多详情，请参阅
[Docker 订阅和功能](/manuals/subscription/details.md)。

  > [!NOTE]
  >
  > 购买自助服务订阅时，屏幕上的说明
  会指导您创建组织。如果您已通过
  Docker Sales 购买订阅但尚未创建
  组织，请参阅[创建组织](/manuals/admin/organization/orgs.md)。

- 熟悉[管理概述](../_index.md)中的 Docker 概念和术语。

## 使用引导式设置进行入门

管理控制台提供引导式设置，帮助您
为组织进行入门设置。引导式设置的步骤包括基本的入门
任务。如果您想在引导式设置之外进行入门，
请参阅[推荐的入门步骤](/manuals/admin/organization/onboard.md#recommended-onboarding-steps)。

要使用引导式设置进行入门，
请导航到[管理控制台](https://app.docker.com)，然后
在左侧导航中选择 **引导式设置**。

引导式设置将引导您完成以下入门步骤：

- **邀请您的团队**：邀请所有者和成员。
- **管理用户访问**：添加和验证域，使用 SSO 管理用户，并
强制 Docker Desktop 登录。
- **Docker Desktop 安全**：配置镜像访问管理、注册表
访问管理和设置管理。

## 推荐的入门步骤

### 步骤一：识别您的 Docker 用户

识别您的用户有助于您有效分配席位，并确保他们
获得您的 Docker 订阅权益。

1. 识别组织中的 Docker 用户。
   - 如果您的组织使用设备管理软件，如 MDM 或 Jamf，
   您可以使用设备管理软件帮助识别 Docker 用户。
   有关详情，请参阅您的设备管理软件文档。您可以通过
   检查每台用户机器上的以下位置是否安装了 Docker Desktop 来识别 Docker 用户：
      - Mac: `/Applications/Docker.app`
      - Windows: `C:\Program Files\Docker\Docker`
      - Linux: `/opt/docker-desktop`
   - 如果您的组织不使用设备管理软件或您的
   用户尚未安装 Docker Desktop，您可以调查用户以
   识别谁在使用 Docker Desktop。
1. 要求用户将其 Docker 账户的电子邮件地址更新为与
您组织域关联的地址，或使用该电子邮件创建新账户。
   - 要更新账户的电子邮件地址，指示用户登录
   到 [Docker Hub](https://hub.docker.com)，并将电子邮件地址更新为
   您组织域中的电子邮件地址。
   - 要创建新账户，指示用户
   [注册](https://hub.docker.com/signup)使用与
   您组织域关联的电子邮件地址。
1. 识别与您组织域关联的 Docker 账户：
   - 询问您的 Docker 销售代表或
   [联系销售](https://www.docker.com/pricing/contact-sales/)获取使用
   您组织域中电子邮件地址的 Docker 账户列表。

### 步骤二：邀请所有者

所有者可以帮助您为组织进行入门设置和管理。

创建组织时，您是唯一的所有者。添加
额外所有者是可选的。

要添加所有者，请邀请用户并为他们分配所有者角色。有关
更多详情，请参阅[邀请成员](/manuals/admin/organization/members.md)和
[角色和权限](/manuals/enterprise/security/roles-and-permissions.md)。

### 步骤三：邀请成员

当您将用户添加到组织时，您可以获得对其
活动的可见性，并且可以强制执行安全设置。您的成员
登录后也会获得更高的拉取限制和其他组织范围内的权益。

要添加成员，请邀请用户并为他们分配成员角色。
有关更多详情，请参阅[邀请成员](/manuals/admin/organization/members.md)和
[角色和权限](/manuals/enterprise/security/roles-and-permissions.md)。

### 步骤四：使用 SSO 和 SCIM 管理用户访问

配置 SSO 和 SCIM 是可选的，仅适用于 Docker Business
订阅者。要将 Docker Team 订阅升级为 Docker Business
订阅，请参阅[更改您的订阅](/manuals/subscription/change.md)。

使用您的身份提供商 (IdP) 管理成员并通过
SSO 和 SCIM 自动将其配置到 Docker。有关更多详情，请参阅以下内容：

   - [配置 SSO](/manuals/enterprise/security/single-sign-on/configure.md)
   在用户通过您的身份提供商登录到 Docker 时进行身份验证并添加成员。
   - 可选。
   [强制 SSO](/manuals/enterprise/security/single-sign-on/connect.md) 确保
   当用户登录到 Docker 时，必须使用 SSO。

     > [!NOTE]
     >
     > 强制单点登录 (SSO) 和强制 Docker Desktop 登录
     是不同的功能。有关更多详情，请参阅
     > [强制登录与强制单点登录 (SSO) 的区别](/manuals/enterprise/security/enforce-sign-in/_index.md#enforcing-sign-in-versus-enforcing-single-sign-on-sso)。

   - [配置 SCIM](/manuals/enterprise/security/provisioning/scim.md) 通过
   您的身份提供商自动配置、添加和取消配置成员到 Docker。

### 步骤五：强制 Docker Desktop 登录

默认情况下，您的组织成员可以在不登录
的情况下使用 Docker Desktop。当用户不作为您组织的成员登录时，他们不会
获得[组织订阅的权益](../../subscription/details.md)，
并且他们可以绕过 [Docker 的安全功能](/manuals/enterprise/security/hardened-desktop/_index.md)。

根据您组织的
Docker 配置，有多种方法可以强制登录：
- [注册表项方法（仅限 Windows）](/manuals/enterprise/security/enforce-sign-in/methods.md#registry-key-method-windows-only)
- [`.plist` 方法（仅限 Mac）](/manuals/enterprise/security/enforce-sign-in/methods.md#plist-method-mac-only)
- [`registry.json` 方法（所有平台）](/manuals/enterprise/security/enforce-sign-in/methods.md#registryjson-method-all)

### 步骤六：管理 Docker Desktop 安全

Docker 提供以下安全功能来管理您组织的
安全态势：

- [镜像访问管理](/manuals/enterprise/security/hardened-desktop/image-access-management.md)：控制您的开发人员可以从 Docker Hub 拉取哪些类型的镜像。
- [注册表访问管理](/manuals/enterprise/security/hardened-desktop/registry-access-management.md)：定义您的开发人员可以访问哪些注册表。
- [设置管理](/manuals/enterprise/security/hardened-desktop/settings-management.md)：为您的用户设置和控制 Docker Desktop 设置。

## 进一步阅读

- [管理 Docker 产品](./manage-products.md) 以配置访问权限和查看使用情况。
- 配置[强化 Docker Desktop](/desktop/hardened-desktop/) 以提高您组织在容器化开发方面的安全态势。
- [管理您的域](/manuals/enterprise/security/domain-management.md) 以确保您域中的所有 Docker 用户都是您组织的一部分。

您的 Docker 订阅提供更多附加功能。要了解更多，
请参阅 [Docker 订阅和功能](/subscription/details/)。
