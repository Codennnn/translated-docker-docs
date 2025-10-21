---
description: 组织常见问题
linkTitle: 组织
weight: 20
keywords: Docker, Docker Hub, SSO 常见问题, 单点登录, 组织, 管理, 管理控制台, 成员, 组织管理, 管理组织
title: 组织常见问题
tags: [FAQ]
aliases:
- /docker-hub/organization-faqs/
- /faq/admin/organization-faqs/
---

### 如何查看我的组织中有多少活跃用户？

如果您的组织使用了软件资产管理工具，可以使用该工具来查看有多少用户安装了 Docker Desktop。如果您的组织没有使用此类软件，可以进行内部调查，了解谁在使用 Docker Desktop。

更多信息，请参阅[识别您的 Docker 用户及其 Docker 账户](../../admin/organization/onboard.md#step-1-identify-your-docker-users-and-their-docker-accounts)。

### 用户是否需要先通过 Docker 身份验证，所有者才能将他们添加到组织？

不需要。组织所有者可以使用电子邮件地址邀请用户，并可以在邀请过程中将他们分配到团队。

### 我可以强制组织成员在使用 Docker Desktop 前进行身份验证吗？这样做有什么好处？

可以。您可以[强制登录](/manuals/enterprise/security/enforce-sign-in/_index.md)。

强制登录的一些好处包括：

- 管理员可以强制执行[镜像访问管理](/manuals/enterprise/security/hardened-desktop/image-access-management.md)和[仓库访问管理](/manuals/enterprise/security/hardened-desktop/registry-access-management.md)等功能。
- 管理员可以通过阻止未以组织成员身份登录的用户使用 Docker Desktop 来确保合规性。

### 我可以将我的个人 Docker ID 转换为组织账户吗？

可以。您可以将用户账户转换为组织账户。一旦将用户账户转换为组织，就无法将其恢复为个人用户账户。

有关前提条件和说明，请参阅[将账户转换为组织](convert-account.md)。

### 组织受邀者会占用席位吗？

会。被邀请加入组织的用户将占用一个已配置的席位，即使该用户尚未接受邀请。

要管理邀请，请参阅[管理组织成员](/manuals/admin/organization/members.md)。

### 组织所有者会占用席位吗？

会。组织所有者会占用席位。

### 用户、受邀者、席位和成员之间有什么区别？

- 用户：拥有 Docker ID 的 Docker 用户。
- 受邀者：管理员已邀请加入组织但尚未接受邀请的用户。
- 席位：组织中购买的席位数量。
- 成员：已收到并接受加入组织邀请的用户。成员也可以指组织内团队的成员。

### 如果我有两个组织，且一个用户同时属于这两个组织，他们会占用两个席位吗？

会。在用户属于两个组织的情况下，他们会在每个组织中各占用一个席位。
