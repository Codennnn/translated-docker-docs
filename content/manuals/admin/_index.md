---
title: 管理功能
description: Docker 管理控制台中的管理功能与角色概述
keywords: 管理, 管理员, 公司, 组织, 管理控制台, 用户账户, 账户管理
weight: 10
params:
  sidebar:
    group: Enterprise
grid:
- title: 公司管理
  description: 探索如何管理公司。
  icon: apartment
  link: /admin/company/
- title: 组织管理
  description: 了解组织管理功能。
  icon: store
  link: /admin/organization/
- title: 组织入驻
  description: 学习如何入驻并保护您的组织。
  icon: explore
  link: /admin/organization/onboard
- title: 公司常见问题
  description: 发现关于公司的常见问题与解答。
  icon: help
  link: /faq/admin/company-faqs/
- title: 组织常见问题
  description: 探索关于组织的热门常见问题。
  icon: help
  link: /faq/admin/organization-faqs/
- title: 安全功能
  description: 探索管理员可用的安全功能。
  icon: shield_locked
  link: /security/
aliases:
- /docker-hub/admin-overview
---

管理员可以使用 [Docker 管理控制台](https://app.docker.com/admin) 管理公司和组织。管理控制台提供集中化的可观测性、访问管理和安全控制，覆盖整个 Docker 环境。

## 公司与组织层次结构

[Docker 管理控制台](https://app.docker.com/admin) 为管理员提供集中化的可观测性、访问管理和针对公司及组织的控制功能。为了实现这些功能，Docker 采用了以下层次结构和角色体系。

![Docker 管理层次结构图，公司位于顶层，其次是组织、团队和成员](./images/docker-admin-structure.webp)

### 公司

公司是多个 Docker 组织的集合，用于集中配置。公司功能仅适用于 Docker Business 订阅用户。

公司提供以下管理员角色：

- **公司所有者**：可以查看和管理公司内的所有组织。拥有公司级设置的完全访问权限，并继承与组织所有者相同的权限。

### 组织

组织包含团队和仓库。所有 Docker Team 和 Business 订阅用户必须至少拥有一个组织。

组织提供以下管理员角色：

- **组织所有者**：可以管理组织设置、用户和访问控制。

### 团队

团队是可选的，允许您将成员分组以集体分配仓库权限。团队简化了跨项目或职能的权限管理。

### 成员

成员是添加到组织中的任何 Docker 用户。组织和公司所有者可以为成员分配角色，以定义其访问级别。

> [!NOTE]
>
> 创建公司是可选的，但组织是 Team 和 Business 订阅的必需项。

## 管理控制台功能

Docker 的[管理控制台](https://app.docker.com/admin) 允许您：

- 创建和管理公司与组织
- 为成员分配角色和权限
- 将成员分组到团队中，按项目或角色管理访问权限
- 设置公司级策略，包括 SCIM 配置和安全强制执行

## 管理公司与组织

在以下章节中学习如何管理公司和组织。

{{< grid >}}
