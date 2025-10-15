---
description: 了解如何在 Docker Hub 上归档或取消归档仓库
keywords: Docker Hub, Hub, 仓库, 归档, 取消归档
title: 归档或取消归档仓库
linkTitle: 归档
toc_max: 3
weight: 35
---

你可以在 Docker Hub 上将仓库设置为“归档”以标记其为只读，并表明该仓库已不再积极维护。这有助于防止在工作流中继续使用过时或不受支持的镜像。若有需要，被归档的仓库也可以取消归档。

对于一年以上未更新的仓库，Docker Hub 会在[**Repositories** 页面](https://hub.docker.com/repositories/)的仓库名称旁显示一个图标（{{< inline-image src="./images/outdated-icon.webp" alt="outdated icon" >}}）以突出提示。建议你审查这些仓库，并在必要时进行归档。

当仓库被归档时，将发生以下变化：

- 无法修改仓库信息。
- 无法向该仓库推送新镜像。
- 公共仓库页面会显示 **Archived** 标签。
- 用户仍可拉取该仓库中的镜像。

你可以取消归档一个已归档的仓库以移除其归档状态。取消归档后：

- 可以修改仓库信息。
- 可以向仓库推送新镜像。
- 公共仓库页面的 **Archived** 标签将被移除。

## 归档仓库

1. Sign in to [Docker Hub](https://hub.docker.com).
2. Select **My Hub** > **Repositories**.

   A list of your repositories appears.

3. Select a repository.

   The **General** page for the repository appears.

4. Select the **Settings** tab.
5. 选择 **Archive repository**。
6. Enter the name of your repository to confirm.
7. 选择 **Archive**。

## 取消归档仓库

1. Sign in to [Docker Hub](https://hub.docker.com).
2. Select **My Hub** > **Repositories**.

   A list of your repositories appears.

3. Select a repository.

   The **General** page for the repository appears.

4. Select the **Settings** tab.
5. 选择 **Unarchive repository**。