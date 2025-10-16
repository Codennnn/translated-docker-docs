---
title: 停用 Docker 账户
linkTitle: 停用账户
weight: 30
description: 了解如何停用 Docker 用户账户。
keywords: Docker Hub, 删除, 停用, 账户, 账户管理, 删除 Docker 账户, 关闭 Docker 账户, 禁用 Docker 账户
---

本文介绍如何停用个人 Docker 账户，以及停用前需要满足的前提条件。

如需了解停用组织的信息，请参见
[停用组织](../admin/organization/deactivate-account.md)。

> [!WARNING]
>
> 停用账户后，所有使用你 Docker 账户的 Docker 产品与服务将无法访问。

## 先决条件

在停用 Docker 账户之前，请确保满足以下要求：

- 如果你是组织或公司的所有者，必须先退出该组织或公司，才能停用 Docker 账户：
    1. 登录 [Docker Home](https://app.docker.com/admin) 并选择你的组织。
    1. 选择 **Members** 并找到你的用户名。
    1. 打开 **Actions** 菜单，然后选择 **Leave organization**。
- 如果你是组织的唯一所有者，必须将所有者角色转移给组织内其他成员，然后将自己移出组织，或直接停用该组织。类似地，如果你是公司的唯一所有者，要么新增一位公司所有者并移除自己，要么停用该公司。
- 如果你有有效的 Docker 订阅，请[降级为 Docker Personal 订阅](../subscription/change.md)。
- 下载你希望保留的镜像与标签。使用 `docker pull -a <image>:<tag>`。
- 取消关联你的 [GitHub 账户](../docker-hub/repos/manage/builds/link-source.md#unlink-a-github-user-account)。

## 停用

完成上述步骤后，你即可停用账户。

> [!WARNING]
>
> 停用账户为永久性操作，无法撤销。请务必先备份重要数据。

1. 登录 [Docker Home](https://app.docker.com/login)。
1. 点击头像打开下拉菜单。
1. 选择 **Account settings**。
1. 选择 **Deactivate**。
1. 选择 **Deactivate account**，再次确认。

## 删除个人数据

停用账户不会删除你的个人数据。若要申请删除个人数据，请填写 Docker 的
[隐私请求表单](https://preferences.docker.com/)。
