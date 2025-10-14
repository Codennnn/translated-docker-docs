---
description: 了解在 Docker 仪表板的 Images 视图中可以执行的操作
keywords: Docker 仪表板, 管理, 容器, 图形界面, 仪表板, 镜像, 用户手册
title: 浏览 Docker Desktop 的 Images 视图
linkTitle: 镜像（Images）
weight: 20
---


**Images** 视图会展示你的 Docker 镜像列表，你可以基于镜像运行容器、从 Docker Hub 拉取镜像的最新版本，以及检查镜像。它还会显示镜像漏洞的概要信息。此外，**Images** 视图提供清理选项，用于删除不需要的镜像以回收磁盘空间。若你已登录，还能看到你和你的组织在 Docker Hub 上共享的镜像。更多信息参见[浏览你的镜像](images.md)。

**Images** 视图让你无需使用 CLI 即可管理 Docker 镜像。默认情况下，它会显示本地磁盘上的所有 Docker 镜像。

登录 Docker Hub 后，你还可以查看 Hub 上的镜像，从而与团队协作并直接通过 Docker Desktop 管理镜像。

**Images** 视图支持核心操作：基于镜像运行容器、从 Docker Hub 拉取镜像的最新版本、将镜像推送到 Docker Hub、检查镜像。

同时会显示镜像的以下元数据：
- Tag（标签）
- Image ID（镜像 ID）
- 创建日期
- 镜像大小

被正在运行或已停止容器使用的镜像旁会显示 **In Use** 标签。你可以通过搜索栏右侧的 **More options** 菜单选择需要展示的信息，并按需切换。

**Images on disk** 状态栏会显示镜像数量、镜像占用的总磁盘空间，以及该信息的上次刷新时间。

## 管理你的镜像

使用 **Search**（搜索）字段查找特定镜像。

你可以按以下方式对镜像进行分类：

- In use（正在使用）
- Unused（未使用）
- Dangling（悬空）

## 基于镜像运行容器

在 **Images** 视图中，将鼠标悬停在某个镜像上并选择 **Run**。

当出现提示时，你可以：

- 展开 **Optional settings**（可选设置），指定名称、端口、卷、环境变量后选择 **Run**。
- 不设置可选项，直接选择 **Run**。

## 检查镜像

选择镜像行即可检查镜像。检查镜像会显示其详细信息，例如：

- 镜像历史
- 镜像 ID
- 创建日期
- 镜像大小
- 构成镜像的各层
- 使用的基础镜像
- 发现的漏洞
- 镜像内的包

[Docker Scout](/manuals/scout/_index.md) 提供上述漏洞信息。
更多关于该视图的说明，参见[镜像详情视图](/manuals/scout/explore/image-details-view.md)。

## 从 Docker Hub 拉取最新镜像

在列表中选择镜像，点击 **More options** 按钮并选择 **Pull**。

> [!NOTE]
>
> 要拉取镜像的最新版本，镜像仓库必须存在于 Docker Hub；拉取私有镜像需要先登录。

## 将镜像推送到 Docker Hub

在列表中选择镜像，点击 **More options** 按钮并选择 **Push to Hub**。

> [!NOTE]
>
> 仅当镜像属于你的 Docker ID 或你的组织时，才能推送到 Docker Hub。也就是说，镜像标签中必须包含正确的用户名/组织名。

## 删除镜像

> [!NOTE]
>
> 若镜像被正在运行或已停止的容器使用，你需要先移除关联容器，才能删除镜像。

未使用镜像指未被任何运行或停止的容器使用的镜像。若使用相同标签构建了镜像的新版本，旧镜像会成为悬空镜像（dangling）。

要删除单个镜像，选择垃圾桶图标。

## Docker Hub 仓库

**Images** 视图也支持管理与操作 Docker Hub 仓库中的镜像。
默认进入 Docker Desktop 的 **Images** 页面时，会显示本地镜像存储中的镜像列表。
顶部的 **Local** 与 **Docker Hub repositories** 选项卡可在本地镜像与远程 Docker Hub 仓库镜像之间切换。

切换到 **Docker Hub repositories** 选项卡时，如果你尚未登录，会提示你登录 Docker Hub。登录后，会显示你有权限访问的组织与仓库中的镜像列表。

在下拉菜单中选择某个组织，可查看该组织的仓库列表。

如果你在仓库上启用了 [Docker Scout](../../scout/_index.md)，则镜像标签旁会显示镜像分析结果（若你的 Docker 组织符合条件，还会显示[健康评分](/manuals/scout/policy/scores.md)）。

将鼠标悬停在某个镜像标签上会出现两个选项：

- **Pull**：从 Docker Hub 拉取该镜像的最新版本。
- **View in Hub**：打开 Docker Hub 页面并查看镜像详情。

## 其他资源

- [什么是镜像？](/get-started/docker-concepts/the-basics/what-is-an-image.md)
