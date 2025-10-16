---
description: 扩展功能
keywords: Docker Extensions, Docker Desktop, Linux, Mac, Windows, feedback
title: Docker 扩展的设置与反馈
linkTitle: 设置与反馈
weight: 40
aliases:
 - /desktop/extensions/settings-feedback/
---

## 设置

### 启用或停用扩展

Docker 扩展默认处于启用状态。要更改设置：

1. 进入 **Settings**。
2. 打开 **Extensions** 页签。
3. 在 **Enable Docker Extensions** 旁，选中或取消复选框以设置期望状态。
4. 在右下角选择 **Apply** 应用设置。

> [!NOTE]
>
> 如果你是[组织所有者](/manuals/admin/organization/manage-a-team.md#organization-owner)，可以为你的用户关闭扩展。打开 `settings-store.json`，将 `"extensionsEnabled"` 设置为 `false`。
> `settings-store.json` 文件位置（Docker Desktop 4.34 及更早版本为 `settings.json`）：
>   - Mac：`~/Library/Group Containers/group.com.docker/settings-store.json`
>   - Windows：`C:\Users\[USERNAME]\AppData\Roaming\Docker\settings-store.json`
>
> 你也可以使用 [Hardened Docker Desktop](/manuals/enterprise/security/hardened-desktop/_index.md) 完成此操作。

### 启用或禁用非市场扩展

你可以通过扩展市场或使用 Extensions SDK 工具安装扩展。你也可以选择仅允许“已发布”的扩展（即在扩展市场通过审核并上架的扩展）。

1. 进入 **Settings**。
2. 打开 **Extensions** 页签。
3. 在 **Allow only extensions distributed through the Docker Marketplace** 旁，选中或取消复选框以设置期望状态。
4. 在右下角选择 **Apply** 应用设置。

### 查看扩展创建的容器

默认情况下，扩展创建的容器不会显示在 Docker Desktop 仪表盘和 Docker CLI 的容器列表中。若要显示，请更新设置：

1. 进入 **Settings**。
2. 打开 **Extensions** 页签。
3. 在 **Show Docker Extensions system containers** 旁，选中或取消复选框以设置期望状态。
4. 在右下角选择 **Apply** 应用设置。

> [!NOTE]
>
> 仅启用扩展本身不会占用计算机资源（CPU/内存）。
>
> 具体扩展的资源占用取决于其功能与实现，但启用扩展并不会预留资源或产生额外成本。

## 提交反馈

你可以通过扩展作者提供的 Slack 频道或 GitHub 进行反馈。要对某个扩展提交反馈：

1. 打开 Docker Desktop 仪表盘，选择 **Manage** 页签。
   这里会显示你已安装的扩展列表。
2. 选择你要反馈的扩展。
3. 滚动到该扩展描述的底部，根据扩展提供的选项，选择：
    - Support
    - Slack
    - Issues（将跳转到 Docker Desktop 之外的页面提交反馈）。

如果某个扩展未提供反馈渠道，可联系我们转交你的意见。请在 **Extensions Marketplace** 右侧选择 **Give feedback**。
