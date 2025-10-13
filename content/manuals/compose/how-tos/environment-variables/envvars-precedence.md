---
title: 在 Docker Compose 中的环境变量优先级
linkTitle: 环境变量优先级
description: 通过场景概览说明 Compose 如何解析环境变量的最终取值
keywords: compose, environment, env file
weight: 20
aliases:
- /compose/envvars-precedence/
- /compose/environment-variables/envvars-precedence/
---

当同一个环境变量在多个来源中被设置时，Docker Compose 会遵循一套优先级规则，以确定容器环境中该变量的最终取值。

本文解释当环境变量在多个位置被定义时，Docker Compose 如何确定其最终值。

优先级顺序（从高到低）如下：
1. 通过命令行使用[`docker compose run -e`](set-environment-variables.md#set-environment-variables-with-docker-compose-run---env) 设置。
2. 在 Compose 文件中使用 `environment` 或 `env_file`，但其值来自[Shell](variable-interpolation.md#substitute-from-the-shell) 或环境文件插值（可来自默认的[`.env` 文件](variable-interpolation.md#env-file)，或在 CLI 中使用[`--env-file` 参数](variable-interpolation.md#substitute-with---env-file) 指定）。
3. 仅使用 Compose 文件中的[`environment` 属性](set-environment-variables.md#use-the-environment-attribute) 设置。
4. 使用 Compose 文件中的[`env_file` 属性](set-environment-variables.md#use-the-env_file-attribute)。
5. 在镜像的 [ENV 指令](/reference/dockerfile.md#env) 中设置。
   只有当 Docker Compose 未提供 `environment`、`env_file` 或 `run --env` 时，`Dockerfile` 中的 `ARG` 或 `ENV` 才会生效。

## 简单示例

下面的示例中，同一环境变量在 `.env` 文件与 Compose 文件的 `environment` 属性中被赋予了不同的值：

```console
$ cat ./webapp.env
NODE_ENV=test

$ cat compose.yaml
services:
  webapp:
    image: 'webapp'
    env_file:
     - ./webapp.env
    environment:
     - NODE_ENV=production
```

由 `environment` 属性定义的环境变量具有更高优先级。

```console
$ docker compose run webapp env | grep NODE_ENV
NODE_ENV=production
```

## 进阶示例

下表以 `VALUE` 为例说明，该变量用于定义镜像版本。

### 表格说明

每一列表示可以设置或替换 `VALUE` 的上下文来源。

其中的“Host OS 环境”和“.env 文件”两列仅用于说明：它们本身不会直接在容器中生成变量，只有与 `environment` 或 `env_file` 属性配合使用时才会生效。

每一行表示在若干上下文组合下，`VALUE` 被设置或被替换（或两者兼有）。“结果”列给出了对应场景下 `VALUE` 的最终取值。

|  # |  `docker compose run`  |  `environment` 属性  |  `env_file` 属性  |  镜像 `ENV` |  宿主机环境  |  `.env` 文件      |   结果  |
|:--:|:----------------:|:-------------------------------:|:----------------------:|:------------:|:-----------------------:|:-----------------:|:----------:|
|  1 |   -              |   -                             |   -                    |   -          |  `VALUE=1.4`            |  `VALUE=1.3`      | -               |
|  2 |   -              |   -                             |  `VALUE=1.6`           |  `VALUE=1.5` |  `VALUE=1.4`            |   -               |**`VALUE=1.6`**  |
|  3 |   -              |  `VALUE=1.7`                    |   -                    |  `VALUE=1.5` |  `VALUE=1.4`            |   -               |**`VALUE=1.7`**  |
|  4 |   -              |   -                             |   -                    |  `VALUE=1.5` |  `VALUE=1.4`            |  `VALUE=1.3`      |**`VALUE=1.5`**  |
|  5 |`--env VALUE=1.8` |   -                             |   -                    |  `VALUE=1.5` |  `VALUE=1.4`            |  `VALUE=1.3`      |**`VALUE=1.8`**  |
|  6 |`--env VALUE`     |   -                             |   -                    |  `VALUE=1.5` |  `VALUE=1.4`            |  `VALUE=1.3`      |**`VALUE=1.4`**  |
|  7 |`--env VALUE`     |   -                             |   -                    |  `VALUE=1.5` |   -                     |  `VALUE=1.3`      |**`VALUE=1.3`**  |
|  8 |   -              |   -                             |   `VALUE`              |  `VALUE=1.5` |  `VALUE=1.4`            |  `VALUE=1.3`      |**`VALUE=1.4`**  |
|  9 |   -              |   -                             |   `VALUE`              |  `VALUE=1.5` |   -                     |  `VALUE=1.3`      |**`VALUE=1.3`**  |
| 10 |   -              |  `VALUE`                        |   -                    |  `VALUE=1.5` |  `VALUE=1.4`            |  `VALUE=1.3`      |**`VALUE=1.4`**  |
| 11 |   -              |  `VALUE`                        |   -                    |  `VALUE=1.5` |  -                      |  `VALUE=1.3`      |**`VALUE=1.3`**  |
| 12 |`--env VALUE`     |  `VALUE=1.7`                    |   -                    |  `VALUE=1.5` |  `VALUE=1.4`            |  `VALUE=1.3`      |**`VALUE=1.4`**  |
| 13 |`--env VALUE=1.8` |  `VALUE=1.7`                    |   -                    |  `VALUE=1.5` |  `VALUE=1.4`            |  `VALUE=1.3`      |**`VALUE=1.8`**  |
| 14 |`--env VALUE=1.8` |   -                             |  `VALUE=1.6`           |  `VALUE=1.5` |  `VALUE=1.4`            |  `VALUE=1.3`      |**`VALUE=1.8`**  |
| 15 |`--env VALUE=1.8` |  `VALUE=1.7`                    |  `VALUE=1.6`           |  `VALUE=1.5` |  `VALUE=1.4`            |  `VALUE=1.3`      |**`VALUE=1.8`**  |

### 理解优先级结果

结果 1：本地环境优先，但 Compose 文件未设置在容器内复制该值，因此容器中未设置此变量。

结果 2：Compose 文件中的 `env_file` 为 `VALUE` 定义了明确取值，因此容器环境据此设置。

结果 3：Compose 文件中的 `environment` 为 `VALUE` 定义了明确取值，因此容器环境据此设置。

结果 4：镜像的 `ENV` 指令声明了变量 `VALUE`，且 Compose 文件未覆盖该值，因此由镜像定义生效。

结果 5：`docker compose run` 使用 `--env` 并显式赋值，覆盖了镜像中的取值。

结果 6：`docker compose run` 使用 `--env` 从环境复制值；宿主机中的值优先生效，并复制到容器环境中。

结果 7：`docker compose run` 使用 `--env` 从环境复制值；取用 `.env` 文件中的值来定义容器环境。

结果 8：Compose 文件的 `env_file` 被设置为从本地环境复制 `VALUE`；宿主机值优先生效并复制到容器环境中。

结果 9：Compose 文件的 `env_file` 被设置为从本地环境复制 `VALUE`；取用 `.env` 文件中的值来定义容器环境。

结果 10：Compose 文件的 `environment` 被设置为从本地环境复制 `VALUE`；宿主机值优先生效并复制到容器环境中。

结果 11：Compose 文件的 `environment` 被设置为从本地环境复制 `VALUE`；取用 `.env` 文件中的值来定义容器环境。

结果 12：`--env` 的优先级高于 `environment` 与 `env_file`，并被设置为从本地环境复制 `VALUE`；宿主机值优先生效并复制到容器环境中。

结果 13 至 15：`--env` 的优先级高于 `environment` 与 `env_file`，因此以其值为准。

## 进一步阅读

- [在 Compose 中设置环境变量](set-environment-variables.md)
- [在 Compose 文件中使用变量插值](variable-interpolation.md)
