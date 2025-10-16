---
description: 关于 Docker 单点登录（SSO）的常见问题
keywords: Docker, Docker Hub, SSO 常见问题, 单点登录, 管理, 安全
title: 通用 SSO 常见问题
linkTitle: 通用
weight: 10
tags: [FAQ]
aliases:
- /single-sign-on/faqs/
- /faq/security/single-sign-on/faqs/
- /single-sign-on/saml-faqs/
- /faq/security/single-sign-on/saml-faqs/
- /security/faqs/single-sign-on/saml-faqs/
---

## Docker 支持哪些 SSO 流程？

Docker 支持由服务提供商发起（SP-initiated）的 SSO 流程。用户需先在 Docker Hub 或 Docker Desktop 发起登录以开始 SSO 认证。

## Docker 的 SSO 是否支持多因素认证？

组织启用 SSO 后，多因素认证由身份提供商（IdP）控制，而非 Docker 平台。

## 使用 SSO 时，是否还能保留我的 Docker ID？

拥有个人 Docker ID 的用户仍然保留其仓库、镜像与资产的所有权。启用 SSO 强制后，使用公司域邮箱的现有账户会与组织关联；首次登录且尚无账户的用户，会自动创建新账户与 Docker ID。

## 配置 SSO 是否需要特定的防火墙规则？

只要能够访问 `login.docker.com`，通常无需额外的防火墙规则。若在配置 SSO 时遇到访问问题，请在组织的防火墙策略中放行该域名。

## Docker 是否采用 IdP 的默认会话超时？

是的。Docker 通过自定义的 `dockerSessionMinutes` SAML 属性来支持 IdP 的会话超时，而不是使用标准的 `SessionNotOnOrAfter` 元素。详见 [SSO 属性](/manuals/enterprise/security/provisioning/_index.md#sso-attributes)。
