---
title: 镜像访问管理
description: 使用镜像访问管理控制开发人员可以访问的 Docker Hub 镜像，增强供应链安全性
keywords: 镜像访问管理, docker 官方镜像, 已验证发布者, 供应链安全, docker 商业版
tags: [admin]
aliases:
 - /docker-hub/image-access-management/
 - /desktop/hardened-desktop/image-access-management/
 - /admin/organization/image-access/
 - /security/for-admins/image-access-management/
 - /security/for-admins/hardened-desktop/image-access-management/
weight: 40
---

{{< summary-bar feature_name="Hardened Docker Desktop" >}}

镜像访问管理允许管理员控制开发人员可以从 Docker Hub 拉取的镜像类型。这可以防止开发人员意外使用不可信的社区镜像，从而避免给组织带来安全风险。

通过镜像访问管理，您可以限制以下类型的镜像访问：

- **Docker 官方镜像**：由 Docker 维护的精选镜像
- **Docker 已验证发布者镜像**：来自可信商业发布者的镜像
- **组织镜像**：您组织的私有仓库
- **社区镜像**：来自个人开发者的公共镜像

## 谁应该使用镜像访问管理？

镜像访问管理通过确保开发人员仅使用可信的容器镜像来帮助防止供应链攻击。例如，开发人员在构建新应用程序时可能会意外使用恶意的社区镜像作为组件。镜像访问管理通过限制只能访问已批准的镜像类型来防止这种情况发生。

常见的安全场景包括：

- 防止使用无人维护或恶意的社区镜像
- 确保开发人员仅使用经过审查的官方基础镜像
- 控制对商业第三方镜像的访问
- 在开发团队间保持一致的安全标准

## 先决条件

在配置镜像访问管理之前，您必须：

- [强制登录](/manuals/enterprise/security/enforce-sign-in/_index.md)以确保用户使用您的组织身份进行身份验证
- 使用[个人访问令牌 (PAT)](/manuals/security/access-tokens.md)进行身份验证（不支持组织访问令牌）
- 拥有 Docker Business 订阅

> [!IMPORTANT]
>
> 镜像访问管理仅在用户使用组织凭据登录 Docker Desktop 时生效。

## 配置镜像访问

要配置镜像访问管理：

1. 登录 [Docker Home](https://app.docker.com) 并从左上角的账户下拉菜单中选择您的组织。
1. 选择 **管理控制台**，然后选择 **镜像访问**。
1. 使用**切换开关**启用镜像访问。
1. 选择允许的镜像类型：
    - **组织镜像**：来自您组织的镜像（默认始终允许）。这些可以是组织内成员创建的公共或私有镜像。
    - **社区镜像**：由各种用户贡献的镜像，可能存在安全风险。此类包括 Docker 赞助的开源镜像，默认关闭。
    - **Docker 已验证发布者镜像**：来自已验证发布者计划中 Docker 合作伙伴的镜像，符合安全供应链标准。
    - **Docker 官方镜像**：精选的 Docker 仓库，提供操作系统仓库、Dockerfile 最佳实践、即用型解决方案和及时的安全更新。

应用限制后，组织成员可以以只读格式查看权限页面。

> [!NOTE]
>
> 镜像访问管理默认关闭。无论策略设置如何，组织所有者都可以访问所有镜像。

## 验证访问限制

配置镜像访问管理后，测试限制是否正常工作。

当开发人员拉取允许的镜像类型时：

```console
$ docker pull nginx  # Docker 官方镜像
# 如果允许 Docker 官方镜像，拉取成功
```

当开发人员拉取被阻止的镜像类型时：

```console
$ docker pull someuser/custom-image  # 社区镜像
Error response from daemon: image access denied: community images not allowed
```

镜像访问限制适用于所有 Docker Hub 操作，包括拉取、使用 `FROM` 指令的构建以及 Docker Compose 服务。

## 安全实施

从最严格的策略开始，然后根据合法的业务需求逐步扩展：

1. 开始阶段：仅允许 Docker 官方镜像和组织镜像
2. 如有需要：添加 Docker 已验证发布者镜像用于商业工具
3. 谨慎评估：仅对特定、经过审查的用例允许社区镜像

其他安全建议包括：

- **监控使用模式**：查看开发人员尝试拉取的镜像，识别对其他镜像类型的合法请求，定期审查已批准的镜像类别是否仍然适用，并使用 Docker Desktop 分析功能监控使用模式。
- **分层安全控制**：镜像访问管理与仓库访问管理结合使用效果最佳，以控制开发人员可以访问的仓库；使用增强容器隔离在运行时保护容器安全；使用设置管理控制 Docker Desktop 配置。

## 范围和绕过注意事项

- 镜像访问管理仅控制对 Docker Hub 镜像的访问。来自其他仓库的镜像不受这些策略影响。使用[仓库访问管理](/manuals/enterprise/security/hardened-desktop/registry-access-management.md)控制对其他仓库的访问。
- 用户可能通过以下方式绕过镜像访问管理：退出 Docker Desktop（除非强制登录）、使用不受限制的其他仓库镜像，或使用仓库镜像或代理。强制登录并结合仓库访问管理可实现全面控制。
- 镜像限制适用于 Dockerfile `FROM` 指令，使用受限镜像的 Docker Compose 服务将失败，如果中间镜像受限，多阶段构建可能会受到影响，使用多种镜像类型的 CI/CD 管道可能会受到影响。
