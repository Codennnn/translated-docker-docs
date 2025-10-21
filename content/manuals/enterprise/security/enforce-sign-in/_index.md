---
title: 强制 Docker Desktop 登录
linkTitle: 强制登录
description: 要求用户登录 Docker Desktop 以访问组织权益和安全功能
toc_max: 2
keywords: 身份验证, registry.json, 配置, 强制登录, docker desktop, 安全, .plist, 注册表项, mac, windows, 组织
tags: [admin]
aliases:
 - /security/for-admins/configure-sign-in/
 - /docker-hub/configure-sign-in/
 - /security/for-admins/enforce-sign-in/
weight: 30
---

{{< summary-bar feature_name="Enforce sign-in" >}}

默认情况下，用户无需登录到您的组织即可访问 Docker Desktop。
当用户未以组织成员身份登录时，他们将无法享受订阅权益，并且可能绕过为您的组织配置的安全功能。

您可以根据环境配置，使用多种方法强制用户登录：

- [注册表项方法（仅限 Windows）](methods.md#registry-key-method-windows-only)
- [配置文件方法（仅限 Mac）](methods.md#configuration-profiles-method-mac-only)
- [`.plist` 方法（仅限 Mac）](methods.md#plist-method-mac-only)
- [`registry.json` 方法（适用于所有平台）](methods.md#registryjson-method-all)

本页面概述了强制登录功能的工作原理。

## 强制登录的工作原理

当 Docker Desktop 检测到注册表项、`.plist` 文件或 `registry.json` 文件时：

- 系统会显示"需要登录！"提示，要求用户必须以组织成员身份登录才能使用 Docker Desktop。
- 如果用户使用非组织成员账户登录，系统会自动将其登出并阻止使用 Docker Desktop。用户可以选择**登录**按钮尝试使用其他账户重新登录。
- 当用户使用组织成员账户登录后，可以正常使用 Docker Desktop。
- 当用户登出后，"需要登录！"提示会再次出现，除非重新登录，否则无法使用 Docker Desktop。

> [!NOTE]
>
> 强制 Docker Desktop 登录不会影响 Docker CLI 的访问权限。只有强制实施单点登录（SSO）的组织才会限制 CLI 访问。

## 强制登录与强制单点登录（SSO）的区别

强制 Docker Desktop 登录和[强制 SSO](/manuals/enterprise/security/single-sign-on/connect.md#optional-enforce-sso)是两种不同的功能，服务于不同的目的：


| 强制方式                         | 描述                                                             | 优势                                                                                                                                                                                                                                                   |
|:----------------------------------|:----------------------------------------------------------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 仅强制登录                       | 用户必须先登录才能使用 Docker Desktop                 | 确保用户享受订阅权益并应用安全功能。此外，您可以了解用户活动情况。                                                                                                    |
| 仅强制单点登录（SSO）            | 如果用户登录，必须使用 SSO 进行登录                  | 集中身份验证并强制执行身份提供商设置的统一策略。                                                                                                                                                                     |
| 同时强制登录和 SSO               | 用户必须使用 SSO 登录才能使用 Docker Desktop       | 确保用户享受订阅权益并应用安全功能。此外，您可以了解用户活动情况。同时集中身份验证并强制执行身份提供商设置的统一策略。 |
| 两者都不强制                     | 如果用户登录，可以使用 SSO 或 Docker 凭据 | 允许用户无障碍访问 Docker Desktop，但会降低安全性和可见性。                                                                                                                                                  |

## 后续步骤

- 要设置强制登录，请参阅[配置强制登录](/manuals/enterprise/security/enforce-sign-in/methods.md)。
- 要配置 SSO 强制执行，请参阅[强制 SSO](/manuals/enterprise/security/single-sign-on/connect.md)。
