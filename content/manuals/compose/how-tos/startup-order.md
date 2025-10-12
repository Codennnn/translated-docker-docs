---
description: 了解如何在 Docker Compose 中使用 depends_on 与 healthcheck 管理服务的启动与停止顺序。
keywords: docker compose startup order, compose shutdown order, depends_on, service healthcheck, control service dependencies
title: 控制 Compose 的启动与停止顺序
linkTitle: 控制启动顺序
weight: 30
aliases:
- /compose/startup-order/
---

你可以通过 [depends_on](/reference/compose-file/services.md#depends_on) 属性来控制服务的启动与停止顺序。Compose 始终按照依赖关系来启动和停止容器；依赖关系由 `depends_on`、`links`、`volumes_from` 与 `network_mode: "service:..."` 决定。

例如，如果你的应用需要访问数据库，并且两个服务都通过 `docker compose up` 启动，那么应用服务可能会先于数据库服务启动，导致其无法连接到可处理 SQL 的数据库，从而引发失败。

## 控制启动

在启动阶段，Compose 只等待容器进入“运行中”状态，并不会等待其完全“就绪”。这在某些场景下会带来问题，例如关系型数据库在可接收连接前需要先完成内部服务的启动。

检测服务“就绪”状态的解决方案是：在 `condition` 属性中使用以下选项之一：

- `service_started`
- `service_healthy`：在启动依赖方服务之前，要求依赖项达到“healthy” 状态（由 `healthcheck` 定义）。
- `service_completed_successfully`：在启动依赖方服务之前，要求依赖项成功运行至完成。

## 示例

```yaml
services:
  web:
    build: .
    depends_on:
      db:
        condition: service_healthy
        restart: true
      redis:
        condition: service_started
  redis:
    image: redis
  db:
    image: postgres
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB}"]
      interval: 10s
      retries: 5
      start_period: 30s
      timeout: 10s
```

Compose 会按依赖顺序创建服务：`db` 与 `redis` 会先于 `web` 创建。

对于标记为 `service_healthy` 的依赖项，Compose 会等待健康检查通过：在创建 `web` 之前，`db` 应当已处于由 `healthcheck` 指示的 “healthy” 状态。

`restart: true` 确保当 `db` 因显式的 Compose 操作（如 `docker compose restart`）而更新或重启时，`web` 服务也会自动重启，从而正确地重新建立连接或依赖。

`db` 服务的健康检查通过 `pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB}` 命令来判断 PostgreSQL 是否就绪。该检查每 10 秒重试一次，最多重试 5 次。

Compose 同样会按依赖顺序移除服务：`web` 会先于 `db` 与 `redis` 被移除。

## 参考信息 

- [`depends_on`](/reference/compose-file/services.md#depends_on)
- [`healthcheck`](/reference/compose-file/services.md#healthcheck)
