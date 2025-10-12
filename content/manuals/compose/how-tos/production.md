---
description: 了解如何为生产环境配置、部署与更新 Docker Compose 应用。
keywords: compose, orchestration, containers, production, production docker compose configuration
title: 在生产环境中使用 Compose
weight: 100
aliases:
- /compose/production/
---

在开发环境中使用 Compose 定义应用后，你可以复用这份定义，将应用运行在不同环境中，例如 CI、预发布（staging）与生产环境。

部署应用的最简单方式是将其运行在单台服务器上，这与开发环境的运行方式类似。若需要扩展应用规模，可以在 Swarm 集群上运行 Compose 应用。

### 为生产环境修改 Compose 文件

为适配生产环境，你可能需要调整应用配置，常见改动包括：

- 移除应用代码的卷绑定，使代码保留在容器内，避免被外部修改
- 在宿主机上绑定不同的端口
- 以不同方式设置环境变量，例如降低日志详细程度，或为外部服务（如邮件服务器）指定配置
- 指定重启策略（如 [`restart: always`](/reference/compose-file/services.md#restart)）以降低停机风险
- 增加辅助服务，如日志聚合器

基于上述原因，建议新增一份面向生产环境的 Compose 文件，例如 `compose.production.yaml`，用于记录生产特有的配置。该文件只需包含相对原始 Compose 文件的差异配置。运行时再将该文件叠加到原始的 `compose.yaml` 上，以生成最终的配置。

准备好第二份配置文件后，可通过 `-f` 选项使用它：

```console
$ docker compose -f compose.yaml -f compose.production.yaml up -d
```

更多完整示例与可选方式，参见：[使用多个 Compose 文件](multiple-compose-files/_index.md)。

### 部署变更

当你修改了应用代码，请记得重建镜像并重新创建应用的容器。要重新部署名为 `web` 的服务，可执行：

```console
$ docker compose build web
$ docker compose up --no-deps -d web
```

第一条命令会为 `web` 重建镜像；第二条命令会停止、销毁并仅重建 `web` 服务本身。`--no-deps` 标志可避免同时重建 `web` 依赖的其他服务。

### 在单台服务器上运行 Compose

你可以通过设置 `DOCKER_HOST`、`DOCKER_TLS_VERIFY` 与 `DOCKER_CERT_PATH` 环境变量，将应用部署到远程 Docker 主机。更多信息参见[预定义环境变量](environment-variables/envvars.md)。

一旦配置好这些环境变量，常规的 `docker compose` 命令即可直接使用，无需额外配置。

## 进一步阅读

- [使用多个 Compose 文件](multiple-compose-files/_index.md)

