---
title: 管理 Docker 账户
linkTitle: 管理账户
weight: 20
description: 了解如何管理你的 Docker 账户。
keywords: accounts, docker ID, account settings, account management, docker home
---

你可以通过 Docker Home 统一管理你的 Docker 账户，包括管理与安全设置。

> [!TIP]
>
> 如果你的账户隶属于启用了单点登录（SSO）的组织，你可能没有权限更改账户设置。
> 需要联系管理员来更新相关设置。

## 更新账户信息

你可以在 **Account settings** 页面查看账户信息，并更新以下内容：

- 全名
- 公司
- 所在地
- 个人网站
- Gravatar 邮箱

若要通过 Gravatar 新增或更新头像：

1. 创建 [Gravatar 账户](https://gravatar.com/)。
1. 创建你的头像。
1. 在 Docker 账户设置中添加你的 Gravatar 邮箱。

头像在 Docker 中更新可能需要一定时间。

## 更新邮箱地址

更新邮箱地址：

1. 登录你的 [Docker 账户](https://app.docker.com/login)。
1. 点击右上角头像，选择 **Account settings**。
1. 选择 **Email**。
1. 输入新邮箱地址，并输入密码确认修改。
1. 选择 **Send verification email**。Docker 会向新邮箱发送验证链接。

在完成验证之前，新邮箱会显示为未验证状态。你可以：

- 如有需要，可重新发送验证邮件。
- 在验证完成前，随时移除该未验证邮箱。

要完成邮箱验证，请在邮箱客户端中按 Docker 验证邮件的指引操作。

> [!NOTE]
>
> Docker 账户同一时间仅支持一个已验证邮箱，用于账户通知与安全相关通信。
> 你无法为账户添加多个已验证邮箱。

## 修改密码

你可以通过邮件发起密码重置来修改密码。步骤如下：

1. 登录你的 [Docker 账户](https://app.docker.com/login)。
1. 点击右上角头像，选择 **Account settings**。
1. 选择 **Password**，再选择 **Reset password**。
1. Docker 会向你发送密码重置邮件，并附带重置指引。

## 管理双重验证

更新双重验证（2FA）设置：

1. 登录你的 [Docker 账户](https://app.docker.com/login)。
1. 点击右上角头像，选择 **Account settings**。
1. 选择 **2FA**。

更多信息参见
[启用双重验证](../security/2fa/_index.md)。

## 管理个人访问令牌

管理个人访问令牌：

1. 登录你的 [Docker 账户](https://app.docker.com/login)。
1. 点击右上角头像，选择 **Account settings**。
1. 选择 **Personal access tokens**。

更多信息参见
[创建与管理访问令牌](../security/access-tokens.md)。

## 管理已连接账户

你可以解除已连接的 Google 或 GitHub 账户：

1. 登录你的 [Docker 账户](https://app.docker.com/login)。
1. 点击右上角头像，选择 **Account settings**。
1. 选择 **Connected accounts**。
1. 在已连接账号上选择 **Disconnect**。

若要彻底解除与 Docker 的关联，还需在 Google 或 GitHub 平台解除与 Docker 的连接。更多信息参见：

- [管理你的 Google 账户与第三方之间的连接](https://support.google.com/accounts/answer/13533235?hl=en)
- [查看与撤销 GitHub 应用授权](https://docs.github.com/en/apps/using-github-apps/reviewing-and-revoking-authorization-of-github-apps)

## 转换账户类型

如需将个人账户转换为组织，请参见
[将账户转换为组织](../admin/organization/convert-account.md)。

## 停用你的账户

关于停用账户，参见
[停用用户账户](./deactivate-user-account.md)。
