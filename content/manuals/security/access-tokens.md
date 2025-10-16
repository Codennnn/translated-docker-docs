---
title: 个人访问令牌（PAT）
linkTitle: 个人访问令牌
description: 创建并管理 Docker 个人访问令牌，用于安全的 CLI 认证与自动化场景
keywords: personal access tokens, PAT, docker cli authentication, docker hub security, programmatic access, 个人访问令牌, 访问令牌
weight: 10
aliases:
 - /docker-hub/access-tokens/
 - /security/for-developers/access-tokens/
---

个人访问令牌（Personal Access Tokens，简称 PAT）是 Docker CLI 认证中比密码更安全的替代方案。使用 PAT 可以让自动化系统、CI/CD 流水线与开发工具完成身份认证，而无需暴露你的 Docker Hub 密码。

## 关键优势

相较使用密码，PAT 在安全性与可控性方面更具优势：

- 更高的安全性：可审计令牌使用行为，及时禁用可疑令牌；即便系统遭入侵，也能降低对账户产生管理员级影响的风险。
- 更好的自动化：可为不同的集成场景签发多枚令牌，并设置不同权限；在不再需要时可独立吊销。
- 兼容双重验证：当你启用双重验证（2FA）时，PAT 是 CLI 访问的必需方式，既能保证安全，又不会绕过 2FA 保护。
- 使用可追踪：可监控令牌的使用时间与方式，便于识别潜在的安全问题或清理不再使用的自动化。

## 适用人群与场景

以下常见场景建议使用 PAT：

- 本地开发：在本地开发流程中让 Docker CLI 完成认证
- CI/CD 流水线：在持续集成系统中自动构建与部署镜像
- 自动化脚本：在自动部署或备份脚本中推送/拉取镜像
- 开发工具：在 IDE、容器管理工具或监控系统中集成 Docker Hub 访问能力
- 双重验证：启用 2FA 时，使用 CLI 必须使用 PAT

> [!NOTE]
>
> 若需面向组织范围的自动化，请考虑使用[组织访问令牌](/manuals/enterprise/security/access-tokens.md)。组织令牌不与个人用户账户绑定。

## 创建个人访问令牌

> [!IMPORTANT]
>
> 将访问令牌视同密码，妥善保管。请使用凭据管理器保存令牌，切勿将令牌提交到任何源码仓库。

创建个人访问令牌：

1. 登录 [Docker Home](https://app.docker.com/)。
1. 点击右上角头像，在下拉菜单中选择 **Account settings**。
1. 选择 **Personal access tokens**。
1. 选择 **Generate new token**。
1. 配置令牌参数：
   - **Description：** 使用能清晰表达用途的名称
   - **Expiration date：** 按你的安全策略设置到期时间
   - **Access permissions：** 选择 **Read**、**Write** 或 **Delete**
1. 选择 **Generate**。复制屏幕上显示的令牌并妥善保存。一旦关闭页面，你将无法再次查看该令牌。

## 使用个人访问令牌

使用 PAT 登录 Docker CLI：

```console
$ docker login --username <YOUR_USERNAME>
Password: [paste your PAT here]
```

当命令提示输入密码时，请输入 PAT，而不是 Docker Hub 密码。

## 修改个人访问令牌

> [!NOTE]
>
> 已创建的个人访问令牌无法修改到期时间；若需调整到期时间，请新建一枚 PAT。

你可以根据需要重命名、启用、停用或删除令牌。这些操作可在账户设置中完成。

1. 登录 [Docker Home](https://app.docker.com/login)。
1. 点击右上角头像，在下拉菜单中选择 **Account settings**。
1. 选择 **Personal access tokens**。
      - 此页面会展示所有令牌的总览，并标注每个令牌是手动创建还是[自动生成](#auto-generated-tokens)；你还可以查看令牌的作用域、启用/停用状态、创建时间、最近使用时间与到期日期。
1. 在某一令牌行最右侧打开操作菜单，可选择 **Deactivate**/**Activate**、**Edit** 或 **Delete** 来修改该令牌。
1. 编辑完成后，选择 **Save token** 保存变更。

## 自动生成的令牌 {#auto-generated-tokens}

当你登录 Docker Desktop 时，系统会自动为你创建认证令牌，具备以下特性：

- 自动创建：登录 Docker Desktop 时自动生成
- 完整权限：包含 Read、Write 与 Delete 权限
- 会话关联：当 Docker Desktop 会话过期后自动移除
- 账户上限：每个账户最多保留 5 枚自动生成的令牌
- 自动清理：新令牌生成时会自动清理更早的令牌

如有需要，你可以手动删除自动生成的令牌；但在使用 Docker Desktop 时它们会再次被创建。

## 合理使用政策（Fair Use）

在使用个人访问令牌时，请注意：过度创建令牌可能导致限流或额外费用。为保证资源公平分配与服务质量，Docker 保留对过度使用 PAT 的账户施加限制的权利。

合理使用的最佳实践包括：

- 在相似用例中复用令牌，避免为每个细分用途创建过多独立令牌
- 定期删除不再使用的令牌
- 面向组织范围的自动化场景，使用[组织访问令牌](/manuals/enterprise/security/access-tokens.md)
- 持续监控令牌使用情况，发现并优化不必要的消耗
