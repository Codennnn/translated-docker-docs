---
title: 创建 Docker 账户
linkTitle: 创建账户
weight: 10
description: 了解如何注册 Docker ID 并登录你的账户
keywords: accounts, docker ID, billing, paid plans, support, Hub, Store, Forums, knowledge
  base, beta access, email, activation, verification
aliases:
- /docker-hub/accounts/
- /docker-id/
---

你可以使用邮箱免费创建 Docker 账户，或通过 Google/GitHub 注册。创建唯一的 Docker ID 后，你即可访问所有 Docker 产品，包括 Docker Hub、Docker Desktop 与 Docker Scout。

你的 Docker ID 会成为托管的 Docker 服务与 [Docker 论坛](https://forums.docker.com/) 的用户名。

> [!TIP]
>
> 了解 [Docker 的订阅方案](https://www.docker.com/pricing/)，看看还能为你提供哪些能力。

## 创建账户

你可以使用邮箱注册，或使用 Google/GitHub 账户完成注册。

### 使用邮箱注册

1. 访问 [Docker 注册页面](https://app.docker.com/signup/)。
1. 输入一个唯一且有效的邮箱地址。
1. 输入一个作为 Docker ID 的用户名。注意：若将来停用该账户，该 Docker ID 将无法再次使用。

    用户名要求：
    - 长度需在 4 到 30 个字符之间
    - 只能包含数字与小写字母

1. 设置至少 9 个字符的密码。
1. 选择 **Sign Up**。
1. 打开邮箱，Docker 会向你提供的地址发送验证邮件。
1. 完成邮箱验证以完成注册。

> [!NOTE]
>
> 你必须完成邮箱验证，才能完整使用 Docker 的各项功能。

### 使用 Google 或 GitHub 注册

> [!IMPORTANT]
>
> 使用第三方账号注册前，你必须先在对应平台完成邮箱验证。

1. 访问 [Docker 注册页面](https://app.docker.com/signup/)。
1. 选择第三方提供方：Google 或 GitHub。
1. 选择要与 Docker 账户关联的第三方账号。
1. 选择 **Authorize Docker** 授权 Docker 访问你的第三方账号信息，随后会跳转回注册页面。
1. 输入一个作为 Docker ID 的用户名。

    用户名要求：
    - 长度需在 4 到 30 个字符之间
    - 只能包含数字与小写字母
1. 选择 **Sign up**。

## 登录你的账户

你可以使用邮箱、Google/GitHub 账户登录，或通过 Docker CLI 登录。

### 使用邮箱或 Docker ID 登录

1. 访问 [Docker 登录页面](https://login.docker.com)。
1. 输入邮箱地址或 Docker ID，选择 **Continue**。
1. 输入密码，选择 **Continue**。

若需重置密码，请参阅[重置密码](#reset-your-password)。

### 使用 Google 或 GitHub 登录

> [!IMPORTANT]
>
> 你的 Google 或 GitHub 账号必须已完成邮箱验证。

你可以使用 Google 或 GitHub 的凭据登录。如果第三方账号使用的邮箱与现有 Docker ID 一致，系统会自动完成关联。

如果不存在对应的 Docker ID，Docker 会为你创建一个新账户。

目前 Docker 不支持将多个登录方式关联到同一个 Docker ID。

### 使用 CLI 登录

使用 `docker login` 命令从命令行完成认证。详见 [`docker login`](/reference/cli/docker/login/)。

> [!WARNING]
>
> `docker login` 会将凭据保存在主目录下的 `.docker/config.json` 中。密码以 Base64 编码形式存储。
>
> 为提升安全性，建议使用
> [Docker 凭据助手](https://github.com/docker/docker-credential-helpers)。
> 如需更强保护，请使用[个人访问令牌](../security/access-tokens.md)
> 替代密码。在 CI/CD 环境或无法使用凭据助手时尤为推荐。

## 重置密码

重置密码步骤：

1. 访问 [Docker 登录页面](https://login.docker.com/)。
1. 输入你的邮箱地址。
1. 在提示输入密码时，选择 **Forgot password?**。

## 故障排查

如果你持有付费的 Docker 订阅，
[联系支持团队](https://hub.docker.com/support/contact/) 获取帮助。

所有 Docker 用户也可以通过以下资源获取问题排查信息与支持（按尽力原则回复）：
   - [Docker 社区论坛](https://forums.docker.com/)
   - [Docker 社区 Slack](http://dockr.ly/comm-slack)
