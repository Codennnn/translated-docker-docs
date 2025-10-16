---
title: Docker 订阅与功能
linkTitle: 订阅与功能
description: 了解 Docker 各订阅层级及其关键功能
keywords: 订阅, Personal, Pro, Team, Business, 功能, docker 订阅
aliases:
- /subscription/core-subscription/details/
weight: 10
---

Docker 订阅为 Docker 产品的商业使用提供授权，并可访问 Docker 完整的开发平台：

- [Docker Desktop](../desktop/_index.md)：业界领先的“以容器为先”的开发方案，包含 Docker Engine、Docker CLI、Docker Compose、
  Docker Build/BuildKit 与 Kubernetes。
- [Docker Hub](../docker-hub/_index.md)：全球最大的云端容器仓库。
- [Docker Build Cloud](../build-cloud/_index.md)：强大的云端构建器，将构建速度最多提升至 39 倍。
- [Docker Scout](../scout/_index.md)：用于软件供应链安全的工具，可快速评估镜像健康状况并加速安全改进。
- [Testcontainers Cloud](https://testcontainers.com/cloud/docs)：基于容器的测试自动化，提供更快的测试、统一的开发者体验等。

从个人开发者到大型企业，选择适合你需求的订阅。

> [!NOTE]
>
> 旧版 Docker 计划适用于最后一次购买或续订发生在 2024 年 12 月 10 日之前的订阅者。这些订阅者将在 2024 年 12 月 10 日或之后的下一次续订之前，继续保留其当前订阅与定价。

## 订阅

{{< tabs >}}
{{< tab name="Docker 订阅" >}}

## Docker Personal

**Docker Personal** 适合开源社区、个人开发者、教育场景和小型企业。

Docker Personal 包含：

- 核心 Docker 工具，免费使用
- 1 个启用漏洞分析的 Docker Scout 仓库
- 无限数量的 Docker Hub 公共仓库
- 认证用户每 6 小时 200 次拉取配额
- Docker Build Cloud 与 Testcontainers Cloud 的 7 天试用

试用结束后，如需继续使用 Docker Build Cloud 或 Testcontainers Cloud，可随时升级至 Docker Pro。

所有未认证用户（包括未认证的 Docker Personal 用户），按 IPv4 地址或 IPv6 /64 子网计，每 6 小时享有 100 次拉取配额。

各层级可用功能清单，参见 [Docker 定价](https://www.docker.com/pricing/)。

## Docker Pro

**Docker Pro** 适合需要完整访问 Docker 开发平台的个人开发者。

Docker Pro 包含：

- 完整访问所有 Docker 工具
- 每月 200 分钟 Docker Build Cloud 构建时长（不结转）
- 2 个启用漏洞分析的 Docker Scout 仓库
- 每月 100 分钟 Testcontainers Cloud 运行时长（不结转）
- Docker Hub 拉取不限速

各层级可用功能清单，参见 [Docker 定价](https://www.docker.com/pricing/)。

## Docker Team

**Docker Team** 适合需要协作与安全能力的开发团队。

Docker Team 包含：

- 每月 500 分钟 Docker Build Cloud 构建时长（不结转）
- 不限数量的启用漏洞分析的 Docker Scout 仓库
- 每月 500 分钟 Testcontainers Cloud 运行时长（不结转）
- Docker Hub 拉取不限速
- 高级协作工具：组织管理、[基于角色的访问控制（RBAC）](/security/for-admins/roles-and-permissions/)、[活动日志](/admin/organization/activity-logs/) 等

各层级可用功能清单，参见 [Docker 定价](https://www.docker.com/pricing/)。

## Docker Business

**Docker Business** 适合需要集中化管理与高级安全能力的企业。

Docker Business 包含：

- 每月 1500 分钟 Docker Build Cloud 构建时长（不结转）
- 不限数量的启用漏洞分析的 Docker Scout 仓库
- 每月 1500 分钟 Testcontainers Cloud 运行时长（不结转）
- Docker Hub 拉取不限速
- 企业级安全功能：
  - [强化版 Docker Desktop](/manuals/enterprise/security/hardened-desktop/_index.md)
  - [镜像访问管理](/manuals/enterprise/security/hardened-desktop/image-access-management.md)
  允许管理员控制开发者可访问的内容
  - [仓库访问管理](/manuals/enterprise/security/hardened-desktop/registry-access-management.md)
  允许管理员控制开发者可访问的仓库
  - [公司层级](/admin/company/) 用于管理多个组织与设置
  - [单点登录](/security/for-admins/single-sign-on/)
  - [跨域身份管理系统（SCIM）](/security/for-admins/provisioning/scim/)

各层级可用功能清单，参见 [Docker 定价](https://www.docker.com/pricing/)。

{{< /tab >}}
{{< tab name="旧版 Docker 计划" >}}

> [!IMPORTANT]
>
> 旧版 Docker 计划适用于最后一次购买或续订发生在 2024 年 12 月 10 日之前的订阅者。这些订阅者将在 2024 年 12 月 10 日或之后的下一次续订之前，继续保留其当前订阅与定价。

如果你持有旧版订阅，续费时会自动升级到新的 Docker 订阅模型。新计划可访问全部 Docker 工具，并提供更高配额与附加功能。

## 旧版 Docker Pro

**Legacy Docker Pro** 让个人开发者更好地掌控其开发环境，提供稳定一致的开发者体验，减少在琐碎重复任务上的时间投入，从而将更多时间用于为客户创造价值。

Legacy Docker Pro 包含：
- Unlimited public repositories
- Unlimited [Scoped Access Tokens](/security/access-tokens/)
- Unlimited [collaborators](/docker-hub/repos/manage/access/#collaborators) for public repositories at no cost per month.
- Access to [Legacy Docker Scout Free](#legacy-docker-scout-free) to get started with software supply chain security.
- Unlimited private repositories
- 5000 image [pulls per day](/manuals/docker-hub/usage/pulls.md)
- [Auto Builds](/docker-hub/builds/) with 5 concurrent builds
- 300 [Vulnerability Scans](/docker-hub/vulnerability-scanning/)

各旧版层级可用功能清单，参见 [Legacy Docker 定价](https://www.docker.com/legacy-pricing/)。

### 升级你的旧版 Docker Pro 订阅

当你将 Legacy Docker Pro 升级到 Docker Pro 后，将获得以下变化：

- Docker Build Cloud build minutes increased from 100/month to 200/month and no monthly fee. Docker Build Cloud minutes do not rollover month to month.
- 2 included repositories with continuous vulnerability analysis in Docker Scout.
- 100 Testcontainers Cloud runtime minutes are now included for use either in Docker Desktop or for CI. Testcontainers Cloud runtime minutes do not rollover month to month.
- Docker Hub image pull rate limits are removed.

各层级可用功能清单，参见 [Docker 定价](https://www.docker.com/pricing/)。

## 旧版 Docker Team

**Legacy Docker Team** 为组织提供协作、效率与安全能力，使团队能够在必要的安全与团队管理功能的加持下，充分释放协作与共享的价值。Docker Team 订阅包含 Docker Desktop 与 Docker Hub 等组件的商业使用许可。

Legacy Docker Team 包含：
- Everything included in legacy Docker Pro
- Unlimited teams
- [Auto Builds](/docker-hub/builds/) with 15 concurrent builds
- Unlimited [Vulnerability Scanning](/docker-hub/vulnerability-scanning/)
- 5000 image [pulls per day](/manuals/docker-hub/usage/pulls.md) for each team member

还提供高级协作与管理工具，包括基于 [角色的访问控制（RBAC）](/security/for-admins/roles-and-permissions/) 的组织与团队管理、[活动日志](/admin/organization/activity-logs/)等。

各旧版层级可用功能清单，参见 [Legacy Docker 定价](https://www.docker.com/legacy-pricing/)。

### 升级你的旧版 Docker Team 订阅

当你将 Legacy Docker Team 升级到 Docker Team 后，将获得以下变化：

- Instead of paying an additional per-seat fee, Docker Build Cloud is now available to all users in your Docker subscription.
- Docker Build Cloud build minutes increase from 400/mo to 500/mo. Docker Build Cloud minutes do not rollover month to month.
- Docker Scout now includes unlimited repositories with continuous vulnerability analysis, an increase from 3.
- 500 Testcontainers Cloud runtime minutes are now included for use either in Docker Desktop or for CI. Testcontainers Cloud runtime minutes do not rollover month to month.
- Docker Hub image pull rate limits are removed.
- The minimum number of users is 1 (lowered from 5).

各层级可用功能清单，参见 [Docker 定价](https://www.docker.com/pricing/)。

## 旧版 Docker Business

**Legacy Docker Business** 为大规模使用 Docker 的企业提供集中化管理与高级安全能力，帮助管理者治理其 Docker 开发生态，并加速安全供应链建设。Docker Business 订阅包含 Docker Desktop 与 Docker Hub 等组件的商业使用许可。

Legacy Docker Business 包含：
- Everything included in legacy Docker Team
- [Hardened Docker Desktop](/manuals/enterprise/security/hardened-desktop/_index.md)
- [Image Access Management](/manuals/enterprise/security/hardened-desktop/image-access-management.md) which lets admins control what content developers can access
- [Registry Access Management](/manuals/enterprise/security/hardened-desktop/registry-access-management.md) which lets admins control what registries developers can access
- [Company layer](/admin/company/) to manage multiple organizations and settings
- [Single Sign-On](/security/for-admins/single-sign-on/)
- [System for Cross-domain Identity Management](/security/for-admins/provisioning/scim/) and more.

各旧版层级可用功能清单，参见 [Legacy Docker 定价](https://www.docker.com/legacy-pricing/)。

### 升级你的旧版 Docker Business 订阅

当你将 Legacy Docker Business 升级到 Docker Business 后，将获得以下变化：

- Instead of paying an additional per-seat fee, Docker Build Cloud is now available to all users in your Docker subscription.
- Docker Build Cloud included minutes increase from 800/mo to 1500/mo. Docker Build Cloud minutes do not rollover month to month.
- Docker Scout now includes unlimited repositories with continuous vulnerability analysis, an increase from 3.
- 1500 Testcontainers Cloud runtime minutes are now included for use either in Docker Desktop or for CI. Testcontainers Cloud runtime minutes do not rollover month to month.
- Docker Hub image pull rate limits are removed.

各层级可用功能清单，参见 [Docker 定价](https://www.docker.com/pricing/)。

## 旧版 Docker Scout 订阅

本节概述 Docker Scout 的旧版订阅。

> [!IMPORTANT]
>
> 自 2024 年 12 月 10 日起，Docker Scout 订阅不再单独提供，已由可访问全部工具的新 Docker 订阅所取代。若你在 2024 年 12 月 10 日之前订阅或续费，旧版 Docker 订阅仍将持续生效，直至下一次续费。详情参见《[Announcing Upgraded Docker Plans](https://www.docker.com/blog/november-2024-updated-plans-announcement/)》。

### 旧版 Docker Scout Free

Legacy Docker Scout Free 面向组织提供。若你拥有 Legacy Docker 订阅，将自动获得对该版本的访问权限。

Legacy Docker Scout Free 包含：

- 本地镜像分析不限量
- 最多 3 个启用 Docker Scout 的仓库
- SDLC 集成（含策略评估与工作负载集成）
- 本地与云端容器仓库集成
- 安全态势报告

### 旧版 Docker Scout Team

Legacy Docker Scout Team 包含：

- 旧版 Docker Scout Free 的全部功能
- 在 3 个启用 Docker Scout 的仓库之外，购买订阅后可额外添加最多 100 个仓库

### 旧版 Docker Scout Business

Legacy Docker Scout Business 包含：

- 旧版 Docker Scout Team 的全部功能
- 启用 Docker Scout 的仓库数量不限

### 升级你的旧版 Docker Scout 订阅

当你将 Legacy Docker Scout 升级到 Docker 订阅后，将获得以下变化：

- Docker Business: Unlimited repositories with continuous vulnerability analysis, an increase from 3.
- Docker Team: Unlimited repositories with continuous vulnerability analysis, an increase from 3
- Docker Pro: 2 included repositories with continuous vulnerability analysis.
- Docker Personal: 1 included repository with continuous vulnerability analysis.

各层级可用功能清单，参见 [Docker 定价](https://www.docker.com/pricing/)。

## 旧版 Docker Build Cloud 订阅

 本节介绍不同旧版 Docker Build Cloud 订阅层级的功能。

> [!IMPORTANT]
>
> 自 2024 年 12 月 10 日起，Docker Build Cloud 仅随新的 Docker Pro、Team 与 Business 计划提供。自该日期起续费后，你每月包含的 Build Cloud 分钟数将增加。详情参见《[Announcing Upgraded Docker Plans](https://www.docker.com/blog/november-2024-updated-plans-announcement/)》。

### 旧版 Docker Build Cloud Starter

如果你拥有 Legacy Docker 订阅，将包含一定基础的 Build Cloud 分钟数与缓存。具体功能随订阅层级而异。

#### 旧版 Docker Pro

- 100 build minutes every month
- Available for one user
- 4 parallel builds

#### 旧版 Docker Team

- 400 build minutes every month shared across your organization
- Option to onboard up to 100 members
- Can buy additional seats to add more minutes

#### 旧版 Docker Business

- All the features listed for Docker Team
- 800 build minutes every month shared across your organization

### 旧版 Docker Build Cloud Team

Legacy Docker Build Cloud Team 提供：

- 200 additional build minutes per seat
- Option to buy reserve minutes
- Increased shared cache

Legacy Docker Build Cloud Team 订阅与 Docker [组织](/admin/organization/) 绑定。要使用该订阅的构建分钟数或共享缓存，用户必须属于与订阅关联的组织。参见“管理席位与邀请”。

### 旧版 Docker Build Cloud Enterprise

如需了解企业订阅的详细信息，请[联系销售](https://www.docker.com/products/build-cloud/#contact_sales)。

### 升级你的旧版 Docker Build Cloud 订阅

无需再单独订阅 Docker Build Cloud 才能使用或扩展分钟数。当你将 Legacy Docker 订阅升级到 Docker 订阅后，将获得以下变化：

- Docker Business: Included minutes are increased from 800/mo to 1500/mo with the option to scale more minutes.
- Docker Team: Included minutes are increased from 400/mo to 500/mo with the option to scale more minutes.
- Docker Pro: Included minutes are increased from 100/mo to 200/mo with the option to scale more minutes.
- Docker Personal: You receive a 7-day trial.

{{< /tab >}}
{{< /tabs >}}

## 订阅管理方式

### 自助服务

你可自行管理所有事项，包括发票、席位、账单信息与订阅变更。

### 销售协助

Docker 指定的客户经理将为 Docker Business 与 Team 订阅提供设置与管理支持。

## 支持

所有 Docker Pro、Team 与 Business 订阅者均可获得订阅相关的邮件支持。