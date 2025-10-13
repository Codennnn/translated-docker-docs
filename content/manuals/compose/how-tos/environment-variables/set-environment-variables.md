---
title: 在容器环境中设置环境变量
linkTitle: 设置环境变量
weight: 10
description: 使用 Compose 设置、使用与管理环境变量
keywords: compose, orchestration, environment, environment variables, container environment variables
aliases:
- /compose/env/
- /compose/link-env-deprecated/
- /compose/environment-variables/set-environment-variables/
---

只有在服务配置中显式声明后，容器的环境才会被设置。借助 Compose，你可以通过 Compose 文件以两种方式为容器设置环境变量。

>[!TIP]
>
> 不要使用环境变量向容器传递敏感信息（例如密码）。请改用 [secrets](../use-secrets.md)。


## 使用 `environment` 属性 {#use-the-environment-attribute}

你可以在 `compose.yaml` 中使用[`environment` 属性](/reference/compose-file/services.md#environment)直接为容器环境设置环境变量。

它同时支持列表语法与映射语法：

```yaml
services:
  webapp:
    environment:
      DEBUG: "true"
```
等价于 
```yaml
services:
  webapp:
    environment:
      - DEBUG=true
```

更多用法示例，参见[`environment` 属性](/reference/compose-file/services.md#environment)。

### 补充说明 

- 你可以选择不为某变量设置取值，而是将 Shell 中的环境变量原样传递给容器。其行为与 `docker run -e VARIABLE ...` 一致：
  ```yaml
  web:
    environment:
      - DEBUG
  ```
容器内 `DEBUG` 变量的取值来自运行 Compose 的 Shell 环境中的同名变量。注意：若 Shell 环境中未设置该变量，Compose 不会给出警告。

- 你也可以利用[变量插值](variable-interpolation.md#interpolation-syntax)。下例与上例类似，但如果 `DEBUG` 未在 Shell 环境或项目目录的 `.env` 文件中设置，Compose 会给出警告。

  ```yaml
  web:
    environment:
      - DEBUG=${DEBUG}
  ```

## 使用 `env_file` 属性 {#use-the-env_file-attribute}

还可以结合[`env_file` 属性](/reference/compose-file/services.md#env_file)与[`.env` 文件](variable-interpolation.md#env-file)来设置容器环境。

```yaml
services:
  webapp:
    env_file: "webapp.env"
```

使用 `.env` 文件可以让你在普通的 `docker run --env-file ...` 命令中复用同一文件，或在多个服务之间共享同一份 `.env` 文件，而无需在每个服务里重复冗长的 `environment` 配置块。

这也有助于将环境变量与主配置文件分离，提供更有条理且更安全的方式来管理敏感信息；你也无需必须把 `.env` 文件放在项目目录的根目录下。

[`env_file` 属性](/reference/compose-file/services.md#env_file)还支持在 Compose 应用中使用多份 `.env` 文件。  

通过 `env_file` 属性指定的 `.env` 文件路径，是相对于 `compose.yaml` 文件所在位置的相对路径。

> [!IMPORTANT]
>
> 在 `.env` 文件中进行插值是 Docker Compose CLI 的特性。
>
> 当使用 `docker run --env-file ...` 时不受支持。

### 补充说明 

- 如果指定了多份文件，它们会按顺序解析，且后者可以覆盖前者中设置的值。
- 自 Docker Compose 2.24.0 起，你可以通过 `required` 字段将由 `env_file` 指定的 `.env` 文件设为可选。当 `required` 设为 `false` 且该 `.env` 文件缺失时，Compose 会静默忽略该条目。
  ```yaml
  env_file:
    - path: ./default.env
      required: true # default
    - path: ./override.env
      required: false
  ``` 
- 自 Docker Compose 2.30.0 起，你可以通过 `format` 属性为 `env_file` 使用替代的文件格式。更多信息见[`format`](/reference/compose-file/services.md#format)。
- 你可以通过命令行使用[`docker compose run -e`](#set-environment-variables-with-docker-compose-run---env) 来覆盖 `.env` 文件中的取值。

## 使用 `docker compose run --env` 设置环境变量 {#set-environment-variables-with-docker-compose-run---env}

与 `docker run --env` 类似，你可以使用 `docker compose run --env`（或其简写 `docker compose run -e`）临时设置环境变量：

```console
$ docker compose run -e DEBUG=1 web python console.py
```

### 补充说明 

- 你也可以不提供值，直接从 Shell 或环境文件中传递变量：

  ```console
  $ docker compose run -e DEBUG web python console.py
  ```

容器内 `DEBUG` 变量的取值来自运行 Compose 的 Shell 环境中的同名变量或来自环境文件。

## 进一步阅读

- [了解环境变量优先级](envvars-precedence.md)
- [设置或更改预定义环境变量](envvars.md)
- [最佳实践](best-practices.md)
- [了解变量插值](variable-interpolation.md)
