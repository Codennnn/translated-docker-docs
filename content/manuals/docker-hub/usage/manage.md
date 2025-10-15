---
description: 了解如何优化与管理 Docker Hub 的使用。
keywords: Docker Hub, 限制, 使用
title: 优化 Docker Hub 使用的最佳实践
linkTitle: 优化使用
weight: 40
---

无论个人还是组织，可通过以下步骤优化与管理 Docker Hub 的使用：

1. [查看你的 Docker Hub 使用情况](https://hub.docker.com/usage)。

2. 利用使用数据识别哪些账户消耗最多的数据、高峰使用时段，以及哪些镜像与最高的数据用量相关。同时关注以下使用趋势：

   - 低效的拉取行为：识别访问频繁的仓库，评估是否可以通过优化缓存或整合使用，降低拉取次数。
   - 低效的自动化系统：检查哪些自动化工具（如 CI/CD 流水线）导致较高的拉取速率，并进行配置，避免不必要的镜像拉取。

3. 通过以下方式优化镜像拉取：

   - 使用缓存：通过[镜像中转（mirroring）](/docker-hub/mirror/)或在 CI/CD 流水线中使用本地镜像缓存，减少重复拉取。
   - 自动化手动流程：配置自动化系统，使其仅在有镜像新版本时才执行拉取，避免不必要的拉取。

4. 通过以下方式优化存储：

    - 定期审计并[删除整个仓库](../repos/delete.md)中未打标签、未使用或过时的镜像。
    - 使用[镜像管理](../repos/manage/hub-images/manage.md)清理仓库内的陈旧或过时镜像。

5. 针对组织，可通过以下方式进行监控并落实组织级策略：

   - 定期[查看 Docker Hub 使用情况](https://hub.docker.com/usage)以持续监控。
   - [强制登录](/security/for-admins/enforce-sign-in/)，确保可跟踪用户使用情况，并让用户享受更高的使用额度。
   - 排查 Docker 中的重复用户账户，并按需将其从组织移除。