---
title: SSO 强制启用常见问题
linkTitle: 强制
description: 关于强制启用 Docker 单点登录（SSO）及其对用户影响的常见问题
keywords: SSO 强制, 单点登录, 个人访问令牌, CLI 认证, 来宾用户
tags: [FAQ]
aliases:
- /single-sign-on/enforcement-faqs/
- /faq/security/single-sign-on/enforcement-faqs/
---

## Docker 的 SSO 是否支持通过命令行进行认证？

启用并强制 SSO 后，[将不再允许使用密码访问 Docker CLI](/security/security-announcements/#deprecation-of-password-logins-on-cli-when-sso-enforced)。你必须使用个人访问令牌（PAT）进行 CLI 认证。

每位用户都需要创建一个 PAT 才能访问 CLI。如何创建 PAT，参见[管理个人访问令牌](/security/access-tokens/)。在 SSO 强制前已使用 PAT 的用户可继续沿用原有 PAT。

## SSO 会如何影响自动化系统与 CI/CD 流水线？

在强制 SSO 之前，你需要[创建个人访问令牌](/security/access-tokens/)以替换自动化系统与 CI/CD 流水线中的密码。

## 是否可以先开启 SSO 而暂不强制？

可以。你可以开启 SSO 而不立即强制。在登录页面，用户可在 Docker ID（邮箱+密码）与域邮箱（SSO）之间选择其一。

## 已强制 SSO，但仍有用户能用用户名和密码登录，为什么？

受邀加入组织但不属于已注册域的来宾用户不通过你的 SSO 身份提供商进行登录。SSO 强制仅适用于属于你已验证域的用户。

## 上生产前能否先测试 SSO 功能？

可以。你可以使用 5 席位的 Business 订阅创建测试组织。测试时建议开启但不强制 SSO，否则所有域邮箱用户会被强制登录到测试环境。

## “强制启用 SSO” 与 “强制登录” 有何区别？

这是两个独立功能，可分别或共同使用：

- 强制启用 SSO：确保用户使用 SSO 凭据而非 Docker ID 登录，便于统一凭据管理。
- 强制登录 Docker Desktop：确保用户始终登录属于你组织的账户，以便统一生效安全设置与订阅权益。

详见[强制登录 Desktop](/manuals/enterprise/security/enforce-sign-in/_index.md#enforcing-sign-in-versus-enforcing-single-sign-on-sso)。
