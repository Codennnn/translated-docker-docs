---
title: 使用手册
description: 通过这一系列用户指南，学习如何安装、配置和使用 Docker 产品
keywords: docker, 文档, 手册, 产品, 用户指南, 操作指南
# hard-code the URL of this page
url: /manuals/
layout: wide
params:
  icon: description
  sidebar:
    groups:
      - Open source
      - AI
      - Products
      - Platform
      - Enterprise
  notoc: true
  open-source:
  - title: Docker Build
    description: 随时随地构建和交付任何应用。
    icon: build
    link: /build/
  - title: Docker Engine
    description: 业界领先的容器运行时引擎。
    icon: developer_board
    link: /engine/
  - title: Docker Compose
    description: 定义并运行多容器应用。
    icon: /icons/Compose.svg
    link: /compose/
  - title: Testcontainers
    description: 使用你熟悉的编程语言以编程方式运行容器。
    icon: /icons/Testcontainers.svg
    link: /testcontainers/
  - title: Cagent
    description: 开源多智能体解决方案，协助你完成各类任务。
    icon: /icons/cagent.svg
    link: /ai/cagent
  ai:
  - title: Ask Gordon
    description: 通过你的个人 AI 助手简化工作流程，充分发挥 Docker 生态系统的优势。
    icon: note_add
    link: /ai/gordon/
  - title: Docker Model Runner
    description: 查看和管理你的本地模型。
    icon: /icons/models.svg
    link: /ai/model-runner/
  - title: MCP Catalog and Toolkit
    description: 使用 MCP 服务器增强你的 AI 工作流。
    icon: /icons/toolkit.svg
    link: /ai/mcp-catalog-and-toolkit/
  products:
  - title: Docker Desktop
    description: 容器开发的指挥中心。
    icon: /icons/Whale.svg
    link: /desktop/
  - title: Docker Hardened Images
    description: 安全、精简的镜像，确保可信的软件交付。
    icon: /icons/dhi.svg
    link: /dhi/
  - title: Docker Offload
    description: 在云端构建和运行容器。
    icon: cloud
    link: /offload/
  - title: Build Cloud
    description: 在云端更快速地构建镜像。
    icon: /icons/logo-build-cloud.svg
    link: /build-cloud/
  - title: Docker Hub
    description: 发现、共享和集成容器镜像。
    icon: hub
    link: /docker-hub/
  - title: Docker Scout
    description: 镜像分析与策略评估。
    icon: /icons/Scout.svg
    link: /scout/
  - title: Docker for GitHub Copilot
    description: 将 Docker 功能与 GitHub Copilot 集成。
    icon: chat
    link: /copilot/
  - title: Docker Extensions
    description: 自定义你的 Docker Desktop 工作流。
    icon: extension
    link: /extensions/
  - title: Testcontainers Cloud
    description: 在云端运行集成测试，使用真实依赖。
    icon: package_2
    link: https://testcontainers.com/cloud/docs/
  platform:
  - title: 管理中心
    description: 为企业和组织提供集中化的可观测性。
    icon: admin_panel_settings
    link: /admin/
  - title: 账单管理
    description: 管理账单和支付方式。
    icon: payments
    link: /billing/
  - title: 账户管理
    description: 管理你的 Docker 账户。
    icon: account_circle
    link: /accounts/
  - title: 安全
    description: 为管理员和开发者提供安全防护。
    icon: lock
    link: /security/
  - title: 订阅
    description: Docker 产品的商业使用许可。
    icon: card_membership
    link: /subscription/
  enterprise:
  - title: 部署 Docker Desktop
    description: 在企业内大规模部署 Docker Desktop
    icon: download
    link: /enterprise/enterprise-deployment/
---

本节包含关于如何安装、配置和使用 Docker 产品的用户指南。

## 开源项目

开源开发与容器化技术。

{{< grid items=open-source >}}

## AI 工具

所有 Docker AI 工具，一站式访问。

{{< grid items=ai >}}

## 产品

为创新团队打造的端到端开发解决方案。

{{< grid items=products >}}

## 平台

与 Docker 平台相关的文档，包括管理中心和订阅管理。

{{< grid items=platform >}}

## 企业版

面向 IT 管理员，提供大规模部署 Docker Desktop 的帮助，以及安全相关功能的配置指导。

{{< grid items=enterprise >}}
