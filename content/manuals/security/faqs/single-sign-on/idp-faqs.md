---
title: SSO 身份提供商常见问题
linkTitle: 身份提供商
description: 关于 Docker SSO 与身份提供商（IdP）配置的常见问题
keywords: 身份提供商, SSO IdP, SAML, Azure AD, Entra ID, 证书管理
tags: [FAQ]
aliases:
- /single-sign-on/idp-faqs/
- /faq/security/single-sign-on/idp-faqs/
---

## Docker SSO 是否支持多个身份提供商？

支持。Docker 允许配置多个 IdP，一个域名也可关联多个 IdP。Docker 支持 Entra ID（原 Azure AD）以及兼容 SAML 2.0 的身份提供商。

## 配置好 SSO 后还能更换身份提供商吗？

可以。先在 Docker 的 SSO 连接中删除现有 IdP 配置，再[使用新的 IdP 进行 SSO 配置](/manuals/enterprise/security/single-sign-on/connect.md)。若已开启强制 SSO，请在更新连接前先关闭强制。

## 从 IdP 处需要哪些信息来配置 SSO？

在 Docker 中启用 SSO，需要从 IdP 获取：

- SAML：Entity ID、ACS URL、Single Logout URL、公钥 X.509 证书
- Entra ID（原 Azure AD）：Client ID、Client Secret、AD Domain

## 现有证书过期会发生什么？

证书过期时，请联系你的 IdP 获取新的 X.509 证书，并在 Docker 管理控制台的[SSO 配置](/manuals/enterprise/security/single-sign-on/manage.md#manage-sso-connections)中更新证书。

## 启用 SSO 时，若 IdP 宕机会怎样？

如果已强制启用 SSO，IdP 宕机时用户将无法访问 Docker Hub。但用户仍可通过 CLI 使用个人访问令牌（PAT）拉取 Docker Hub 镜像。

如果仅开启但未强制 SSO，则用户可回退为用户名/密码方式进行认证。

## 机器人账号访问组织时是否也需要席位？

需要。与普通用户一样，机器人账号也需要使用非别名的域邮箱在 IdP 中注册，并占用 Docker Hub 席位。你可以在 IdP 中添加机器人账号，并创建访问令牌替代其他凭据。

## SAML SSO 是否使用即时供应（JIT）？

默认情况下，SSO 实现采用即时（JIT）供应。若启用 SCIM 自动供应，可在管理控制台选择关闭 JIT。参见[即时供应](/security/for-admins/provisioning/just-in-time/)。

## 我的 Entra ID SSO 连接失败且出现错误，如何排查？

请确认已在 Entra ID 中为你的 SSO 连接配置必要的 API 权限，并在租户中授予管理员同意。参见[Entra ID（原 Azure AD）文档](https://learn.microsoft.com/en-us/azure/active-directory/manage-apps/grant-admin-consent?pivots=portal#grant-admin-consent-in-app-registrations)。
