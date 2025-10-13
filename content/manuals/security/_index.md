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
- title: Set up two-factor authentication
  description: Add an extra layer of authentication to your Docker account.
  link: /security/2fa/
  icon: phonelink_lock
- title: Manage access tokens
  description: Create personal access tokens as an alternative to your password.
  icon: password
  link: /security/access-tokens/
- title: Static vulnerability scanning
  description: Automatically run a point-in-time scan on your Docker images for vulnerabilities.
  icon: image_search
  link: /docker-hub/repos/manage/vulnerability-scanning/
- title: Docker Engine security
  description: Understand how to keep Docker Engine secure.
  icon: security
  link: /engine/security/
- title: Secrets in Docker Compose
  description: Learn how to use secrets in Docker Compose.
  icon: privacy_tip
  link: /compose/how-tos/use-secrets/
grid_resources:
- title: Security FAQs
  description: Explore common security FAQs.
  icon: help
  link: /faq/security/general/
- title: Security best practices
  description: Understand the steps you can take to improve the security of your container.
  icon: category
  link: /develop/security-best-practices/
- title: Suppress CVEs with VEX
  description: Learn how to suppress non-applicable or fixed vulnerabilities found in your images.
  icon: query_stats
  link: /scout/guides/vex/
- title: Docker Hardened Images
  description: Learn how to use Docker Hardened Images to enhance your software supply security.
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
