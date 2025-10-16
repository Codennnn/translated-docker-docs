---
description: 关于 Docker 安全、认证与组织管理的常见问题
keywords: Docker 安全, 常见问题, 认证, SSO, 漏洞报告, 会话管理
title: 通用安全常见问题
linkTitle: 通用
weight: 10
tags: [FAQ]
aliases:
- /faq/security/general/
---

## 我如何报告安全漏洞？

如果你发现了 Docker 的安全漏洞，请负责任地发送至 security@docker.com，以便 Docker 能够尽快处理。

## 多次登录失败后 Docker 会锁定账户吗？

在 5 分钟内连续 10 次登录失败时，Docker Hub 会锁定用户账户，锁定时长为 5 分钟。该策略同样适用于 Docker Hub、Docker Desktop 与 Docker Scout 的认证流程。

## 是否支持使用 YubiKey 等物理多因素认证（MFA）？

你可以通过 SSO 使用身份提供商（IdP）配置物理多因素认证（MFA）。请向你的 IdP 确认是否支持 YubiKey 等物理 MFA 设备。

## 会话如何管理？是否会过期？

Docker 使用令牌来管理用户会话，并设置不同的过期时长：

- Docker Desktop：90 天后自动登出，或 30 天无活动则登出
- Docker Hub 与 Docker Home：24 小时后自动登出

Docker 也支持通过 SAML 属性采用你的 IdP 默认会话超时策略。详见 [SSO 属性](/manuals/enterprise/security/provisioning/_index.md#sso-attributes)。

## Docker 如何区分员工与外包/承包商用户？

组织可使用已验证域名来区分用户类型。邮箱域名不属于已验证域的团队成员，会在组织中显示为“Guest（访客）”。

## 活动日志会保留多久？

Docker 活动日志保留 90 天。若需更长期的留存，请自行导出日志，或配置驱动将日志发送至你的内部门户/系统。

## 能导出用户及其角色和权限的列表吗？

可以。使用 [导出成员](../../admin/organization/members.md#export-members) 功能，可导出包含组织用户、角色与团队信息的 CSV 文件。

## Docker Desktop 如何存储认证信息？

Docker Desktop 使用宿主操作系统的安全密钥管理来存储认证令牌：

- macOS：[钥匙串（Keychain）](https://support.apple.com/guide/security/keychain-data-protection-secb0694df1a/web)
- Windows：[通过 Wincred 的安全与身份 API](https://learn.microsoft.com/en-us/windows/win32/api/wincred/)
- Linux：[Pass](https://www.passwordstore.org/)

## 在未启用 SCIM 的 SSO 场景下，如何移除不属于我 IdP 的用户？

若未启用 SCIM，则需要手动从组织中移除用户。SCIM 可以自动移除用户，但仅适用于启用 SCIM 之后新增的用户；启用前已添加的用户需要手动移除。

详见[管理组织成员](/manuals/admin/organization/members.md)。

## Scout 会从容器镜像中收集哪些元数据？

关于 Docker Scout 存储的元数据，参见[数据处理](/manuals/scout/deep-dive/data-handling.md)。

## 市场（Marketplace）扩展如何进行安全审查？

扩展的安全审查在规划中，目前尚未实施。扩展不属于 Docker 第三方风险管理计划的覆盖范围。

## 能否阻止用户向 Docker Hub 私有仓库推送镜像？

目前没有直接禁用私有仓库的设置。不过，借助[仓库访问管理](/manuals/enterprise/security/hardened-desktop/registry-access-management.md)，管理员可以通过管理控制台，控制开发者在 Docker Desktop 中可访问的仓库。
