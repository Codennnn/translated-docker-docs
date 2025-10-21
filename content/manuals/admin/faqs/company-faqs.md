---
title: 公司常见问题
linkTitle: 公司
weight: 30
description: 公司常见问题
keywords: Docker, Docker Hub, SSO 常见问题, 单点登录, 公司, 管理, 公司管理
tags: [FAQ]
aliases:
- /docker-hub/company-faqs/
- /faq/admin/company-faqs/
---

### 我的一些组织没有 Docker Business 订阅。我仍可以使用父公司吗？

可以，但您只能将拥有 Docker Business 订阅的组织添加到公司中。

### 如果我的某个组织从 Docker Business 降级，但我仍需要作为公司所有者访问，该怎么办？

要访问和管理子组织，该组织必须拥有 Docker Business 订阅。如果该组织未包含在此订阅中，则该组织的所有者必须在公司外部管理该组织。

### 公司所有者会占用订阅席位吗？

公司所有者不会占用席位，除非满足以下条件之一：

- 他们被添加为公司下组织的成员
- 启用了 SSO

尽管公司所有者在公司内的所有组织中拥有与组织所有者相同的访问权限，但无需将他们添加到任何组织。这样做会导致他们占用席位。

当您首次创建公司时，您的账户既是公司所有者又是组织所有者。在这种情况下，只要您仍然是组织所有者，您的账户就会占用一个席位。

为避免占用席位，请[指定另一用户为组织所有者](/manuals/admin/organization/members.md#update-a-member-role)并将自己从组织中移除。
作为公司所有者，您将保留完整的管理访问权限，而不占用订阅席位。

### 公司所有者在关联/嵌套组织中拥有什么权限？

公司所有者可以导航到**组织**页面，在一个位置查看所有嵌套组织。他们还可以查看或编辑组织成员，并更改单点登录 (SSO) 和跨域身份管理系统 (SCIM) 设置。公司设置的更改会影响公司下每个组织中的所有用户。

更多信息，请参阅[角色和权限](/manuals/enterprise/security/roles-and-permissions.md)。
