---
title: 在 Compose 文件中通过插值设置、使用与管理变量
linkTitle: 变量插值
description: 通过插值在 Compose 文件中设置、使用与管理变量
keywords: compose, orchestration, environment, variables, interpolation
weight: 40
aliases:
- /compose/env-file/
- /compose/environment-variables/env-file/
- /compose/environment-variables/variable-interpolation/
---

Compose 文件可以使用变量以获得更高的灵活性。比如你想在多个镜像标签之间快速切换以测试不同版本，或需要将某个卷的源路径调整为本地环境，每次都无需修改 Compose 文件；你可以定义变量，在运行时将其插值到 Compose 文件中。

插值还可在运行时把值插入 Compose 文件中，用于向容器环境传递变量。

下面是一个简单示例： 

```console
$ cat .env
TAG=v1.5
$ cat compose.yaml
services:
  web:
    image: "webapp:${TAG}"
```

When you run `docker compose up`, the `web` service defined in the Compose file [interpolates](variable-interpolation.md) in the image `webapp:v1.5` which was set in the `.env` file. You can verify this with the
[config command](/reference/cli/docker/compose/config.md), which prints your resolved application config to the terminal:

```console
$ docker compose config
services:
  web:
    image: 'webapp:v1.5'
```

## 插值语法 {#interpolation-syntax}

插值适用于未加引号与使用双引号的值。
同时支持带花括号（`${VAR}`）与不带花括号（`$VAR`）的表达式。

对于带花括号的表达式，支持以下格式：
- 直接替换
  - `${VAR}` -> `VAR` 的值
- 默认值
  - `${VAR:-default}` -> 若 `VAR` 已设置且非空，则取其值，否则取 `default`
  - `${VAR-default}` -> 若 `VAR` 已设置，则取其值，否则取 `default`
- 必需值
  - `${VAR:?error}` -> 若 `VAR` 已设置且非空，则取其值，否则报错退出
  - `${VAR?error}` -> 若 `VAR` 已设置，则取其值，否则报错退出
- 备选值
  - `${VAR:+replacement}` -> 若 `VAR` 已设置且非空，则取 `replacement`，否则为空
  - `${VAR+replacement}` -> 若 `VAR` 已设置，则取 `replacement`，否则为空

更多信息见 Compose 规范中的[插值](/reference/compose-file/interpolation.md)。 

## 通过插值设置变量的方式

Docker Compose 可以从多个来源将变量插值进 Compose 文件。

注意：当同一变量由多个来源声明时，会应用优先级：

1. 来自 Shell 环境的变量
2. 若未设置 `--env-file`，取本地工作目录（`PWD`）中的 `.env` 文件定义的变量
3. 由 `--env-file` 指定的文件，或项目目录中的 `.env` 文件定义的变量

你可以运行 `docker compose config --environment` 查看 Compose 在插值 Compose 模型时使用的变量及其取值。

### `.env` 文件 {#env-file}

在 Docker Compose 中，`.env` 文件是一个文本文件，用于定义在运行 `docker compose up` 时可用于插值的变量。该文件通常包含变量的键值对，便于集中管理配置；当你需要存储多个变量时非常有用。

`.env` 文件是设置变量的默认方式。`.env` 文件应放置于项目目录根部、与 `compose.yaml` 文件同级。关于环境文件的格式，参见[环境文件语法](#env-file-syntax)。

基础示例： 

```console
$ cat .env
## define COMPOSE_DEBUG based on DEV_MODE, defaults to false
COMPOSE_DEBUG=${DEV_MODE:-false}

$ cat compose.yaml 
  services:
    webapp:
      image: my-webapp-image
      environment:
        - DEBUG=${COMPOSE_DEBUG}

$ DEV_MODE=true docker compose config
services:
  webapp:
    environment:
      DEBUG: "true"
```

#### 补充说明 

- 如果在 `.env` 文件中定义了变量，你可以在 `compose.yaml` 中通过[`environment` 属性](/reference/compose-file/services.md#environment)直接引用。例如，当 `.env` 文件包含环境变量 `DEBUG=1`，且 `compose.yaml` 如下：
   ```yaml
    services:
      webapp:
        image: my-webapp-image
        environment:
          - DEBUG=${DEBUG}
   ```
   Docker Compose 会将 `${DEBUG}` 替换为 `.env` 文件中的值。

   > [!IMPORTANT]
   >
   > 当 `.env` 文件中的变量被用于容器环境时，请注意[环境变量优先级](envvars-precedence.md)。

- 你可以将 `.env` 文件放在项目根目录之外的位置，并通过[CLI 的 `--env-file` 选项](#substitute-with---env-file)让 Compose 定位到它。

- 如果通过[`--env-file` 进行替换](#substitute-with---env-file)，你的 `.env` 文件可被其他 `.env` 覆盖。

> [!IMPORTANT]
>
> 从 `.env` 文件进行替换是 Docker Compose CLI 的特性。
>
> 在使用 `docker stack deploy` 的 Swarm 中不受支持。

#### `.env` 文件语法 {#env-file-syntax}

环境文件适用以下语法规则：

- 以 `#` 开头的行被视为注释并忽略。
- 空行会被忽略。
- 未加引号与使用双引号（`"`）的值会应用插值。
- 每一行代表一对键值。值可选择性加引号。
  - `VAR=VAL` -> `VAL`
  - `VAR="VAL"` -> `VAL`
  - `VAR='VAL'` -> `VAL`
- 未加引号值的行内注释前必须有空格。
  - `VAR=VAL # comment` -> `VAL`
  - `VAR=VAL# not a comment` -> `VAL# not a comment`
- 对于加引号的值，行内注释必须放在结束引号之后。
  - `VAR="VAL # not a comment"` -> `VAL # not a comment`
  - `VAR="VAL" # comment` -> `VAL`
- 单引号（`'`）包裹的值按字面量处理。
  - `VAR='$OTHER'` -> `$OTHER`
  - `VAR='${OTHER}'` -> `${OTHER}`
- 引号可使用 `\` 转义。
  - `VAR='Let\'s go!'` -> `Let's go!`
  - `VAR="{\"hello\": \"json\"}"` -> `{"hello": "json"}`
- 双引号值支持常见 Shell 转义序列，包括 `\n`、`\r`、`\t` 与 `\\`。
  - `VAR="some\tvalue"` -> `some  value`
  - `VAR='some\tvalue'` -> `some\tvalue`
  - `VAR=some\tvalue` -> `some\tvalue`
- 单引号值可跨多行。例如：

   ```yaml
   KEY='SOME
   VALUE'
   ```

   随后运行 `docker compose config`，你会看到：
  
   ```yaml
   environment:
     KEY: |-
       SOME
       VALUE
   ```

### 使用 `--env-file` 进行替换 {#substitute-with---env-file}

你可以在一个 `.env` 文件中为多个环境变量设置默认值，然后在 CLI 中把该文件作为参数传入。

这种方式的优势是：你可以将该文件存储在任意位置并自定义命名。该文件路径是相对于执行 Docker Compose 命令时的当前工作目录。通过 `--env-file` 选项传入文件路径：

```console
$ docker compose --env-file ./config/.env.dev up
```

#### 补充说明 

- 当你想临时覆盖 `compose.yaml` 已引用的 `.env` 文件时，此方法非常有用。例如，你可能为生产（`.env.prod`）与测试（`.env.test`）准备了不同的 `.env` 文件。
  在下面的示例中，存在两个环境文件：`.env` 与 `.env.dev`，它们对 `TAG` 设置了不同的值。
  ```console
  $ cat .env
  TAG=v1.5
  $ cat ./config/.env.dev
  TAG=v1.6
  $ cat compose.yaml
  services:
    web:
      image: "webapp:${TAG}"
  ```
  若命令行未使用 `--env-file`，将默认加载 `.env` 文件：
  ```console
  $ docker compose config
  services:
    web:
      image: 'webapp:v1.5'
  ```
  传入 `--env-file` 参数会覆盖默认文件路径：
  ```console
  $ docker compose --env-file ./config/.env.dev config
  services:
    web:
      image: 'webapp:v1.6'
  ```
  当 `--env-file` 指定了无效的文件路径时，Compose 会返回错误：
  ```console
  $ docker compose --env-file ./doesnotexist/.env.dev  config
  ERROR: Couldn't find env file: /home/user/./doesnotexist/.env.dev
  ```
- 你可以使用多个 `--env-file` 选项来指定多份环境文件，Docker Compose 会按顺序读取。后面的文件可以覆盖前面文件中的变量。
  ```console
  $ docker compose --env-file .env --env-file .env.override up
  ```
- 在启动容器时，你可以通过命令行覆盖特定环境变量。 
  ```console
  $ docker compose --env-file .env.dev up -e DATABASE_URL=mysql://new_user:new_password@new_db:3306/new_database
  ```

### 本地 `.env` 文件与 <项目目录> `.env` 文件的关系

`.env` 文件也可用于声明用于控制 Compose 行为与加载文件的[预定义环境变量](envvars.md)。

当未显式提供 `--env-file` 标志时，Compose 会在你的工作目录（[PWD](https://www.gnu.org/software/bash/manual/html_node/Bash-Variables.html#index-PWD)）中搜索 `.env` 文件，并加载其值用于自配置与插值。如果此文件中的取值定义了预定义变量 `COMPOSE_FILE`，从而将项目目录设置到另一个文件夹，则 Compose 还会（若存在）加载第二个 `.env` 文件。第二个 `.env` 文件的优先级更低。

借助该机制，你可以通过一组自定义变量来调用现有 Compose 项目作为覆盖，而无需通过命令行逐个传入环境变量。

### 从 Shell 进行替换 {#substitute-from-the-shell}

你可以使用宿主机已有的环境变量，或执行 `docker compose` 命令时所在 Shell 环境中的变量。这允许你在运行时将取值动态注入到 Docker Compose 配置中。

例如，假设 Shell 中包含 `POSTGRES_VERSION=9.3`，并提供如下配置：

```yaml
db:
  image: "postgres:${POSTGRES_VERSION}"
```

当你以该配置运行 `docker compose up` 时，Compose 会在 Shell 中查找 `POSTGRES_VERSION` 环境变量并进行替换。就本例而言，Compose 会在运行配置前将镜像解析为 `postgres:9.3`。

如果某环境变量未设置，Compose 会用空字符串进行替换。在上例中，若未设置 `POSTGRES_VERSION`，镜像选项的取值将为 `postgres:`。

> [!NOTE]
>
> `postgres:` 不是一个合法的镜像引用。Docker 期望使用不带标签的引用（如 `postgres`，默认使用最新镜像）或带标签的引用（如 `postgres:15`）。


