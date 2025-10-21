---
title: 强化版 Docker Desktop
linkTitle: 强化版 Docker Desktop
description: 帮助组织在不影响生产力的情况下保护开发者环境的安全功能
keywords: 安全, 强化版桌面, 增强容器隔离, 注册表访问管理, 设置管理, 管理员, docker desktop, 镜像访问管理, 空隙容器
tags: [admin]
aliases:
 - /desktop/hardened-desktop/
 - /security/for-admins/hardened-desktop/
grid:
  - title: "设置管理"
    description: 了解设置管理如何保护您开发者的工作流程。
    icon: shield_locked
    link: /enterprise/security/hardened-desktop/settings-management/
  - title: "增强容器隔离"
    description: 了解增强容器隔离如何防止容器攻击。
    icon: "security"
    link: /enterprise/security/hardened-desktop/enhanced-container-isolation/
  - title: "注册表访问管理"
    description: 控制开发者在使用 Docker Desktop 时可以访问的注册表。
    icon: "home_storage"
    link: /enterprise/security/hardened-desktop/registry-access-management/
  - title: "镜像访问管理"
    description: 控制开发者可以从 Docker Hub 拉取的镜像。
    icon: "photo_library"
    link: /enterprise/security/hardened-desktop/image-access-management/
  - title: "空隙容器"
    description: 限制容器访问不需要的网络资源。
    icon: "vpn_lock"
    link: /enterprise/security/hardened-desktop/air-gapped-containers/
weight: 60
---

{{< summary-bar feature_name="Hardened Docker Desktop" >}}

强化版 Docker Desktop 提供了一系列安全功能，旨在加强开发者环境，同时不影响生产力或开发者体验。

通过强化版 Docker Desktop，您可以实施严格的安全策略，防止开发者和容器绕过组织控制。您还可以增强容器隔离，以防范可能突破 Docker Desktop Linux 虚拟机或底层主机系统的安全威胁，如恶意负载。

## 谁应该使用强化版 Docker Desktop？

强化版 Docker Desktop 非常适合注重安全的组织，这些组织：

- 不向开发者的机器提供 root 或管理员访问权限
- 希望对 Docker Desktop 配置进行集中控制
- 必须满足特定的合规性要求

## 强化版 Docker Desktop 的工作原理

强化版 Docker Desktop 功能可以独立工作，也可以协同工作，创建深度防御安全策略。它们在多个层面保护开发者工作站免受攻击，包括 Docker Desktop 配置、容器镜像管理和容器运行时安全：

- 注册表访问管理和镜像访问管理可防止访问未经授权的容器注册表和镜像类型，减少暴露于恶意负载的风险
- 增强容器隔离在 Linux 用户命名空间内以非 root 权限运行容器，限制恶意容器的影响
- 空隙容器允许您为容器配置网络限制，防止恶意容器访问组织的内部网络资源
- 设置管理锁定 Docker Desktop 配置，以强制执行公司策略，防止开发者（无论是有意还是无意）引入不安全的设置

## 后续步骤

探索强化版 Docker Desktop 功能，了解它们如何加强您组织的安全态势：

{{< grid >}}
