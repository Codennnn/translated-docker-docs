---
title: SCIM 用户配置
linkTitle: SCIM
description: 了解跨域身份管理系统的工作原理以及如何进行设置。
keywords: SCIM, SSO, 用户配置, 用户取消配置, 角色映射, 分配用户
aliases:
  - /security/for-admins/scim/
  - /docker-hub/scim/
  - /security/for-admins/provisioning/scim/
weight: 20
---

{{< summary-bar feature_name="SSO" >}}

使用跨域身份管理系统（SCIM）为您的 Docker 组织自动化用户管理。SCIM 可以自动配置和取消配置用户、同步团队成员资格，并使您的 Docker 组织与身份提供商保持同步。

本页面介绍如何使用 SCIM 为 Docker 自动化用户配置和取消配置。

## 先决条件

开始之前，您必须具备：

- 已为您的组织配置 SSO
- 对 Docker Home 和身份提供商的管理员访问权限

## SCIM 工作原理

SCIM 通过您的身份提供商为 Docker 自动化用户配置和取消配置。启用 SCIM 后，在您的身份提供商中分配给 Docker 应用程序的任何用户都会自动配置并添加到您的 Docker 组织中。当用户从您身份提供商中的 Docker 应用程序中移除时，SCIM 会停用他们并将其从您的 Docker 组织中移除。

除了配置和移除之外，SCIM 还会同步在您的身份提供商中所做的个人资料更新（如姓名更改）。您可以将 SCIM 与 Docker 默认的即时（JIT）配置一起使用，也可以在禁用 JIT 的情况下单独使用 SCIM。

SCIM 自动化以下操作：

- 创建用户
- 更新用户个人资料
- 移除和停用用户
- 重新激活用户
- 组映射

> [!NOTE]
>
> SCIM 仅管理在启用 SCIM 后通过您的身份提供商配置的用户。它无法移除在设置 SCIM 之前手动添加到您的 Docker 组织的用户。
><br><br>
> 要移除这些用户，请从您的 Docker 组织中手动删除它们。
> 有关更多信息，请参阅[管理组织成员](/manuals/admin/organization/members.md)。

## 支持的属性

SCIM 使用属性（姓名、电子邮件等）在您的身份提供商和 Docker 之间同步用户信息。在您的身份提供商中正确映射这些属性可确保用户配置顺利进行，并防止在使用单点登录时出现重复用户帐户等问题。

Docker 支持以下 SCIM 属性：

| 属性    | 描述 |
|:---------------------------------------------------------------|:-------------------------------------------------------------------------------------------|
| `userName`             | 用户的主要电子邮件地址，用作唯一标识符 |
| `name.givenName` | 用户的名字 |
| `name.familyName` | 用户的姓氏 |
| `active` | 指示用户是启用还是禁用，设置为 "false" 可取消配置用户 |

有关支持的属性和 SCIM 的其他详细信息，请参阅 [Docker Hub API SCIM 参考](/reference/api/hub/latest/#tag/scim)。

> [!IMPORTANT]
>
> 默认情况下，Docker 对 SSO 使用即时（JIT）配置。如果启用了 SCIM，JIT 值仍然优先，并将覆盖 SCIM 设置的属性值。
> 为避免冲突，请确保您的 JIT 属性值与您的 SCIM 值匹配。
><br><br>
> 或者，您可以禁用 JIT 配置，仅依赖 SCIM。
> 有关详细信息，请参阅[即时配置](just-in-time.md)。

## 在 Docker 中启用 SCIM

要启用 SCIM：

1. 登录 [Docker Home](https://app.docker.com)。
1. 选择 **Admin Console**（管理控制台），然后选择 **SSO and SCIM**（SSO 和 SCIM）。
1. 在 **SSO connections**（SSO 连接）表中，选择您的连接的 **Actions**（操作）图标，然后选择 **Setup SCIM**（设置 SCIM）。
1. 复制 **SCIM Base URL**（SCIM 基础 URL）和 **API Token**（API 令牌），并将这些值粘贴到您的 IdP 中。

## 在您的 IdP 中启用 SCIM

您的身份提供商的用户界面可能与以下步骤略有不同。您可以参考您的身份提供商的文档进行验证。有关其他详细信息，请参阅您的身份提供商的文档：

- [Okta](https://help.okta.com/en-us/Content/Topics/Apps/Apps_App_Integration_Wizard_SCIM.htm)
- [Entra ID/Azure AD SAML 2.0](https://learn.microsoft.com/en-us/azure/active-directory/app-provisioning/user-provisioning)

> [!NOTE]
>
> Microsoft 目前不支持在 Entra ID 中的同一个非库应用程序中同时支持 SCIM 和 OIDC。本页面提供了一个经过验证的解决方法，使用单独的非库应用程序进行 SCIM 配置。虽然 Microsoft 没有正式记录此设置，但它在实践中被广泛使用和支持。

{{< tabs >}}
{{< tab name="Okta" >}}

### 步骤一：在 Okta 中启用 SCIM

1. 登录 Okta 并选择 **Admin**（管理员）打开管理门户。
1. 打开您在配置 SSO 连接时创建的应用程序。
1. 在应用程序页面上，选择 **General**（常规）选项卡，然后选择 **Edit App Settings**（编辑应用程序设置）。
1. 启用 SCIM 配置，然后选择 **Save**（保存）。
1. 导航到 **Provisioning**（配置），然后选择 **Edit SCIM Connection**（编辑 SCIM 连接）。
1. 要在 Okta 中配置 SCIM，请使用以下值和设置设置您的连接：
    - SCIM Base URL（SCIM 基础 URL）：SCIM 连接器基础 URL（从 Docker Home 复制）
    - 用户的唯一标识符字段：`email`
    - 支持的配置操作：**Push New Users**（推送新用户）和 **Push Profile Updates**（推送个人资料更新）
    - 身份验证模式：HTTP Header
    - SCIM Bearer Token：HTTP Header Authorization Bearer Token（从 Docker Home 复制）
1. 选择 **Test Connector Configuration**（测试连接器配置）。
1. 查看测试结果并选择 **Save**（保存）。

### 步骤二：启用同步

1. 在 Okta 中，选择 **Provisioning**（配置）。
1. 选择 **To App**（到应用程序），然后选择 **Edit**（编辑）。
1. 启用 **Create Users**（创建用户）、**Update User Attributes**（更新用户属性）和 **Deactivate Users**（停用用户）。
1. 选择 **Save**（保存）。
1. 移除不必要的映射。必要的映射是：
    - 用户名
    - 名字
    - 姓氏
    - 电子邮件

接下来，[设置角色映射](#设置角色映射)。

{{< /tab >}}
{{< tab name="Entra ID (OIDC)" >}}

Microsoft 不支持在同一个非库应用程序中同时支持 SCIM 和 OIDC。
您必须在 Entra ID 中创建第二个非库应用程序用于 SCIM 配置。

### 步骤一：创建单独的 SCIM 应用程序

1. 在 Azure 门户中，转到 **Microsoft Entra ID** > **Enterprise Applications**（企业应用程序）>
**New application**（新应用程序）。
1. 选择 **Create your own application**（创建您自己的应用程序）。
1. 为您的应用程序命名并选择 **Integrate any other application you don't find in the gallery**（集成您在库中找不到的任何其他应用程序）。
1. 选择 **Create**（创建）。

### 步骤二：配置 SCIM 配置

1. 在您的新 SCIM 应用程序中，转到 **Provisioning**（配置）> **Get started**（入门）。
1. 将 **Provisioning Mode**（配置模式）设置为 **Automatic**（自动）。
1. 在 **Admin Credentials**（管理员凭据）下：
    - **Tenant URL**（租户 URL）：粘贴来自 Docker Home 的 **SCIM Base URL**（SCIM 基础 URL）。
    - **Secret Token**（密钥令牌）：粘贴来自 Docker Home 的 **SCIM API token**（SCIM API 令牌）。
1. 选择 **Test Connection**（测试连接）进行验证。
1. 选择 **Save**（保存）存储凭据。

接下来，[设置角色映射](#设置角色映射)。

{{< /tab >}}
{{< tab name="Entra ID (SAML 2.0)" >}}

1. 在 Azure 门户中，转到 **Microsoft Entra ID** > **Enterprise Applications**（企业应用程序），
然后选择您的 Docker SAML 应用程序。
1. 选择 **Provisioning**（配置）> **Get started**（入门）。
1. 将 **Provisioning Mode**（配置模式）设置为 **Automatic**（自动）。
1. 在 **Admin Credentials**（管理员凭据）下：
    - **Tenant URL**（租户 URL）：粘贴来自 Docker Home 的 **SCIM Base URL**（SCIM 基础 URL）。
    - **Secret Token**（密钥令牌）：粘贴来自 Docker Home 的 **SCIM API token**（SCIM API 令牌）。
1. 选择 **Test Connection**（测试连接）进行验证。
1. 选择 **Save**（保存）存储凭据。

接下来，[设置角色映射](#设置角色映射)。

{{< /tab >}}
{{< /tabs >}}

## 设置角色映射

您可以通过在 IdP 中添加可选的 SCIM 属性为用户分配 [Docker 角色](../roles-and-permissions.md)。这些属性会覆盖在您的 SSO 配置中设置的默认角色和团队值。

> [!NOTE]
>
> 角色映射同时支持 SCIM 和即时（JIT）配置。对于 JIT，角色映射仅在首次配置用户时应用。

下表列出了支持的可选用户级属性：

| 属性 | 可能的值    | 备注          |
| --------- | ------------------ | -------------- |
| `dockerRole` | `member`（成员）、`editor`（编辑者）或 `owner`（所有者） | 如果未设置，用户默认为 `member`（成员）角色。设置此属性会覆盖默认值。<br><br>有关角色定义，请参阅[角色和权限](../roles-and-permissions.md)。 |
| `dockerOrg` | Docker `organizationName`（组织名称）（例如，`moby`） | 覆盖在您的 SSO 连接中配置的默认组织。<br><br>如果未设置，用户将被配置到默认组织。如果同时设置了 `dockerOrg` 和 `dockerTeam`，用户将被配置到指定组织内的团队。 |
| `dockerTeam` | Docker `teamName`（团队名称）（例如，`developers`） | 将用户配置到默认或指定组织中的指定团队。如果团队不存在，将自动创建。<br><br>您仍然可以使用[组映射](group-mapping.md)将用户分配到跨组织的多个团队。 |

这些属性使用的外部命名空间是：`urn:ietf:params:scim:schemas:extension:docker:2.0:User`。
在为 Docker 创建自定义 SCIM 属性时，您的身份提供商中需要此值。

{{< tabs >}}
{{< tab name="Okta" >}}

### 步骤一：在 Okta 中设置角色映射

1. 首先设置 [SSO](../single-sign-on/configure/_index.md) 和 SCIM。
1. 在 Okta 管理门户中，转到 **Directory**（目录），选择 **Profile Editor**（个人资料编辑器），然后选择 **User (Default)**（用户（默认））。
1. 选择 **Add Attribute**（添加属性）并配置要添加的角色、组织或团队的值。确切的命名不是必需的。
1. 返回 **Profile Editor**（个人资料编辑器）并选择您的应用程序。
1. 选择 **Add Attribute**（添加属性）并输入所需的值。**External Name**（外部名称）和 **External Namespace**（外部命名空间）必须完全匹配。
    - 组织/团队/角色映射的外部名称值分别是 `dockerOrg`、`dockerTeam` 和 `dockerRole`，如上表所列。
    - 所有这些的外部命名空间都是相同的：`urn:ietf:params:scim:schemas:extension:docker:2.0:User`。
1. 创建属性后，导航到页面顶部并选择 **Mappings**（映射），然后选择 **Okta User to YOUR APP**（Okta 用户到您的应用程序）。
1. 转到新创建的属性并将变量名称映射到外部名称，然后选择 **Save Mappings**（保存映射）。如果您使用 JIT 配置，请继续以下步骤。
1. 导航到 **Applications**（应用程序）并选择 **YOUR APP**（您的应用程序）。
1. 选择 **General**（常规），然后选择 **SAML Settings**（SAML 设置）和 **Edit**（编辑）。
1. 选择 **Step 2**（步骤 2）并配置从用户属性到 Docker 变量的映射。

### 步骤二：按用户分配角色

1. 在 Okta 管理门户中，选择 **Directory**（目录），然后选择 **People**（人员）。
1. 选择 **Profile**（个人资料），然后选择 **Edit**（编辑）。
1. 选择 **Attributes**（属性）并将属性更新为所需值。

### 步骤三：按组分配角色

1. 在 Okta 管理门户中，选择 **Directory**（目录），然后选择 **People**（人员）。
1. 选择 **YOUR GROUP**（您的组），然后选择 **Applications**（应用程序）。
1. 打开 **YOUR APPLICATION**（您的应用程序）并选择 **Edit**（编辑）图标。
1. 将属性更新为所需值。

如果用户尚未设置属性，添加到组的用户将在配置时继承这些属性。

{{< /tab >}}
{{< tab name="Entra ID/Azure AD (SAML 2.0 and OIDC)" >}}

### 步骤一：配置属性映射

1. 完成 [SCIM 配置设置](#在-docker-中启用-scim)。
1. 在 Azure 门户中，打开 **Microsoft Entra ID** > **Enterprise Applications**（企业应用程序），
然后选择您的 SCIM 应用程序。
1. 转到 **Provisioning**（配置）> **Mappings**（映射）> **Provision Azure Active Directory Users**（配置 Azure Active Directory 用户）。
1. 添加或更新以下映射：
    - `userPrincipalName` -> `userName`
    - `mail` -> `emails.value`
    - 可选。使用以下[映射方法之一](#步骤二选择角色映射方法)映射 `dockerRole`、`dockerOrg` 或 `dockerTeam`。
1. 移除任何不支持的属性以防止同步错误。
1. 可选。转到 **Mappings**（映射）> **Provision Azure Active Directory Groups**（配置 Azure Active Directory 组）：
    - 如果组配置导致错误，将 **Enabled**（启用）设置为 **No**（否）。
    - 如果启用，请仔细测试组映射。
1. 选择 **Save**（保存）应用映射。

### 步骤二：选择角色映射方法

您可以使用以下方法之一映射 `dockerRole`、`dockerOrg` 或 `dockerTeam`：

#### 表达式映射

如果只需要分配 Docker 角色（如 `member`、`editor` 或 `owner`），请使用此方法。

1. 在 **Edit Attribute**（编辑属性）视图中，将映射类型设置为 **Expression**（表达式）。
1. 在 **Expression**（表达式）字段中：
    1. 如果您的应用程序角色与 Docker 角色完全匹配，使用：`SingleAppRoleAssignment([appRoleAssignments])`
    1. 如果不匹配，使用 switch 表达式：`Switch(SingleAppRoleAssignment([appRoleAssignments]), "My Corp Admins", "owner", "My Corp Editors", "editor", "My Corp Users", "member")`
1. 设置：
    - **Target attribute**（目标属性）：`urn:ietf:params:scim:schemas:extension:docker:2.0:User:dockerRole`
    - **Match objects using this attribute**（使用此属性匹配对象）：否
    - **Apply this mapping**（应用此映射）：始终
1. 保存您的更改。

> [!WARNING]
>
> 您不能使用 `dockerOrg` 或 `dockerTeam` 与此方法。表达式映射仅与一个属性兼容。

#### 直接映射

如果需要映射多个属性（`dockerRole` + `dockerTeam`），请使用此方法。

1. 对于每个 Docker 属性，选择唯一的 Entra 扩展属性（`extensionAttribute1`、`extensionAttribute2` 等）。
1. 在 **Edit Attribute**（编辑属性）视图中：
    - 将映射类型设置为 **Direct**（直接）。
    - 将 **Source attribute**（源属性）设置为您选择的扩展属性。
    - 将 **Target attribute**（目标属性）设置为以下之一：
        - `dockerRole: urn:ietf:params:scim:schemas:extension:docker:2.0:User:dockerRole`
        - `dockerOrg: urn:ietf:params:scim:schemas:extension:docker:2.0:User:dockerOrg`
        - `dockerTeam: urn:ietf:params:scim:schemas:extension:docker:2.0:User:dockerTeam`
    - 将 **Apply this mapping**（应用此映射）设置为 **Always**（始终）。
1. 保存您的更改。

要分配值，您需要使用 Microsoft Graph API。

### 步骤三：分配用户和组

对于任一映射方法：

1. 在 SCIM 应用程序中，转到 **Users and Groups**（用户和组）> **Add user/group**（添加用户/组）。
1. 选择要配置到 Docker 的用户或组。
1. 选择 **Assign**（分配）。

如果您使用表达式映射：

1. 转到 **App registrations**（应用程序注册）> 您的 SCIM 应用程序 > **App Roles**（应用程序角色）。
1. 创建与 Docker 角色匹配的应用程序角色。
1. 在 **Users and Groups**（用户和组）下将用户或组分配给应用程序角色。

如果您使用直接映射：

1. 转到 [Microsoft Graph Explorer](https://developer.microsoft.com/en-us/graph/graph-explorer)
并以租户管理员身份登录。
1. 使用 Microsoft Graph API 分配属性值。示例 PATCH 请求：

```bash
PATCH https://graph.microsoft.com/v1.0/users/{user-id}
Content-Type: application/json

{
  "extensionAttribute1": "owner",
  "extensionAttribute2": "moby",
  "extensionAttribute3": "developers"
}
```

> [!NOTE]
>
> 您必须为每个 SCIM 字段使用不同的扩展属性。

{{< /tab >}}
{{< /tabs >}}

有关其他详细信息，请参阅您的 IdP 的文档：

- [Okta](https://help.okta.com/en-us/Content/Topics/users-groups-profiles/usgp-add-custom-user-attributes.htm)
- [Entra ID/Azure AD](https://learn.microsoft.com/en-us/azure/active-directory/app-provisioning/customize-application-attributes#provisioning-a-custom-extension-attribute-to-a-scim-compliant-application)

## 测试 SCIM 配置

完成角色映射后，您可以手动测试配置。

{{< tabs >}}
{{< tab name="Okta" >}}

1. 在 Okta 管理门户中，转到 **Directory > People**（目录 > 人员）。
1. 选择您已分配给 SCIM 应用程序的用户。
1. 选择 **Provision User**（配置用户）。
1. 等待几秒钟，然后在 Docker
[Admin Console](https://app.docker.com/admin)（管理控制台）的 **Members**（成员）下检查。
1. 如果用户未出现，请在 **Reports > System Log**（报告 > 系统日志）中查看日志，
并确认应用程序中的 SCIM 设置。

{{< /tab >}}
{{< tab name="Entra ID/Azure AD (OIDC and SAML 2.0)" >}}

1. 在 Azure 门户中，转到 **Microsoft Entra ID** > **Enterprise Applications**（企业应用程序），
然后选择您的 SCIM 应用程序。
1. 转到 **Provisioning**（配置）> **Provision on demand**（按需配置）。
1. 选择用户或组并选择 **Provision**（配置）。
1. 确认用户出现在 Docker
[Admin Console](https://app.docker.com/admin)（管理控制台）的 **Members**（成员）下。
1. 如果需要，请检查 **Provisioning logs**（配置日志）中的错误。

{{< /tab >}}
{{< /tabs >}}

## 禁用 SCIM

如果禁用 SCIM，通过 SCIM 配置的任何用户将保留在组织中。您的用户的未来更改不会从您的 IdP 同步。用户取消配置仅在手动从组织中移除用户时才可能。

要禁用 SCIM：

1. 登录 [Docker Home](https://app.docker.com)。
1. 选择 **Admin Console**（管理控制台），然后选择 **SSO and SCIM**（SSO 和 SCIM）。
1. 在 **SSO connections**（SSO 连接）表中，选择 **Actions**（操作）图标。
1. 选择 **Disable SCIM**（禁用 SCIM）。


## 下一步

- 设置[组映射](/manuals/enterprise/security/provisioning/group-mapping.md)。
- [配置故障排除](/manuals/enterprise/troubleshoot/troubleshoot-provisioning.md)。
