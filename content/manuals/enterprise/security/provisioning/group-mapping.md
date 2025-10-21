---
title: 组映射
description: 通过同步身份提供商组与 Docker 团队来自动化团队成员资格
keywords: 组映射, SCIM, Docker 管理员, 管理员, 安全, 团队管理, 用户配置, 身份提供商
aliases:
- /admin/company/settings/group-mapping/
- /admin/organization/security-settings/group-mapping/
- /docker-hub/group-mapping/
- /security/for-admins/group-mapping/
- /security/for-admins/provisioning/group-mapping/
weight: 30
---

{{< summary-bar feature_name="SSO" >}}

组映射自动将您的身份提供商（IdP）中的用户组与 Docker 组织中的团队进行同步。例如，当您将开发人员添加到 IdP 中的"backend-team"组时，他们会自动被添加到 Docker 中对应的团队。

本页面解释组映射的工作原理以及如何设置组映射。

> [!TIP]
>
> 组映射非常适合将用户添加到多个组织或一个组织内的多个团队。如果您不需要设置多组织或多团队分配，SCIM [用户级属性](scim.md#set-up-role-mapping)可能更适合您的需求。

## 先决条件

开始之前，您必须具备：

- 已为您的组织配置 SSO
- 对 Docker Home 和身份提供商的管理员访问权限

## 组映射工作原理

组映射通过以下关键组件使您的 Docker 团队与 IdP 组保持同步：

- 身份验证流程：当用户通过 SSO 登录时，您的 IdP 会与 Docker 共享用户属性，包括电子邮件、姓名和组成员资格。
- 自动更新：Docker 使用这些属性创建或更新用户配置文件，并根据 IdP 组的更改管理团队分配。
- 唯一标识：Docker 使用电子邮件地址作为唯一标识符，因此每个 Docker 帐户必须具有唯一的电子邮件地址。
- 团队同步：用户在 Docker 中的团队成员资格会自动反映在您的 IdP 组中所做的更改。

## 设置组映射

组映射设置涉及配置您的身份提供商以与 Docker 共享组信息。这需要：

- 使用 Docker 的命名格式在 IdP 中创建组
- 配置属性，使您的 IdP 在身份验证期间发送组数据
- 将用户添加到适当的组
- 测试连接以确保组正确同步

您可以仅将组映射与 SSO 一起使用，或者将组映射与 SSO 和 SCIM 一起使用以增强用户生命周期管理。

### 组命名格式

使用格式 `organization:team` 在您的 IdP 中创建组。

例如：

- 对于 "moby" 组织中的 "developers" 团队：`moby:developers`
- 对于多组织访问：`moby:backend` 和 `whale:desktop`

当组同步时，如果团队不存在，Docker 会自动创建团队。

### 支持的属性

| 属性 | 描述 |
|:--------- | :---------- |
| `id` | 组的唯一 ID，UUID 格式。此属性为只读。 |
| `displayName` | 组的名称，遵循组映射格式：`organization:team`。 |
| `members` | 属于此组的用户列表。 |
| `members(x).value` | 属于此组的用户的唯一 ID。成员通过 ID 引用。 |

## 使用 SSO 配置组映射

将组映射与使用 SAML 身份验证方法的 SSO 连接一起使用。

> [!NOTE]
>
> 使用 SSO 的组映射不支持 Azure AD (OIDC) 身份验证方法。这些配置不需要 SCIM。

{{< tabs >}}
{{< tab name="Okta" >}}

您的身份提供商的用户界面可能与以下步骤略有不同。请参考 [Okta 文档](https://help.okta.com/oie/en-us/content/topics/apps/define-group-attribute-statements.htm)进行验证。

要设置组映射：

1. 登录 Okta 并打开您的应用程序。
1. 导航到应用程序的 **SAML Settings**（SAML 设置）页面。
1. 在 **Group Attribute Statements (optional)**（组属性语句（可选））部分，按如下方式配置：
   - **Name**（名称）：`groups`
   - **Name format**（名称格式）：`Unspecified`（未指定）
   - **Filter**（过滤器）：`Starts with`（以...开头）+ `organization:`，其中 `organization` 是您的组织名称
   此过滤器选项将过滤掉与您的 Docker 组织无关的组。
1. 选择 **Directory**（目录），然后选择 **Groups**（组）来创建您的组。
1. 使用 `organization:team` 格式添加您的组，该格式匹配 Docker 中您的组织和团队的名称。
1. 将用户分配给您创建的组。

下次与 Docker 同步组时，您的用户将映射到您定义的 Docker 组。

{{< /tab >}}
{{< tab name="Entra ID" >}}

您的身份提供商的用户界面可能与以下步骤略有不同。请参考 [Entra ID 文档](https://learn.microsoft.com/en-us/azure/active-directory/app-provisioning/customize-application-attributes)进行验证。

要设置组映射：

1. 登录 Entra ID 并打开您的应用程序。
1. 选择 **Manage**（管理），然后选择 **Single sign-on**（单点登录）。
1. 选择 **Add a group claim**（添加组声明）。
1. 在组声明部分，选择 **Groups assigned to the application**（分配给应用程序的组），源属性为 **Cloud-only group display names (Preview)**（仅云组显示名称（预览））。
1. 选择 **Advanced options**（高级选项），然后选择 **Filter groups**（过滤组）选项。
1. 按如下方式配置属性：
   - **Attribute to match**（要匹配的属性）：`Display name`（显示名称）
   - **Match with**（匹配方式）：`Contains`（包含）
   - **String**（字符串）：`:`
1. 选择 **Save**（保存）。
1. 选择 **Groups**（组）、**All groups**（所有组），然后选择 **New group**（新建组）来创建您的组。
1. 将用户分配给您创建的组。

下次与 Docker 同步组时，您的用户将映射到您定义的 Docker 组。

{{< /tab >}}
{{< /tabs >}}

## 使用 SCIM 配置组映射

将组映射与 SCIM 一起使用以实现更高级的用户生命周期管理。开始之前，请确保您已[设置 SCIM](./scim.md#enable-scim)。

{{< tabs >}}
{{< tab name="Okta" >}}

您的身份提供商的用户界面可能与以下步骤略有不同。请参考 [Okta 文档](https://help.okta.com/en-us/Content/Topics/users-groups-profiles/usgp-enable-group-push.htm)进行验证。

要设置您的组：

1. 登录 Okta 并打开您的应用程序。
1. 选择 **Applications**（应用程序），然后选择 **Provisioning**（配置）和 **Integration**（集成）。
1. 选择 **Edit**（编辑）以在连接上启用组，然后选择 **Push groups**（推送组）。
1. 选择 **Save**（保存）。保存此配置会将 **Push Groups**（推送组）选项卡添加到您的应用程序。
1. 通过导航到 **Directory**（目录）并选择 **Groups**（组）来创建您的组。
1. 使用 `organization:team` 格式添加您的组，该格式匹配 Docker 中您的组织和团队的名称。
1. 将用户分配给您创建的组。
1. 返回 **Integration**（集成）页面，然后选择 **Push Groups**（推送组）选项卡，打开您可以控制和管理如何配置组的视图。
1. 选择 **Push Groups**（推送组），然后选择 **Find groups by rule**（按规则查找组）。
1. 按如下方式配置组规则：
    - 输入规则名称，例如 `Sync groups with Docker Hub`（与 Docker Hub 同步组）
    - 按名称匹配组，例如以 `docker:` 开头或包含 `:` 用于多组织
    - 如果启用 **Immediately push groups by rule**（立即按规则推送组），则一旦组或组分配发生更改，就会发生同步。如果您不想手动推送组，请启用此选项。

在 **Pushed Groups**（推送的组）列的 **By rule**（按规则）下找到您的新规则。匹配该规则的组列在右侧的组表中。

要从此表推送组：

1. 选择 **Group in Okta**（Okta 中的组）。
1. 选择 **Push Status**（推送状态）下拉菜单。
1. 选择 **Push Now**（立即推送）。

{{< /tab >}}
{{< tab name="Entra ID" >}}

您的身份提供商的用户界面可能与以下步骤略有不同。请参考 [Entra ID 文档](https://learn.microsoft.com/en-us/azure/active-directory/app-provisioning/customize-application-attributes)进行验证。

在配置组映射之前，请完成以下操作：

1. 登录 Entra ID 并转到您的应用程序。
1. 在您的应用程序中，选择 **Provisioning**（配置），然后选择 **Mappings**（映射）。
1. 选择 **Provision Microsoft Entra ID Groups**（配置 Microsoft Entra ID 组）。
1. 选择 **Show advanced options**（显示高级选项），然后选择 **Edit attribute list**（编辑属性列表）。
1. 将 `externalId` 类型更新为 `reference`（引用），然后选择 **Multi-Value**（多值）复选框并选择引用的对象属性 `urn:ietf:params:scim:schemas:core:2.0:Group`。
1. 选择 **Save**（保存），然后选择 **Yes**（是）进行确认。
1. 转到 **Provisioning**（配置）。
1. 将 **Provision Status**（配置状态）切换为 **On**（开），然后选择 **Save**（保存）。

接下来，设置组映射：

1. 转到应用程序概述页面。
1. 在 **Provision user accounts**（配置用户帐户）下，选择 **Get started**（入门）。
1. 选择 **Add user/group**（添加用户/组）。
1. 使用 `organization:team` 格式创建您的组。
1. 将组分配给配置组。
1. 选择 **Start provisioning**（开始配置）以开始同步。

要验证，请选择 **Monitor**（监控），然后选择 **Provisioning logs**（配置日志）以查看您的组是否已成功配置。在您的 Docker 组织中，您可以检查组是否已正确配置以及成员是否已添加到适当的团队。

{{< /tab >}}
{{< /tabs >}}

完成后，通过 SSO 登录 Docker 的用户会自动添加到 IdP 中映射的组织和团队。

> [!TIP]
>
> [启用 SCIM](scim.md)以利用自动用户配置和取消配置。如果您不启用 SCIM，用户只会被自动配置。您必须手动取消配置他们。
