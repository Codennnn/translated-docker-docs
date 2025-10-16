---
title: 管理未关联的机器
description: 了解如何使用 Docker 管理控制台管理未关联的机器
keywords: unassociated machines, insights, manage users, enforce sign-in
sitemap: false
pagefind_exclude: true
noindex: true
params:
    sidebar:
        group: Enterprise
---

{{% restricted title="关于未关联的机器" %}}
未关联的机器是私有功能，并非对所有账户开放。
{{% /restricted %}}

Docker 管理员可以识别、查看并管理那些很可能属于其组织、但当前尚未与用户账户关联的 Docker Desktop 机器。通过这种自助方式，你可以更好地了解全组织范围内的 Docker Desktop 使用情况，并在无需 IT 介入的情况下简化用户接入流程。

## 先决条件

- Docker Business 或 Team 订阅
- 你在 Docker 组织中的所有者权限

## 关于未关联的机器

未关联的机器是指：基于使用模式，Docker 判断这些 Docker Desktop 实例很可能属于你的组织，但其使用者并未使用你组织内的账户登录 Docker Desktop。

## Docker 如何识别未关联的机器

Docker 基于遥测数据来识别哪些机器可能属于你的组织：

- 域名匹配：用户使用与你的组织相关的电子邮件域名登录
- 仓库模式：分析容器仓库的访问行为，以判断是否为组织内使用

## 查看未关联的机器

查看未关联机器的详细信息：

1. Sign in to the [Admin Console](https://app.docker.com/admin) and select
your organization.
1. In **User management**, select **Unassociated**.

机器列表将显示：

- 机器 ID（由 Docker 生成的标识符）
- 用于预测用户是否属于你组织的仓库地址
- 用户邮箱（仅当用户在使用期间已登录 Docker Desktop 时显示）
- Docker Desktop 版本
- 操作系统（OS）
- 最近活动日期
- 登录强制状态

你可以：

- 将列表导出为 CSV
- 对单台或多台机器执行操作

## 为未关联的机器启用登录强制

> [!NOTE]
>
> 未关联机器的登录强制与通过 `registry.json` 与配置文件提供的[组织级登录强制](/enterprise/security/enforce-sign-in/)不同。这里的登录强制仅要求用户完成登录，以便管理员识别正在使用该机器的人员，用户可以使用任意邮箱地址登录。若你需要更严格的安全控制，仅允许已属于你组织的用户登录，请参见[强制登录](/enterprise/security/enforce-sign-in/)。

登录强制可帮助你识别组织中正在使用未关联机器的用户。启用后，这些机器上的用户必须登录 Docker Desktop。用户登录后，其邮箱地址会显示在 **Unassociated** 列表中，你即可将其添加到组织。

你可以通过两种方式启用登录强制：

- 针对组织内所有未关联的机器
- 针对单台未关联的机器

> [!IMPORTANT]
>
> 登录强制在 Docker Desktop 重启后才会生效。用户可继续使用 Docker Desktop，直到下次重启。

### 为所有未关联的机器启用登录强制

为所有未关联机器启用登录强制：

1. Sign in to the [Admin Console](https://app.docker.com/admin) and select
your organization.
1. In **User management**, select **Unassociated**.
1. Turn on the **Enforce sign-in** toggle.
1. In the pop-up modal, select **Require sign-in** to confirm.

所有未关联机器的 **Sign-in required** 状态将更新为 **Yes**。

> [!NOTE]
>
> 当你为所有未关联机器启用登录强制后，未来检测到的新机器也会自动启用登录强制。登录强制要求 Docker Desktop 版本为 4.41 或更高版本。较旧版本的用户不会收到登录提示，并可在升级前继续正常使用 Docker Desktop；在他们升级到 4.41 或更高版本之前，其状态将显示为 **Pending**。

### 为单台未关联的机器启用登录强制

为单台未关联机器启用登录强制：

1. Sign in to the [Admin Console](https://app.docker.com/admin) and select
your organization.
1. In **User management**, select **Unassociated**.
1. Locate the machine you want to enable sign-in enforcement for.
1. Select the **Actions** menu and choose **Turn on sign-in enforcement**.
1. In the pop-up modal, select **Require sign-in** to confirm.

该机器的 **Sign-in required** 状态将更新为 **Yes**。

> [!NOTE]
>
> 登录强制要求 Docker Desktop 版本为 4.41 或更高版本。较旧版本的用户不会收到登录提示，并可在升级前继续正常使用 Docker Desktop；在他们升级到 4.41 或更高版本之前，其状态将显示为 **Pending**。

### 用户登录后会发生什么

启用登录强制后：

1. Users must restart Docker Desktop. Enforcement only takes effect after
restart.
1. When users open Docker Desktop, they see a sign-in prompt. They must sign
in to continue using Docker Desktop.
1. User email addresses appear in the **Unassociated** list.
1. You can add users to your organization.

用户登录后即可继续使用 Docker Desktop，即使他们尚未被添加到你的组织中。

## 将未关联的机器加入你的组织

当组织内的用户未登录就使用 Docker 时，其机器会出现在 **Unassociated** 列表中。你可以通过两种方式将这些用户添加到组织：

- 自动添加：
    - 自动配置：如果你已验证域名并启用自动配置，使用匹配邮箱域名登录的用户会被自动添加至你的组织。关于域名验证与自动配置，参见[域名管理](/manuals/enterprise/security/domain-management.md)。
    - SSO 用户配置：如果你已配置 SSO，并启用了[即时供应（Just-in-Time）](/manuals/enterprise/security/provisioning/just-in-time.md)，通过 SSO 登录的用户将被自动添加至你的组织。
- 手动添加：如果你未启用自动配置或 SSO，或用户的邮箱域名与已配置的域名不匹配，其邮箱会显示在 **Unassociated** 列表中，你可以选择直接将其添加。

> [!NOTE]
>
> 如果在添加用户时组织可用席位不足，系统会弹出提示，建议你 **Get more seats**。

### 添加单个用户

1. Sign in to the [Admin Console](https://app.docker.com/admin) and select
your organization.
1. In **User management**, select **Unassociated**.
1. Locate the machine you want to add to your organization.
1. Select the **Actions** menu and choose **Add to organization**.
1. In the pop-up modal, select **Add user**.

### 批量添加用户

1. Sign in to the [Admin Console](https://app.docker.com/admin) and select
your organization.
1. In **User management**, select **Unassociated**.
1. Use the **checkboxes** to select the machines you want to add to your
organizations.
1. Select the **Add to organization** button.
1. In the pop-up modal, select **Add users** to confirm.

## 关闭登录强制

### 对所有未关联的机器关闭登录强制

1. Sign in to the [Admin Console](https://app.docker.com/admin) and select
your organization.
1. In **User management**, select **Unassociated**.
1. Turn off the **Enforce sign-in** toggle.
1. In the pop-up modal, select **Turn off sign-in requirement** to confirm.

所有未关联机器的 **Sign-in required** 状态将更新为 **No**。

### 对指定未关联的机器关闭登录强制

1. Sign in to the [Admin Console](https://app.docker.com/admin) and select
your organization.
1. In **User management**, select **Unassociated**.
1. Locate the machine you want to disable sign-in enforcement for.
1. Select the **Actions** menu and choose **Turn off sign-in enforcement**.
1. In the pop-up modal, select **Turn off sign-in requirement** to confirm.

该机器的 **Sign-in required** 状态将更新为 **No**。
