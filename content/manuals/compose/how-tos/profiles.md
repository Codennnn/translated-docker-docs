---
title: 在 Compose 中使用配置集
linkTitle: 使用服务配置集
weight: 20
description: 如何在 Docker Compose 中使用配置集
keywords: cli, compose, profile, profiles reference
aliases:
- /compose/profiles/
---

{{% include "compose/profiles.md" %}}

## 为服务分配配置集

服务通过 [`profiles` 属性](/reference/compose-file/services.md#profiles) 与配置集关联，
该属性接收一个配置集名称数组：

```yaml
services:
  frontend:
    image: frontend
    profiles: [frontend]

  phpmyadmin:
    image: phpmyadmin
    depends_on: [db]
    profiles: [debug]

  backend:
    image: backend

  db:
    image: mysql
```

上例中，`frontend` 与 `phpmyadmin` 分别被分配到 `frontend` 与 `debug` 配置集，
因此只有在启用相应配置集时才会启动。

未声明 `profiles` 属性的服务始终处于启用状态。上述示例中直接运行 `docker compose up`，
只会启动 `backend` 和 `db`。

有效的配置集名称需匹配正则表达式 `[a-zA-Z0-9][a-zA-Z0-9_.-]+`。

> [!TIP]
>
> 应用的核心服务不应分配 `profiles`，这样它们会始终启用并自动启动。

## 启动指定的配置集

要启用某个特定配置集，可使用 `--profile` [命令行选项](/reference/cli/docker/compose.md)，
或设置 [`COMPOSE_PROFILES` 环境变量](environment-variables/envvars.md#compose_profiles)：

```console
$ docker compose --profile debug up
```
```console
$ COMPOSE_PROFILES=debug docker compose up
```

两种方式都会在启用 `debug` 配置集的情况下启动服务。
在上面的 `compose.yaml` 中，这将启动 `db`、`backend` 和 `phpmyadmin` 服务。

### 启动多个配置集

你也可以同时启用多个配置集。比如运行
`docker compose --profile frontend --profile debug up`，
将同时启用 `frontend` 与 `debug` 配置集。

可通过传入多个 `--profile` 标志，或在 `COMPOSE_PROFILES` 环境变量中使用逗号分隔的列表：

```console
$ docker compose --profile frontend --profile debug up
```

```console
$ COMPOSE_PROFILES=frontend,debug docker compose up
```

如果希望一次性启用所有配置集，可以运行 `docker compose --profile "*"`。

## 自动启动的配置集与依赖解析

当你在命令行显式指定某个已分配配置集的服务时，无需手动启用该配置集；
无论该配置集是否被激活，Compose 都会运行该服务。这对于一次性服务或调试工具非常有用。

仅会启动被显式指定的服务（以及其通过 `depends_on` 声明的依赖）。
同属该配置集的其他服务不会启动，除非：
- 这些服务也被显式指定；或
- 通过 `--profile` 或 `COMPOSE_PROFILES` 明确启用了该配置集。

当一个分配了 `profiles` 的服务被显式指定时，其配置集会自动生效，
因此无需手动启用。这一能力适合一次性服务与调试工具。
例如，考虑如下配置：

```yaml
services:
  backend:
    image: backend

  db:
    image: mysql

  db-migrations:
    image: backend
    command: myapp migrate
    depends_on:
      - db
    profiles:
      - tools
```

```sh
# 仅启动 backend 和 db（未涉及配置集）
$ docker compose up -d

# 在不手动启用 'tools' 配置集的情况下运行 db-migrations 服务
$ docker compose run db-migrations
```

在此示例中，尽管 `db-migrations` 被分配到 `tools` 配置集，但由于它被显式指定，仍会运行。
同时由于在 `depends_on` 中声明，`db` 服务也会自动启动。

如果被指定的服务的依赖也被某个配置集“门控”，需确保这些依赖满足以下任一条件：
- 与其位于同一配置集
- 单独启动
- 未分配任何配置集（始终启用）

## 停止包含特定配置集的应用与服务

与启动指定配置集类似，你可以使用 `--profile` [命令行选项](/reference/cli/docker/compose.md#use--p-to-specify-a-project-name)，
或使用 [`COMPOSE_PROFILES` 环境变量](environment-variables/envvars.md#compose_profiles)：

```console
$ docker compose --profile debug down
```
```console
$ COMPOSE_PROFILES=debug docker compose down
```

两种方式都会停止并移除属于 `debug` 配置集的服务，以及未分配配置集的服务。
在下面的 `compose.yaml` 中，这将停止 `db`、`backend` 与 `phpmyadmin`。

```yaml
services:
  frontend:
    image: frontend
    profiles: [frontend]

  phpmyadmin:
    image: phpmyadmin
    depends_on: [db]
    profiles: [debug]

  backend:
    image: backend

  db:
    image: mysql
```

如果你只想停止 `phpmyadmin` 服务，可以运行 

```console 
$ docker compose down phpmyadmin
``` 
或 
```console 
$ docker compose stop phpmyadmin
```

> [!NOTE]
>
> 直接运行 `docker compose down` 只会停止 `backend` 与 `db`。

## 参考信息

[`profiles`](/reference/compose-file/services.md#profiles)
