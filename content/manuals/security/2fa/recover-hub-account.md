---
title: 找回你的 Docker 账号
description: 找回 Docker 账号并管理双重身份验证的恢复代码
keywords: 账号找回, 双重身份验证, 2FA, 恢复代码, Docker Hub 安全
aliases:
 - /docker-hub/2fa/recover-hub-account/
 - /security/for-developers/2fa/recover-hub-account/
 - /security/2fa/new-recovery-code/
weight: 20
---

本文介绍如何找回你的 Docker 账号，以及如何管理双重身份验证（2FA）的恢复代码。

## 生成新的恢复代码

如果你遗失了 2FA 恢复代码，但仍可登录 Docker Hub 账号，则可以生成新的恢复代码。

1. 使用用户名与密码登录你的 [Docker 账号](https://app.docker.com/login)。
1. 点击头像，从下拉菜单中选择 **Account settings**（账号设置）。
1. 选择 **2FA**。
1. 输入密码并选择 **Confirm**（确认）。
1. 选择 **Generate new code**（生成新代码）。

系统会为你生成新的恢复代码。点击可见性图标以查看代码。请保存并妥善保管该代码。

## 无法访问时找回账号

如果你同时无法使用身份验证器应用，也没有恢复代码：

1. 使用用户名与密码登录你的 [Docker 账号](https://app.docker.com/login)。
1. 选择 **I've lost my authentication device**（我丢失了验证设备）与 **I've lost my recovery code**（我丢失了恢复代码）。
1. 填写并提交 [联系支持表单](https://hub.docker.com/support/contact/?category=2fa-lockout)。

你必须在联系支持表单中填写与你 Docker ID 绑定的主邮箱地址，以便接收找回账号的后续指引。
