---
title: 创建和管理团队
weight: 40
description: 了解如何为您的组织创建和管理团队
keywords: docker, registry, teams, organizations, plans, Dockerfile, Docker
  Hub, docs, documentation, repository permissions, configure repository access, team management
aliases:
- /docker-hub/manage-a-team/
---

{{< summary-bar feature_name="Admin orgs" >}}

您可以在管理控制台或 Docker Hub 中为您的组织创建团队，并在 Docker Hub 中配置团队的仓库访问权限。

团队是隶属于一个组织的 Docker 用户组。一个组织可以拥有多个团队。组织所有者可以使用 Docker ID 或电子邮件地址创建新团队并向现有团队添加成员。成员不一定需要加入团队才能与组织关联。

组织所有者可以指定其他组织所有者，通过授予所有者角色来帮助他们管理组织中的用户、团队和仓库。

## 什么是组织所有者？

组织所有者是具有以下权限的管理员：

- 管理仓库并向组织添加团队成员
- 访问私有仓库、所有团队、账单信息和组织设置
- 为组织中的每个团队指定[权限](#permissions-reference)
- 为组织启用[SSO](/manuals/enterprise/security/single-sign-on/_index.md)

当为您的组织启用 SSO 后，组织所有者还可以管理用户。Docker 可以通过 SSO 强制为新终端用户或希望拥有独立公司使用 Docker ID 的用户自动配置 Docker ID。

组织所有者可以添加其他具有所有者角色的用户，帮助他们管理组织中的用户、团队和仓库。

有关角色的更多信息，请参阅[角色和权限](/manuals/enterprise/security/roles-and-permissions.md)。

## 创建团队

1. 登录 [Docker Home](https://app.docker.com) 并选择您的组织。
1. 选择 **Teams**（团队）。

## 设置团队仓库权限

您必须先创建团队，然后才能配置仓库权限。有关更多详细信息，请参阅[创建和管理团队](/manuals/admin/organization/manage-a-team.md)。

要设置团队仓库权限：

1. 登录 [Docker Hub](https://hub.docker.com)。
1. 选择 **My Hub** > **Repositories**（仓库）。

   您的仓库列表将会显示。

1. 选择一个仓库。

   仓库的 **General**（常规）页面将会显示。

1. 选择 **Permissions**（权限）选项卡。
1. 添加、修改或删除团队的仓库权限。

   - 添加：指定**团队**，选择**权限**，然后选择**添加**。
   - 修改：在团队旁边指定新权限。
   - 删除：选择团队旁边的**删除权限**图标。

### 权限参考

- `只读`（Read-only）权限允许用户查看、搜索和拉取私有仓库，就像访问公共仓库一样。
- `读写`（Read & Write）权限允许用户拉取、推送和查看仓库。此外，还允许用户查看、取消、重试或触发构建。
- `管理员`（Admin）权限允许用户拉取、推送、查看、编辑和删除仓库。您还可以编辑构建设置并更新仓库描述、协作者权限、公共/私有可见性以及删除仓库。

权限是累积的。例如，如果您拥有"读写"权限，则自动拥有"只读"权限。

下表显示了每个权限级别允许用户执行的操作：

| 操作 | 只读 | 读写 | 管理员 |
|:------------------:|:---------:|:------------:|:-----:|
| 拉取仓库 | ✅ | ✅ | ✅ |
| 查看仓库 | ✅ | ✅ | ✅ |
| 推送仓库 | ❌ | ✅ | ✅ |
| 编辑仓库 | ❌ | ❌ | ✅ |
| 删除仓库 | ❌ | ❌ | ✅ |
| 更新仓库描述 | ❌ | ❌ | ✅ |
| 查看构建 | ✅ | ✅ | ✅ |
| 取消构建 | ❌ | ✅ | ✅ |
| 重试构建 | ❌ | ✅ | ✅ |
| 触发构建 | ❌ | ✅ | ✅ |
| 编辑构建设置 | ❌ | ❌ | ✅ |

> [!NOTE]
>
> 未验证电子邮件地址的用户仅对仓库具有`只读`访问权限，无论其团队成员身份授予了何种权限。

## 查看所有仓库的团队权限

要查看团队在所有仓库中的权限：

1. 登录 [Docker Hub](https://hub.docker.com)。
1. 选择 **My Hub** 并选择您的组织。
1. 选择 **Teams**（团队）并选择您的团队名称。
1. 选择 **Permissions**（权限）选项卡，您可以在此查看该团队可以访问的仓库。

## 删除团队

组织所有者可以删除团队。当您从组织中删除团队时，此操作将撤销成员对团队授权资源的访问权限。这不会将用户从他们所属的其他团队中移除，也不会删除任何资源。

1. 登录 [Docker Home](https://app.docker.com/) 并选择您的组织。
1. 选择 **Teams**（团队）。
1. 选择要删除的团队名称旁边的 **Actions**（操作）图标。
1. 选择 **Delete team**（删除团队）。
1. 查看确认消息，然后选择 **Delete**（删除）。

## 更多资源

- [视频：Docker 团队](https://youtu.be/WKlT1O-4Du8?feature=shared&t=348)
- [视频：角色、团队和仓库](https://youtu.be/WKlT1O-4Du8?feature=shared&t=435)
