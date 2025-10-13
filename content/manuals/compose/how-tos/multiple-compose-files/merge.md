---
description: Compose 文件如何合并
keywords: compose, docker, merge, compose file
title: 合并 Compose 文件
linkTitle: 合并
weight: 10
aliases:
- /compose/multiple-compose-files/merge/
---

Docker Compose 允许你将一组 Compose 文件进行合并与覆盖，从而生成一个组合后的 Compose 文件。

默认情况下，Compose 会读取两份文件：`compose.yaml` 与可选的 `compose.override.yaml`。
约定上，`compose.yaml` 存放基础配置；覆盖文件可以对现有服务进行配置覆盖，或新增全新的服务。

如果某个服务在两份文件中都有定义，Compose 会依据下文与
[Compose 规范](/reference/compose-file/merge.md)中描述的规则来合并配置。

## 如何合并多个 Compose 文件

若要使用多份覆盖文件，或使用不同名称的覆盖文件，你可以使用预定义的
[COMPOSE_FILE](../environment-variables/envvars.md#compose_file) 环境变量，或通过 `-f` 选项指定文件列表。

Compose 会按照命令行中指定的顺序依次合并文件。后出现的文件可以合并、覆盖或追加前面文件中的内容。

例如：

```console
$ docker compose -f compose.yaml -f compose.admin.yaml run backup_db
```

`compose.yaml` 可能定义了一个 `webapp` 服务：

```yaml
webapp:
  image: examples/web
  ports:
    - "8000:8000"
  volumes:
    - "/data"
```

`compose.admin.yaml` 也为同一个服务提供了配置：

```yaml
webapp:
  environment:
    - DEBUG=1
```

相同字段会覆盖上一份文件中的值；新增字段会追加到 `webapp` 的配置中：

```yaml
webapp:
  image: examples/web
  ports:
    - "8000:8000"
  volumes:
    - "/data"
  environment:
    - DEBUG=1
```

## 合并规则 

- 路径以基础文件为基准进行解析。使用多份 Compose 文件时，必须确保所有路径都相对于基础 Compose 文件（即通过 `-f` 首先指定的那份）。
  之所以要求这样，是因为覆盖文件未必是完整合法的 Compose 文件；它们可以只包含少量配置片段。
  追踪服务的每个片段到底应相对哪个路径既困难又易混淆。为便于理解，所有路径都必须基于基础文件来定义。

   >[!TIP]
   >
   > 你可以使用 `docker compose config` 来审查合并后的配置，避免路径相关问题。

- Compose 会将原始服务的配置复制到本地服务上。
  如果某个配置项同时在原始服务与本地服务中定义，则本地值会替换或扩展原始值。

   - 对于 `image`、`command`、`mem_limit` 这类单值选项，新值会替换旧值：

      原始服务：

      ```yaml
      services:
        myservice:
          # ...
          command: python app.py
      ```

      本地服务：

      ```yaml
      services:
        myservice:
          # ...
          command: python otherapp.py
      ```

      结果：

      ```yaml
      services:
        myservice:
          # ...
          command: python otherapp.py
      ```

   - 对于 `ports`、`expose`、`external_links`、`dns`、`dns_search`、`tmpfs` 等多值选项，Compose 会将两组值连接起来：

      原始服务：

      ```yaml
      services:
        myservice:
          # ...
          expose:
            - "3000"
      ```

      本地服务：

      ```yaml
      services:
        myservice:
          # ...
          expose:
            - "4000"
            - "5000"
      ```

      结果：

      ```yaml
      services:
        myservice:
          # ...
          expose:
            - "3000"
            - "4000"
            - "5000"
      ```

   - 针对 `environment`、`labels`、`volumes`、`devices`，Compose 会“合并”各项条目，并以本地定义为优先。
     对于 `environment` 与 `labels`，由变量名或标签名决定使用哪个值：

      原始服务：

      ```yaml
      services:
        myservice:
          # ...
          environment:
            - FOO=original
            - BAR=original
      ```

      本地服务：

      ```yaml
      services:
        myservice:
          # ...
          environment:
            - BAR=local
            - BAZ=local
      ```

      结果：

      ```yaml
      services:
        myservice:
          # ...
          environment:
            - FOO=original
            - BAR=local
            - BAZ=local
      ```

   - `volumes` 与 `devices` 条目会按容器内的挂载路径进行合并：

      原始服务：

      ```yaml
      services:
        myservice:
          # ...
          volumes:
            - ./original:/foo
            - ./original:/bar
      ```

      本地服务：

      ```yaml
      services:
        myservice:
          # ...
          volumes:
            - ./local:/bar
            - ./local:/baz
      ```

      结果：

      ```yaml
      services:
        myservice:
          # ...
          volumes:
            - ./original:/foo
            - ./local:/bar
            - ./local:/baz
      ```

更多合并规则见 Compose 规范的[合并与覆盖](/reference/compose-file/merge.md)。 

### 更多信息

- 使用 `-f` 是可选的。如果未提供，Compose 会在工作目录及其父目录中查找 `compose.yaml` 与 `compose.override.yaml`。至少需要提供 `compose.yaml`。
  如果两者在同一目录层级同时存在，Compose 会将它们合并为单一配置。

- 你可以将 `-f` 的文件名设为 `-`（短横线）以从 `stdin` 读取配置。例如： 
   ```console
   $ docker compose -f - <<EOF
     webapp:
       image: examples/web
       ports:
        - "8000:8000"
       volumes:
        - "/data"
       environment:
        - DEBUG=1
     EOF
   ```
   
   当使用 `stdin` 时，配置中的所有路径都相对于当前工作目录。
   
- 你可以使用 `-f` 指定不在当前目录中的 Compose 文件路径；也可在 shell 或环境文件中设置
  [COMPOSE_FILE 环境变量](../environment-variables/envvars.md#compose_file) 来实现。

   例如，若你在运行
   [Compose Rails 示例](https://github.com/docker/awesome-compose/tree/master/official-documentation-samples/rails/README.md)，并且在 `sandbox/rails` 目录下有 `compose.yaml`。
   你可以在任意位置通过如下方式使用 `-f`，配合
   [docker compose pull](/reference/cli/docker/compose/pull.md) 获取 `db` 服务所需的 postgres 镜像：
   `docker compose -f ~/sandbox/rails/compose.yaml pull db`

   完整示例如下：

   ```console
   $ docker compose -f ~/sandbox/rails/compose.yaml pull db
   Pulling db (postgres:latest)...
   latest: Pulling from library/postgres
   ef0380f84d05: Pull complete
   50cf91dc1db8: Pull complete
   d3add4cd115c: Pull complete
   467830d8a616: Pull complete
   089b9db7dc57: Pull complete
   6fba0a36935c: Pull complete
   81ef0e73c953: Pull complete
   338a6c4894dc: Pull complete
   15853f32f67c: Pull complete
   044c83d92898: Pull complete
   17301519f133: Pull complete
   dcca70822752: Pull complete
   cecf11b8ccf3: Pull complete
   Digest: sha256:1364924c753d5ff7e2260cd34dc4ba05ebd40ee8193391220be0f9901d4e1651
   Status: Downloaded newer image for postgres:latest
   ```

## 示例

一个常见场景是：基于多份文件，将开发环境的 Compose 应用调整为接近生产的环境（例如生产、预发布或 CI）。
为支持这些差异，你可以将 Compose 配置拆分为多份文件：

首先使用一份基础文件，定义各服务的标准配置。

`compose.yaml`

```yaml
services:
  web:
    image: example/my_web_app:latest
    depends_on:
      - db
      - cache

  db:
    image: postgres:latest

  cache:
    image: redis:latest
```

在这个示例中，开发环境配置会向宿主机暴露一些端口、将代码以卷方式挂载，并构建 web 镜像。

`compose.override.yaml`

```yaml
services:
  web:
    build: .
    volumes:
      - '.:/code'
    ports:
      - 8883:80
    environment:
      DEBUG: 'true'

  db:
    command: '-d'
    ports:
     - 5432:5432

  cache:
    ports:
      - 6379:6379
```

当你运行 `docker compose up` 时，会自动读取覆盖配置。

若要在生产环境中使用该 Compose 应用，你可以再创建一份覆盖文件。它可能存放在不同的 Git 仓库中，或由其他团队管理。

`compose.prod.yaml`

```yaml
services:
  web:
    ports:
      - 80:80
    environment:
      PRODUCTION: 'true'

  cache:
    environment:
      TTL: '500'
```

要使用这份生产配置进行部署，可以运行：

```console
$ docker compose -f compose.yaml -f compose.prod.yaml up -d
```

该命令会基于 `compose.yaml` 与 `compose.prod.yaml` 的配置部署全部三个服务，
而不会使用 `compose.override.yaml` 中的开发配置。

更多信息，参见[在生产环境中使用 Compose](../production.md)。 

## 限制

Docker Compose 支持在应用模型中为多种资源使用相对路径：服务镜像的构建上下文、环境变量文件的位置、绑定挂载卷所使用的本地目录路径等。
受此约束影响，在单一仓库（monorepo）中组织代码会变得困难——直觉上的做法是为每个团队或组件使用独立文件夹，但这会让 Compose 文件中的相对路径失去意义。

## 参考信息

- [合并规则](/reference/compose-file/merge.md)
