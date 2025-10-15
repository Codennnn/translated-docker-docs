---
description: 了解如何在 Docker Hub 上管理存储库访问控制。
keywords: Docker Hub, Hub, 存储库访问, 协作者, 存储库可见性, 隐私
title: 访问管理
LinkTItle: 访问
weight: 50
aliases:
- /docker-hub/repos/access/
---

本主题介绍如何管理存储库的访问控制能力，包括可见性（public/private）、协作者、组织角色、团队，以及组织访问令牌（Organization Access Tokens，OAT）。

## 存储库可见性

最基础的访问控制由“可见性”决定。存储库可以设置为公开（public）或私有（private）。

当存储库为公开时，它会出现在 Docker Hub 搜索结果中，并且任何人都可以拉取（pull）。
要管理“公开的个人存储库”的推送（push）权限，可以使用协作者；
要管理“公开的组织存储库”的推送权限，可以使用组织角色、团队或组织访问令牌。

当存储库为私有时，它不会出现在 Docker Hub 搜索结果中，仅授予了权限的用户可访问。
要管理“私有的个人存储库”的推送与拉取权限，可以使用协作者；
要管理“私有的组织存储库”的推送与拉取权限，可以使用组织角色、团队或组织访问令牌。

### 更改存储库可见性

在 Docker Hub 创建存储库时，你可以设置其可见性。
此外，你可以在个人设置中配置“新建存储库的默认可见性”。
以下说明如何在存储库创建完成后更改可见性。

更改存储库可见性：

1. 登录 [Docker Hub](https://hub.docker.com)。
2. 选择 **My Hub** > **Repositories**。
3. 选择一个存储库。

   将进入该存储库的 **General** 页面。

4. 切换到 **Settings** 选项卡。
5. 在 **Visibility settings** 下，选择以下其一：

   - **Make public**：存储库将出现在 Docker Hub 搜索结果中，且任何人都可拉取。
   - **Make private**：存储库不会出现在 Docker Hub 搜索结果中，仅你与协作者可访问；
     若该存储库位于某组织命名空间中，则具有相应角色或权限的组织成员也可访问。

6. 输入存储库名称以确认更改。
7. 点击 **Make public** 或 **Make private**。

## 协作者（Collaborators）

协作者是指你希望授予某“个人存储库”推送（`push`）和拉取（`pull`）权限的用户。
协作者无法执行管理类操作，例如删除存储库，或将可见性从私有改为公开；
同时，协作者也不能为该存储库添加其他协作者。

仅个人存储库支持协作者。
对于公开的个人存储库，你可以添加不限数量的协作者；
对于私有的个人存储库，Docker Pro 账户最多可添加 1 名协作者。

组织存储库不支持协作者；你可以通过组织成员角色、团队或组织访问令牌来管理访问权限。

### 管理协作者

1. 登录 [Docker Hub](https://hub.docker.com)。

2. 选择 **My Hub** > **Repositories**。

   随即显示你的存储库列表。

3. 选择一个存储库。

   将进入该存储库的 **General** 页面。

4. 切换到 **Collaborators** 选项卡。

5. 根据对方的 Docker 用户名添加或移除协作者。

你也可以在该存储库的 **Settings** 页面选择协作者并管理其对私有存储库的访问权限。

## 组织角色（Organization roles）

组织可以为个人分配角色，以授予其在组织内的不同权限。
详见[角色与权限](/manuals/enterprise/security/roles-and-permissions.md)。

## 组织团队（Organization teams）

组织可以使用团队来管理权限，并为团队分配更细粒度的存储库访问级别。

### 配置团队的存储库权限

在配置存储库权限之前，你需要先创建团队。
详见[创建与管理团队](/manuals/admin/organization/manage-a-team.md)。

配置团队的存储库权限：

1. 登录 [Docker Hub](https://hub.docker.com)。

2. 选择 **My Hub** > **Repositories**。

   随即显示你的存储库列表。

3. 选择一个存储库。

   将进入该存储库的 **General** 页面。

4. 切换到 **Permissions** 选项卡。

5. 添加、修改或移除某团队的存储库权限。

   - 添加：指定 **Team**，选择 **Permission**，然后点击 **Add**。
   - 修改：在对应团队旁边指定新的权限。
   - 移除：点击团队旁的 **Remove permission** 图标。

## 组织访问令牌（Organization access tokens，OATs）

组织可以使用 OAT 进行权限管理。通过 OAT，你可以将细粒度的存储库访问权限授予令牌。
详见[组织访问令牌](/manuals/enterprise/security/access-tokens.md)。

## Gated distribution

{{< summary-bar feature_name="Gated distribution" >}}

Gated distribution 允许发布者在不授予完整组织访问权限、也不暴露团队、协作者或其他存储库信息的前提下，
将私有容器镜像安全地分享给外部客户或合作伙伴。

该功能非常适合商业软件发布者：既能精细控制谁可以拉取特定镜像，又能保持内部用户与外部使用者之间的清晰隔离。

如果你对 Gated distribution 感兴趣，请联系[Docker 销售团队](https://www.docker.com/pricing/contact-sales/)了解更多信息。

### 关键特性

-. **私有存储库分发**：内容存放于私有存储库，仅对被明确邀请的用户开放。

-. **无需加入组织**：外部用户无需加入内部组织即可拉取镜像。

-. **仅拉取权限**：外部用户仅具备拉取权限，不能推送或修改存储库内容。

-. **邀请制访问**：通过 API 管理基于认证邮件的邀请来授予访问权限。

### 通过 API 邀请分发成员（distributor members）

> [!NOTE]
> 在发送邀请时，你需要为成员分配角色。
> 各角色的访问权限说明见[角色与权限](/manuals/enterprise/security/roles-and-permissions.md)。

分发成员（用于 Gated distribution）目前只能通过 Docker Hub API 邀请，暂不支持在 UI 中邀请。
要邀请分发成员，请使用“批量创建邀请”的 API 端点。

邀请分发成员：

1. 使用 [Authentication API](https://docs.docker.com/reference/api/hub/latest/#tag/authentication-api/operation/AuthCreateAccessToken) 为你的 Docker Hub 账号生成 Bearer Token。

2. 在 Hub UI 中创建团队，或使用 [Teams API](https://docs.docker.com/reference/api/hub/latest/#tag/groups/paths/~1v2~1orgs~1%7Borg_name%7D~1groups/post)。

3. 为该团队授予存储库访问权限：
   - 通过 Hub UI：进入存储库设置，为团队添加“Read-only”权限；
   - 使用 [Repository Teams API](https://docs.docker.com/reference/api/hub/latest/#tag/repositories/paths/~1v2~1repositories~1%7Bnamespace%7D~1%7Brepository%7D~1groups/post)：将团队分配到指定存储库，并设为 "read-only" 访问级别。

4. 使用 [Bulk create invites 端点](https://docs.docker.com/reference/api/hub/latest/#tag/invites/paths/~1v2~1invites~1bulk/post) 发送带有“distributor_member”角色的邮件邀请；
   在请求体中将 "role" 字段设置为 "distributor_member"。

5. 受邀用户会收到带有接受链接的邮件。其使用 Docker ID 登录并接受后，将作为分发成员获得对指定私有存储库的仅拉取（pull-only）访问权限。
