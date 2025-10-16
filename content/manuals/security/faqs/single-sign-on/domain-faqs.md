---
title: SSO 域名常见问题
linkTitle: 域名
description: 关于 Docker 单点登录（SSO）的域名验证与管理常见问题
keywords: SSO 域名, 域名验证, DNS, TXT 记录, 单点登录
tags: [FAQ]
aliases:
- /single-sign-on/domain-faqs/
- /faq/security/single-sign-on/domain-faqs/
---

## 是否可以添加子域名？

可以。你可以在 SSO 连接中添加子域名。所有邮箱地址必须使用你已添加到该连接中的域名。请确认 DNS 服务商支持为同一域名配置多个 TXT 记录。

## DNS TXT 记录是否需要长期保留？

在完成一次性验证并添加域名后，可以移除该 TXT 记录。但如果组织更换身份提供商并需要重新配置 SSO，则需要再次完成域名验证。

## 同一个域名能否为多个组织完成验证？

在“组织”层级，无法将同一域名验证给多个组织。若需对多个组织复用同一域名验证，你需要 Docker Business 订阅并创建“公司”。“公司”支持在公司层级集中管理组织与域名验证。
