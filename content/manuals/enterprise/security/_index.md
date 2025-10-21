---
linkTitle: 安全性
title: 企业级安全性
description: 了解 Docker 提供的企业级安全功能并探索最佳实践
keywords: docker, docker hub, docker desktop, 安全, 企业, 扩展
weight: 10
params:
  sidebar:
    group: Enterprise
grid_admins:
- title: 设置管理
  description: 了解设置管理如何保护您开发者的工作流程。
  icon: shield_locked
  link: /enterprise/security/hardened-desktop/settings-management/
- title: 增强容器隔离
  description: 了解增强容器隔离如何防止容器攻击。
  icon: security
  link: /enterprise/security/hardened-desktop/enhanced-container-isolation/
- title: 仓库访问管理
  description: 控制开发者在使用 Docker Desktop 时可以访问的仓库。
  icon: home_storage
  link: /enterprise/security/hardened-desktop/registry-access-management/
- title: 镜像访问管理
  description: 控制开发者可以从 Docker Hub 拉取的镜像。
  icon: photo_library
  link: /enterprise/security/hardened-desktop/image-access-management/
- title: "隔离容器"
  description: 限制容器访问不需要的网络资源。
  icon: "vpn_lock"
  link: /enterprise/security/hardened-desktop/air-gapped-containers/
- title: 强制登录
  description: 为您的团队和组织成员配置登录。
  link: /enterprise/security/enforce-sign-in/
  icon: passkey
- title: 域管理
  description: 识别组织中未捕获的用户。
  link: /enterprise/security/domain-management/
  icon: person_search
- title: Docker Scout
  description: 探索 Docker Scout 如何帮助您创建更安全的软件供应链。
  icon: query_stats
  link: /scout/
- title: SSO
  description: 了解如何为您的公司或组织配置 SSO。
  icon: key
  link: /enterprise/security/single-sign-on/
- title: SCIM
  description: 设置 SCIM 以自动配置和取消配置用户。
  icon: checklist
  link: /enterprise/security/provisioning/scim/
- title: 角色和权限
  description: 为个人分配角色，赋予他们在组织内的不同权限。
  icon: badge
  link: /enterprise/security/roles-and-permissions/
- title: 扩展私有市场 (Beta)
  description: 了解如何为您的 Docker Desktop 用户配置和设置包含精选扩展列表的私有市场。
  icon: storefront
  link: /desktop/extensions/private-marketplace/
- title: 组织访问令牌
  description: 创建组织访问令牌作为密码的替代方案。
  link: /enterprise/security/access-tokens/
  icon: password
---

Docker 为管理员和开发者提供了全面的安全防护措施。

如果您是管理员，可以为您的开发者强制执行 Docker 产品的登录，并通过增强容器隔离和仓库访问管理等 DevOps 安全控制来扩展、管理和保护您的 Docker Desktop 实例。

## 管理员

探索 Docker 提供的安全功能，以满足您公司的安全策略。

{{< grid items="grid_admins" >}}
