---
title: SSO 用户管理常见问题
linkTitle: 用户管理
description: 使用 Docker 单点登录（SSO）进行用户管理的常见问题
keywords: SSO 用户管理, 用户供应, SCIM, 即时供应, 组织成员
tags: [FAQ]
aliases:
- /single-sign-on/users-faqs/
- /faq/security/single-sign-on/users-faqs/
---

## 是否需要手动将用户添加到组织？

不需要。只要用户在你的身份提供商（IdP）中已有账户，用户使用域邮箱登录 Docker 并完成认证后，会自动加入组织。

## 用户能否使用不同邮箱通过 SSO 认证？

所有用户必须使用 SSO 配置阶段指定的邮箱域进行认证。若未强制启用 SSO，邮箱不属于已验证域的受邀用户可作为来宾使用用户名与密码登录。

## 用户如何得知自己被加入某个 Docker 组织？

启用 SSO 后，用户下次登录 Docker Hub 或 Docker Desktop 时会被提示通过 SSO 进行认证。系统会识别其域邮箱，并引导其使用 SSO 凭据登录。

对于 CLI 访问，用户必须使用个人访问令牌（PAT）进行认证。

## 能否将现有的非 SSO 用户转换为 SSO 账户？

可以。请确保用户满足以下条件：

- 使用公司域邮箱，并在你的 IdP 中拥有账户
- Docker Desktop 版本为 4.4.2 或更高
- 已创建个人访问令牌（PAT），用于替换 CLI 密码
- 已将 CI/CD 流水线更新为使用 PAT 而非密码

详细步骤参见[配置单点登录](/manuals/enterprise/security/single-sign-on/configure.md)。

## Docker 的 SSO 是否与 IdP 完全同步？

Docker 的 SSO 默认提供即时（JIT）供应：用户在通过 SSO 认证时才会被开通账号。若用户离开组织，管理员需手动[移除用户](../../../admin/organization/members.md#remove-a-member-or-invitee)。

[SCIM](/manuals/enterprise/security/provisioning/scim.md) 可与用户和组进行完全同步。使用 SCIM 时，建议关闭 JIT，将所有自动开通交由 SCIM 处理。

你也可以使用 [Docker Hub API](/reference/api/hub/latest/) 完成相关流程。

## 关闭即时供应（JIT）会如何影响用户登录？

在管理控制台配合 SCIM 关闭 JIT 后，用户必须已是组织成员或拥有待接受的邀请才可访问 Docker。不满足条件的用户将收到“Access denied”错误，需要管理员邀请。

详见[禁用 JIT 供应下的 SSO 认证](/manuals/enterprise/security/provisioning/just-in-time.md#sso-authentication-with-jit-provisioning-disabled)。

## 是否可以在没有邀请的情况下加入组织？

不可以（在未启用 SSO 的前提下）。加入组织需要组织所有者的邀请。启用并强制 SSO 后，使用已验证域邮箱的用户在登录时可自动加入组织。

## 启用 SCIM 后，现有的持证用户会有什么变化？

启用 SCIM 并不会立即移除或修改现有的持证用户。他们会保留当前的访问与角色，但从启用起将由你的 IdP 管理。若随后关闭 SCIM，此前由 SCIM 管理的用户仍保留在 Docker 中，但不再依据 IdP 自动更新。

## 用户信息是否会在 Docker Hub 上公开可见？

所有 Docker 账户都与各自的命名空间关联公开资料页。若不希望显示用户信息（如姓名），可在 SSO 与 SCIM 的映射中移除相应属性，或使用其他标识替代用户姓名。
