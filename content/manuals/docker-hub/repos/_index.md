---
description: 了解如何在 Docker Hub 上管理仓库
keywords: Docker Hub, Hub, 仓库
title: 仓库（Repositories）
weight: 20
aliases:
- /engine/tutorials/dockerrepos/
- /docker-hub/repos/configure/
---

Docker Hub 的“仓库”是容器镜像的集合，你可以在其中公开或私密地存储、管理与分享 Docker 镜像。每个仓库都作为一个独立空间，用于存放与某个应用、微服务或项目相关的镜像。仓库内容通过标签（tag）组织，代表同一应用的不同版本，便于在需要时拉取正确的版本。

本章节将介绍如何：

- [创建](./create.md) 仓库。
- 管理仓库，包括：

   - [仓库信息](./manage/information.md)：添加描述、概述与分类，帮助用户理解仓库的用途与使用方式。完善清晰的信息有助于提升可发现性与可用性。

   - [访问控制](./manage/access.md)：通过灵活的选项控制谁可访问你的仓库。你可以将仓库设为公共或私有，添加协作者；对于组织，还可以通过角色与团队来维护安全与控制。

   - [镜像](./manage/hub-images/_index.md)：仓库支持多种内容类型（包括 OCI 工件），并通过标签实现版本管理。你可以推送新镜像，并在多个仓库间灵活管理现有内容。

   - [镜像安全洞察](./manage/vulnerability-scanning.md)：利用持续的 Docker Scout 分析与静态漏洞扫描，检测、理解并处理容器镜像中的安全问题。

   - [Webhooks](./manage/webhooks.md)：为仓库事件（如镜像推送或更新）配置自动响应，通过触发外部系统的通知或动作来简化工作流。

   - [自动化构建](./manage/builds/_index.md)：与 GitHub 或 Bitbucket 集成实现自动构建。每次代码变更都会触发镜像重建，支持持续集成与持续交付。

   - [可信内容](./manage/trusted-content/_index.md)：参与 Docker 官方镜像，或在“已验证发布者（Verified Publisher）”与“赞助开源（Sponsored Open Source）”项目中管理仓库，包括设置徽标、访问分析数据与启用漏洞扫描等。

- [归档](./archive.md) 已过时或不再支持的仓库。
- [删除](./delete.md) 仓库。
- [管理个人设置](./settings.md)：你可以为自己的账户配置仓库相关的个性化设置，包括默认仓库隐私与自动构建通知。
