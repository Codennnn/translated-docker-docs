---
title: 为 Docker 账号启用双重身份验证
linkTitle: 双重身份验证
description: 在 Docker 账号上启用或关闭双重身份验证，提升安全性与账号保护
keywords: 双重身份验证, 2FA, Docker Hub 安全, 账号安全, TOTP, 身份验证器应用, 关闭 2FA
weight: 20
aliases:
 - /docker-hub/2fa/
 - /security/2fa/disable-2fa/
 - /security/for-developers/2fa/
 - /security/for-developers/2fa/disable-2fa/
---

双重身份验证（2FA）通过在登录时除了密码还需要输入一次性验证码，为你的 Docker 账号增加了一层关键的安全保障。即使密码泄露，也能有效阻止未授权访问。

启用 2FA 后，Docker 会生成与你账号唯一绑定的恢复代码。请妥善保存该代码，当你无法使用身份验证器应用时，它可用于找回账号。

## 关键优势

双重身份验证能够显著提升账号安全性：

- 密码泄露防护：即使密码被窃取或泄漏，没有第二因素，攻击者也无法登录。
- 安全的 CLI 访问：启用 2FA 后，Docker CLI 认证需要第二因素；这也确保自动化工具使用个人访问令牌（PAT）而非密码。
- 符合合规要求：许多组织要求在访问开发与生产资源时启用 2FA。
- 安心保障：你的仓库、镜像与账号设置将受到业界标准的安全机制保护。

## 先决条件

在启用 2FA 之前，你需要：

- 一部安装了基于时间的一次性密码（TOTP）身份验证器应用的手机或设备
- 可用的 Docker 账号密码

## 启用双重身份验证

为你的 Docker 账号启用 2FA：

1. 登录你的 [Docker 账号](https://app.docker.com/login)。
1. 点击头像，从下拉菜单中选择 **Account settings**（账号设置）。
1. 选择 **2FA**。
1. 输入账号密码，并选择 **Confirm**（确认）。
1. 保存恢复代码并妥善保管。若日后无法使用验证器应用，你可以用该代码找回账号。
1. 使用 TOTP 应用扫描二维码或手动输入文本代码。
1. 绑定完成后，在输入框中填写六位数字验证码。
1. 选择 **Enable 2FA**（启用 2FA）。

此后你的账号将开启双重身份验证。每次登录都需要在验证器应用中获取并输入一次性验证码。

## 关闭双重身份验证

> [!WARNING]
>
> 关闭双重身份验证会降低 Docker 账号的安全性。

1. 登录你的 [Docker 账号](https://app.docker.com/login)。
2. 点击头像，从下拉菜单中选择 **Account settings**（账号设置）。
3. 选择 **2FA**。
4. 输入密码并选择 **Confirm**（确认）。
5. 选择 **Disable 2FA**（关闭 2FA）。
