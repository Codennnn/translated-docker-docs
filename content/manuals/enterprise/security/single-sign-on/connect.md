---
title: 连接单点登录
linkTitle: 连接
description: 连接 Docker 和您的身份提供商，测试设置并启用强制执行
keywords: 配置 sso, 设置 sso, docker sso 设置, docker 身份提供商, sso 强制执行, docker hub, 安全
aliases:
 - /security/for-admins/single-sign-on/connect/
---

{{< summary-bar feature_name="SSO" >}}

设置单点登录(SSO)连接涉及配置 Docker 和您的身份提供商(IdP)。本指南将引导您完成在 Docker 中的设置、在 IdP 中的设置以及最终连接。

> [!TIP]
>
> 您需要在 Docker 和 IdP 之间复制和粘贴值。请在一次会话中完成本指南，并为 Docker 和 IdP 打开单独的浏览器窗口。

## 支持的身份提供商

Docker 支持任何 SAML 2.0 或 OIDC 兼容的身份提供商。本指南为最常用的提供商提供详细的设置说明：Okta 和 Microsoft Entra ID。

如果您使用的是不同的 IdP，一般流程保持不变：

1. 在 Docker 中配置连接。
1. 使用来自 Docker 的值在 IdP 中设置应用程序。
1. 通过将 IdP 的值输入回 Docker 来完成连接。
1. 测试连接。

## 先决条件

开始之前：

- 验证您的域
- 设置您的身份提供商(IdP)账户
- 完成[配置单点登录](configure.md)指南中的步骤

## 步骤一：在 Docker 中创建 SSO 连接

> [!NOTE]
>
> 在创建 SSO 连接之前，您必须[验证至少一个域](/manuals/enterprise/security/single-sign-on/configure.md)。

1. 登录 [Docker Home](https://app.docker.com) 并选择您的组织。
1. 选择**管理控制台**，然后选择**SSO 和 SCIM**。
1. 选择**创建连接**并为连接提供名称。
1. 选择身份验证方法：**SAML** 或 **Azure AD (OIDC)**。
1. 复制 IdP 所需的值：
    - Okta SAML：**实体 ID**、**ACS URL**
    - Azure OIDC：**重定向 URL**

保持此窗口打开，以便稍后粘贴来自 IdP 的值。

## 步骤二：在您的 IdP 中创建 SSO 连接

根据您的 IdP 提供商使用以下选项卡。

{{< tabs >}}
{{< tab name="Okta SAML" >}}

1. 登录您的 Okta 账户并打开管理门户。
1. 选择**管理**，然后选择**创建应用集成**。
1. 选择**SAML 2.0**，然后选择**下一步**。
1. 将您的应用命名为"Docker"。
1. 可选。上传徽标。
1. 粘贴来自 Docker 的值：
    - Docker ACS URL -> **单一登录 URL**
    - Docker 实体 ID -> **受众 URI (SP 实体 ID)**
1. 配置以下设置：
    - 名称 ID 格式：`EmailAddress`
    - 应用程序用户名：`Email`
    - 更新应用程序：`创建和更新`
1. 可选。添加 SAML 属性。请参阅 [SSO 属性](/manuals/enterprise/security/provisioning/_index.md#sso-attributes)。
1. 选择**下一步**。
1. 选择**这是我们创建的内部应用**复选框。
1. 选择**完成**。

{{< /tab >}}
{{< tab name="Entra ID SAML 2.0" >}}

1. 登录 Microsoft Entra（前身为 Azure AD）。
1. 选择**默认目录** > **添加** > **企业应用程序**。
1. 选择**创建您自己的应用程序**，将其命名为"Docker"，并选择**非库**。
1. 创建应用程序后，转到**单一登录**并选择**SAML**。
1. 在**基本 SAML 配置**部分选择**编辑**。
1. 编辑**基本 SAML 配置**并粘贴来自 Docker 的值：
    - Docker 实体 ID -> **标识符**
    - Docker ACS URL -> **回复 URL**
1. 可选。添加 SAML 属性。请参阅 [SSO 属性](/manuals/enterprise/security/provisioning/_index.md#sso-attributes)。
1. 保存配置。
1. 从**SAML 签名证书**部分，下载您的**证书(Base64)**。

{{< /tab >}}
{{< tab name="Azure Connect (OIDC)" >}}

### 注册应用程序

1. 登录 Microsoft Entra（前身为 Azure AD）。
1. 选择**应用注册** > **新注册**。
1. 将应用程序命名为"Docker"。
1. 设置账户类型并粘贴来自 Docker 的**重定向 URI**。
1. 选择**注册**。
1. 复制**客户端 ID**。

### 创建客户端密钥

1. 在您的应用程序中，转到**证书和密钥**。
1. 选择**新建客户端密钥**，描述并配置持续时间，然后选择**添加**。
1. 复制新密钥的**值**。

### 设置 API 权限

1. 在您的应用程序中，转到**API 权限**。
1. 选择**授予管理员同意**并确认。
1. 选择**添加权限** > **委托的权限**。
1. 搜索并选择 `User.Read`。
1. 确认已授予管理员同意。

{{< /tab >}}
{{< /tabs >}}

## 步骤三：将 Docker 连接到您的 IdP

通过将 IdP 值粘贴到 Docker 中来完成集成。

{{< tabs >}}
{{< tab name="Okta SAML" >}}

1. 在 Okta 中，选择您的应用程序并转到**查看 SAML 设置说明**。
1. 复制**SAML 登录 URL** 和 **x509 证书**。

    > [!IMPORTANT]
    >
    > 复制整个证书，包括 `----BEGIN CERTIFICATE----` 和 `----END CERTIFICATE----` 行。
1. 返回 Docker 管理控制台。
1. 粘贴**SAML 登录 URL** 和 **x509 证书**值。
1. 可选。选择默认团队。
1. 检查并选择**创建连接**。

{{< /tab >}}
{{< tab name="Entra ID SAML 2.0" >}}

1. 在文本编辑器中打开您下载的**证书(Base64)**。
1. 复制以下值：
    - 来自 Azure AD：**登录 URL**
    - **证书(Base64)** 内容

    > [!IMPORTANT]
    >
    > 复制整个证书，包括 `----BEGIN CERTIFICATE----` 和 `----END CERTIFICATE----` 行。
1. 返回 Docker 管理控制台。
1. 粘贴**登录 URL** 和 **证书(Base64)**值。
1. 可选。选择默认团队。
1. 检查并选择**创建连接**。

{{< /tab >}}
{{< tab name="Azure Connect (OIDC)" >}}

1. 返回 Docker 管理控制台。
1. 粘贴以下值：
    - **客户端 ID**
    - **客户端密钥**
    - **Azure AD 域**
1. 可选。选择默认团队。
1. 检查并选择**创建连接**。

{{< /tab >}}
{{< /tabs >}}

## 步骤四：测试连接

1. 打开无痕浏览器窗口。
1. 使用您的**域电子邮件地址**登录管理控制台。
1. 浏览器将重定向到您的身份提供商登录页面进行身份验证。如果您有[多个 IdP](#optional-configure-multiple-idps)，请选择登录选项**继续使用 SSO**。
1. 通过您的域电子邮件进行身份验证，而不是使用您的 Docker ID。

如果您使用的是 CLI，则必须使用个人访问令牌进行身份验证。

## 可选：配置多个 IdP

Docker 支持多个 IdP 配置。要在一个域中使用多个 IdP：

- 为每个 IdP 重复本页上的步骤 1-4。
- 每个连接必须使用相同的域。
- 用户将在登录时选择**继续使用 SSO** 来选择他们的 IdP。

## 可选：强制执行 SSO

> [!IMPORTANT]
>
> 如果不强制执行 SSO，用户仍然可以使用 Docker 用户名和密码登录。

强制执行 SSO 要求用户在登录 Docker 时使用 SSO。这集中了身份验证并强制执行由 IdP 设置的策略。

1. 登录 [Docker Home](https://app.docker.com/) 并选择您的组织或公司。
1. 选择**管理控制台**，然后选择**SSO 和 SCIM**。
1. 在 SSO 连接表中，选择**操作**菜单，然后选择**启用强制执行**。
1. 按照屏幕上的说明操作。
1. 选择**打开强制执行**。

当强制执行 SSO 时，您的用户将无法修改他们的电子邮件地址和密码、将用户账户转换为组织或通过 Docker Hub 设置 2FA。如果您想使用 2FA，必须通过您的 IdP 启用 2FA。

## 进一步阅读

- [配置用户](/manuals/enterprise/security/provisioning/_index.md)。
- [强制登录](../enforce-sign-in/_index.md)。
- [创建个人访问令牌](/manuals/enterprise/security/access-tokens.md)。
- [排查 SSO](/manuals/enterprise/troubleshoot/troubleshoot-sso.md) 问题。
