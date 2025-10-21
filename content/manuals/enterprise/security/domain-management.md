---
title: 域管理
description: 添加、验证和管理域，以控制用户访问权限并在 Docker 组织中启用自动配置
keywords: 域管理, 域验证, 自动配置, 用户管理, DNS, TXT 记录, 管理控制台
weight: 55
aliases:
 - /security/for-admins/domain-management/
---

{{< summary-bar feature_name="Domain management" >}}

域管理功能允许您为组织添加并验证域，然后启用自动配置，当用户使用与您已验证域匹配的电子邮件地址登录时，系统会自动将其添加到组织中。这种方法简化了用户管理流程，确保安全设置的一致性，并降低了未经管理的用户在无可见性或控制的情况下访问 Docker 的风险。

本页面提供了添加和删除域、配置自动配置以及审计未捕获用户的步骤。

## 添加和验证域

添加域需要验证以确认所有权。验证过程使用 DNS 记录来证明您控制该域。

### 添加域

1. 登录 [Docker Home](https://app.docker.com) 并选择您的组织。如果您的组织隶属于某个公司，请选择该公司，并在公司级别为组织配置域。
1. 选择 **Admin Console**（管理控制台），然后选择 **Domain management**（域管理）。
1. 选择 **Add a domain**（添加域）。
1. 输入您的域名并选择 **Add domain**（添加域）。
1. 在弹出的模态窗口中，复制 **TXT Record Value**（TXT 记录值）以验证您的域。

### 验证域

验证通过向域名系统 (DNS) 主机添加 TXT 记录来确认您拥有该域。DNS 更改最多可能需要 72 小时才能传播。Docker 会自动检查该记录，并在更改被识别后确认所有权。

> [!TIP]
>
> 记录名称字段决定了 TXT 记录在您的域中的添加位置（根域或子域）。对于像 `example.com` 这样的根域，根据您的提供商，使用 `@` 或将记录名称留空。不要输入 docker、`docker-verification`、`www 或您的域名等值，因为这些可能会指向错误的位置。请查看您的 DNS 提供商文档以确认记录名称要求。

按照您的 DNS 提供商的步骤添加 **TXT Record Value**（TXT 记录值）。如果您的提供商未列出，请使用"其他提供商"的步骤：

{{< tabs >}}
{{< tab name="AWS Route 53" >}}

1. 按照 [使用 Amazon Route 53 控制台创建记录](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/resource-record-sets-creating.html) 的说明，将您的 TXT 记录添加到 AWS。
1. 等待最多 72 小时以进行 TXT 记录验证。
1. 返回 [Admin Console](https://app.docker.com/admin) 的 **Domain management**（域管理）页面，在您的域名旁边选择 **Verify**（验证）。

{{< /tab >}}
{{< tab name="Google Cloud DNS" >}}

1. 按照 [使用 TXT 记录验证您的域](https://cloud.google.com/identity/docs/verify-domain-txt) 的说明，将您的 TXT 记录添加到 Google Cloud DNS。
1. 等待最多 72 小时以进行 TXT 记录验证。
1. 返回 [Admin Console](https://app.docker.com/admin) 的 **Domain management**（域管理）页面，在您的域名旁边选择 **Verify**（验证）。

{{< /tab >}}
{{< tab name="GoDaddy" >}}

1. 按照 [添加 TXT 记录](https://www.godaddy.com/help/add-a-txt-record-19232) 的说明，将您的 TXT 记录添加到 GoDaddy。
1. 等待最多 72 小时以进行 TXT 记录验证。
1. 返回 [Admin Console](https://app.docker.com/admin) 的 **Domain management**（域管理）页面，在您的域名旁边选择 **Verify**（验证）。

{{< /tab >}}
{{< tab name="Other providers" >}}

1. 登录您的域主机。
1. 使用来自 Docker 的 **TXT Record Value**（TXT 记录值）向您的 DNS 设置添加 TXT 记录。
1. 等待最多 72 小时以进行 TXT 记录验证。
1. 返回 [Admin Console](https://app.docker.com/admin) 的 **Domain management**（域管理）页面，在您的域名旁边选择 **Verify**（验证）。

{{< /tab >}}
{{< /tabs >}}

## 配置自动配置

自动配置功能会在用户使用与您已验证域匹配的电子邮件地址登录时，自动将其添加到您的组织中。必须先验证域，然后才能启用自动配置。

> [!IMPORTANT]
>
> 对于属于 SSO 连接一部分的域，在将用户添加到组织时，即时 (JIT) 配置优先于自动配置。

### 自动配置的工作原理

当为已验证的域启用自动配置时：

- 使用匹配的电子邮件地址登录 Docker 的用户会自动添加到您的组织中。
- 自动配置仅将现有的 Docker 用户添加到您的组织中，不会创建新账户。
- 用户的登录过程不会有任何变化。
- 当添加新用户时，公司和组织所有者会收到电子邮件通知。
- 您可能需要[管理席位](/manuals/subscription/manage-seats.md)以容纳新用户。

### 启用自动配置

自动配置是按域配置的。要启用它：

1. 登录 [Docker Home](https://app.docker.com) 并选择您的公司或组织。
1. 选择 **Admin Console**（管理控制台），然后选择 **Domain management**（域管理）。
1. 选择要为其启用自动配置的域旁边的 **Actions menu**（操作菜单）。
1. 选择 **Enable auto-provisioning**（启用自动配置）。
1. 可选。如果在公司级别启用自动配置，请选择一个组织。
1. 选择 **Enable**（启用）进行确认。

该域的 **Auto-provisioning**（自动配置）列将更新为 **Enabled**（已启用）。

### 禁用自动配置

要为用户禁用自动配置：

1. 登录 [Docker Home](https://app.docker.com) 并选择您的组织。如果您的组织隶属于某个公司，请选择该公司，并在公司级别为组织配置域。
1. 选择 **Admin Console**（管理控制台），然后选择 **Domain management**（域管理）。
1. 选择您的域旁边的 **Actions menu**（操作菜单）。
1. 选择 **Disable auto-provisioning**（禁用自动配置）。
1. 选择 **Disable**（禁用）进行确认。

## 审计未捕获用户

{{< summary-bar feature_name="Domain audit" >}}

域审计功能用于识别未捕获用户。未捕获用户是指使用与您已验证域关联的电子邮件地址进行身份验证，但不是您的 Docker 组织成员的 Docker 用户。

### 限制

域审计无法识别：

- 未经身份验证访问 Docker Desktop 的用户
- 使用没有与您已验证域关联的电子邮件地址的账户进行身份验证的用户

为防止无法识别的用户访问 Docker Desktop，请[强制登录](/manuals/enterprise/security/enforce-sign-in/_index.md)。

### 运行域审计

1. 登录 [Docker Home](https://app.docker.com) 并选择您的公司。
1. 选择 **Admin Console**（管理控制台），然后选择 **Domain management**（域管理）。
1. 在 **Domain audit**（域审计）中，选择 **Export Users**（导出用户）以导出未捕获用户的 CSV 文件。

CSV 文件包含以下列：
- Name：Docker 用户的显示名称
- Username：用户的 Docker ID
- Email：用户的电子邮件地址

### 邀请未捕获用户

您可以使用导出的 CSV 文件批量邀请未捕获用户加入您的组织。有关批量邀请用户的更多信息，请参阅[管理组织成员](/manuals/admin/organization/members.md)。

## 删除域

删除域会移除其 TXT 记录值并禁用任何关联的自动配置。

>[!WARNING]
>
> 删除域将禁用该域的自动配置并移除验证。此操作无法撤销。

要删除域：

1. 登录 [Docker Home](https://app.docker.com) 并选择您的组织。如果您的组织隶属于某个公司，请选择该公司，并在公司级别为组织配置域。
1. 选择 **Admin Console**（管理控制台），然后选择 **Domain management**（域管理）。
1. 对于要删除的域，选择 **Actions**（操作）菜单，然后选择 **Delete domain**（删除域）。
1. 在弹出的模态窗口中，选择 **Delete domain**（删除域）进行确认。
