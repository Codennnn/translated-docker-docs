---
title: 配置单点登录
linkTitle: 配置
description: 了解如何为您的组织或公司配置单点登录。
keywords: 配置, sso, docker hub, hub, docker 管理员, 管理员, 安全
aliases:
  - /docker-hub/domains/
  - /docker-hub/sso-connection/
  - /docker-hub/enforcing-sso/
  - /single-sign-on/configure/
  - /admin/company/settings/sso-configuration/
  - /admin/organization/security-settings/sso-configuration/
  - /security/for-admins/single-sign-on/configure/
---

{{< summary-bar feature_name="SSO" >}}

了解如何通过添加和验证成员用于登录的域，为您的 Docker 组织设置单点登录(SSO)。

## 第一步：添加域

> [!NOTE]
>
> Docker 支持多个身份提供商(IdP)配置。您可以将一个域与多个 IdP 关联。

添加域的步骤：

1. 登录 [Docker Home](https://app.docker.com) 并选择您的组织。如果组织属于某个公司，请先选择该公司以在该级别管理域。
1. 选择**管理控制台**，然后选择**域管理**。
1. 选择**添加域**。
1. 在文本框中输入您的域，然后选择**添加域**。
1. 在弹出的模态框中，复制为域验证提供的**TXT 记录值**。

## 第二步：验证您的域

要确认域所有权，请使用 Docker 提供的 TXT 记录值向您的域名系统(DNS)主机添加 TXT 记录。DNS 传播可能需要长达 72 小时。在此期间，Docker 会自动检查该记录。

> [!TIP]
>
> 添加记录名称时，对于像 `example.com` 这样的根域，**使用 `@` 或留空**。**避免使用常见值**，如 `docker`、`docker-verification`、`www` 或您的域名本身。始终**检查您的 DNS 提供商文档**以验证其特定的记录名称要求。

{{< tabs >}}
{{< tab name="AWS Route 53" >}}

1. 要将 TXT 记录添加到 AWS，请参阅[使用 Amazon Route 53 控制台创建记录](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/resource-record-sets-creating.html)。
1. 等待最多 72 小时进行 TXT 记录验证。
1. 记录生效后，转到[管理控制台](https://app.docker.com/admin)中的**域管理**并选择**验证**。

{{< /tab >}}
{{< tab name="Google Cloud DNS" >}}

1. 要将 TXT 记录添加到 Google Cloud DNS，请参阅[使用 TXT 记录验证您的域](https://cloud.google.com/identity/docs/verify-domain-txt)。
1. 等待最多 72 小时进行 TXT 记录验证。
1. 记录生效后，转到[管理控制台](https://app.docker.com/admin)中的**域管理**并选择**验证**。

{{< /tab >}}
{{< tab name="GoDaddy" >}}

1. 要将 TXT 记录添加到 GoDaddy，请参阅[添加 TXT 记录](https://www.godaddy.com/help/add-a-txt-record-19232)。
1. 等待最多 72 小时进行 TXT 记录验证。
1. 记录生效后，转到[管理控制台](https://app.docker.com/admin)中的**域管理**并选择**验证**。

{{< /tab >}}
{{< tab name="其他提供商" >}}

1. 登录您的域主机。
1. 向您的 DNS 设置添加 TXT 记录并保存记录。
1. 等待最多 72 小时进行 TXT 记录验证。
1. 记录生效后，转到[管理控制台](https://app.docker.com/admin)中的**域管理**并选择**验证**。

{{< /tab >}}
{{< /tabs >}}

## 进一步阅读

- [连接 Docker 和您的 IdP](connect.md)。
- [排查](/manuals/enterprise/troubleshoot/troubleshoot-sso.md) SSO 问题。
