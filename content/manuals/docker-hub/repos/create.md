---
description: 了解如何在 Docker Hub 上创建仓库
keywords: Docker Hub, Hub, 仓库, 创建
title: 创建仓库
linkTitle: 创建
toc_max: 3
weight: 20
---

1. 登录 [Docker Hub](https://hub.docker.com)。
2. 选择 **My Hub** > **Repositories**。
3. 在右上角选择 **Create repository**。
4. 选择 **Namespace**。

   你可以将其创建在自己的用户账户下，或任一你在其中为所有者或编辑者的组织命名空间下。

5. 指定 **Repository Name**。

   仓库名称需要满足：
    - 唯一性
    - 长度在 2 到 255 个字符之间
    - 仅包含小写字母、数字、连字符（`-`）与下划线（`_`）

   > [!NOTE]
   >
   > 仓库创建后无法重命名。

6. 填写 **Short description**。

   描述最多 100 个字符，会显示在搜索结果中。

7. 选择默认可见性。

   - **Public**：仓库会出现在 Docker Hub 搜索结果中，任何人都可以拉取。
   - **Private**：仓库不会出现在 Docker Hub 搜索结果中，仅你与协作者可访问；若选择组织命名空间，则具备相应角色或权限的成员可访问。详见[角色与权限](/manuals/enterprise/security/roles-and-permissions.md)。

   > [!NOTE]
   >
   > 若组织在创建新仓库时不确定应选择何种可见性，建议选择 **Private**。

8. 选择 **Create**。

创建完成后会进入 **General** 页面。你可以继续管理：

- [仓库信息](./manage/information.md)
- [访问控制](./manage/access.md)
- [镜像](./manage/hub-images/_index.md)
- [自动化构建](./manage/builds/_index.md)
- [Webhooks](./manage/webhooks.md)
- [镜像安全洞察](./manage/vulnerability-scanning.md)
