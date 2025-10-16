---
title: 面向开发者的安全
linkTitle: 安全
description: 了解开发者层面的安全功能，例如双重认证（2FA）与访问令牌
keywords: docker, docker hub, docker desktop, 安全, 开发者安全, 2FA, 访问令牌
weight: 40
params:
  sidebar:
    group: Platform
grid_developers:
- title: 启用双重验证
  description: 为你的 Docker 账户增加一层身份验证。
  link: /security/2fa/
  icon: phonelink_lock
- title: 管理访问令牌
  description: 创建个人访问令牌，作为密码的替代方案。
  icon: password
  link: /security/access-tokens/
- title: 静态漏洞扫描
  description: 对你的 Docker 镜像执行一次性自动漏洞扫描。
  icon: image_search
  link: /docker-hub/repos/manage/vulnerability-scanning/
- title: Docker Engine 安全
  description: 了解如何保障 Docker Engine 的安全。
  icon: security
  link: /engine/security/
- title: 在 Docker Compose 中使用机密
  description: 了解如何在 Docker Compose 中使用机密（secrets）。
  icon: privacy_tip
  link: /compose/how-tos/use-secrets/
grid_resources:
- title: 安全常见问题
  description: 浏览常见的安全相关问题。
  icon: help
  link: /faq/security/general/
- title: 安全最佳实践
  description: 了解提升容器安全性的措施。
  icon: category
  link: /develop/security-best-practices/
- title: 使用 VEX 抑制 CVE
  description: 了解如何抑制镜像中不适用或已修复的漏洞。
  icon: query_stats
  link: /scout/guides/vex/
- title: Docker 加固镜像
  description: 了解如何使用 Docker Hardened Images 增强软件供应链安全。
  icon: encrypted_add_circle
  link: /dhi/
---

Docker 通过面向开发者的安全功能，帮助你保护本地环境、基础设施与网络。

你可以使用双重认证（2FA）、个人访问令牌（PAT）与 Docker Scout 等工具，在工作流早期进行访问控制与漏洞发现。你还可以通过 Docker Compose 在开发栈中安全集成机密（secrets），或使用 Docker Hardened Images 提升软件供应链安全性。

浏览以下章节了解更多信息。

## 面向开发者

{{< grid items="grid_developers" >}}

## 更多资源

{{< grid items="grid_resources" >}}
