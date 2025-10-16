---
title: Docker Home、管理控制台、计费、安全和订阅功能的发布说明
linkTitle: 发布说明
description: 了解 Docker Home、管理控制台以及计费和订阅功能的新功能、错误修复和重大变更
keywords: Docker Home, Docker Admin Console, billing, subscription, security, admin, releases, what's new
weight: 60
params:
  sidebar:
    group: Platform
tags: [Release notes, admin]
---

本页面提供 Docker Home、管理控制台、计费、安全和订阅功能的新功能、增强功能、已知问题和错误修复的详细信息。

## 2025-01-30

### 新功能

- 通过 PKG 安装程序安装 Docker Desktop 现已正式发布。
- 通过配置配置文件强制登录现已正式发布。

## 2024-12-10

### 新功能

- 新的 Docker 订阅现已可用。更多信息，请参阅 [Docker 订阅和功能](/manuals/subscription/details.md) 和 [宣布升级的 Docker 计划：更简单、更有价值、更好的开发和生产力](https://www.docker.com/blog/november-2024-updated-plans-announcement/)。

## 2024-11-18

### 新功能

- 管理员现在可以：
  - 使用[配置配置文件](/manuals/enterprise/security/enforce-sign-in/methods.md#configuration-profiles-method-mac-only)强制登录（早期访问）。
  - 同时为多个组织强制登录（早期访问）。
  - 使用 [PKG 安装程序](/manuals/enterprise/enterprise-deployment/pkg-install-and-configure.md)批量部署 Docker Desktop for Mac（早期访问）。
  - [通过 Docker 管理控制台使用桌面设置管理](/manuals/enterprise/security/hardened-desktop/settings-management/configure-admin-console.md)（早期访问）。

### 错误修复和增强功能

- 增强容器隔离（ECI）已改进为：
  - 允许管理员[关闭 Docker socket 挂载限制](/manuals/enterprise/security/hardened-desktop/enhanced-container-isolation/config.md#allowing-all-containers-to-mount-the-docker-socket)。
  - 在使用 [`allowedDerivedImages` 设置](/manuals/enterprise/security/hardened-desktop/enhanced-container-isolation/config.md#docker-socket-mount-permissions-for-derived-images)时支持通配符标签。

## 2024-11-11

### 新功能

- [个人访问令牌](/security/access-tokens/)（PAT）现在支持过期日期。

## 2024-10-15

### 新功能

- Beta：您现在可以创建[组织访问令牌](/security/for-admins/access-tokens/)（OAT）来增强组织的安全性，并简化 Docker 管理控制台中组织的访问管理。

## 2024-08-29

### 新功能

- 通过 [MSI 安装程序](/manuals/enterprise/enterprise-deployment/msi-install-and-configure.md)部署 Docker Desktop 现已正式发布。
- 两种新的[强制登录](/manuals/enterprise/security/enforce-sign-in/_index.md)方法（Windows 注册表项和 `.plist` 文件）现已正式发布。

## 2024-08-24

### 新功能

- 管理员现在可以查看[组织洞察](/manuals/admin/organization/insights.md)。

## 2024-07-17

### 新功能

- 您现在可以在 [Docker Home](https://app.docker.com) 中集中访问和管理 Docker 产品。