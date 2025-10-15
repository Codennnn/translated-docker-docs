---
description: 了解 Docker Hub 中与仓库相关的个人设置
keywords: Docker Hub, Hub, 仓库, 设置
title: 仓库的个人设置
linkTitle: 个人设置
toc_max: 3
weight: 50
---

你可以为自己的账户配置与仓库相关的个人设置，包括默认仓库隐私与自动构建通知。

## 默认仓库隐私

在 Docker Hub 中创建新仓库时，你可以指定仓库可见性；之后也可随时修改。

当你使用 `docker push` 推送到一个尚不存在的仓库时，默认设置非常有用。在这种情况下，Docker Hub 会根据你的默认仓库隐私自动创建仓库。

### 配置默认仓库隐私

1. 登录 [Docker Hub](https://hub.docker.com)。
2. 选择 **My Hub** > **Settings** > **Default privacy**。
3. 选择新建仓库的 **Default privacy**。

   - **Public**：所有新建仓库都会出现在 Docker Hub 搜索结果中，任何人都可以拉取。
   - **Private**：所有新建仓库不会出现在 Docker Hub 搜索结果中，仅你与协作者可访问；如果仓库创建在组织命名空间下，则具备相应角色或权限的成员可访问。

4. 选择 **Save**。

## 自动构建通知（Autobuild notifications）

你可以为所有使用自动构建的仓库配置邮件通知。

### 配置自动构建通知

1. 登录 [Docker Hub](https://hub.docker.com)。
2. 选择 **My Hub** > **Repositories** > **Settings** > **Notifications**。
3. 选择希望通过邮件接收的通知类型。

   - **Off**：不接收通知。
   - **Only failures**：仅接收构建失败通知。
   - **Everything**：接收成功与失败的全部通知。

4. 选择 **Save**。
