---
description: Compose 预定义环境变量
keywords: fig, composition, compose, docker, orchestration, cli, reference, compose environment configuration, docker env variables
title: 在 Docker Compose 中配置预定义环境变量
linkTitle: 预定义环境变量
weight: 30
aliases:
- /compose/reference/envvars/
- /compose/environment-variables/envvars/
---

Docker Compose 包含若干预定义的环境变量，同时也继承了常见的 Docker CLI 环境变量，例如 `DOCKER_HOST` 与 `DOCKER_CONTEXT`。详见 [Docker CLI 环境变量参考](/reference/cli/docker/#environment-variables)。

本页说明如何设置或更改以下预定义环境变量：

- `COMPOSE_PROJECT_NAME`
- `COMPOSE_FILE`
- `COMPOSE_PROFILES`
- `COMPOSE_CONVERT_WINDOWS_PATHS`
- `COMPOSE_PATH_SEPARATOR`
- `COMPOSE_IGNORE_ORPHANS`
- `COMPOSE_REMOVE_ORPHANS`
- `COMPOSE_PARALLEL_LIMIT`
- `COMPOSE_ANSI`
- `COMPOSE_STATUS_STDOUT`
- `COMPOSE_ENV_FILES`
- `COMPOSE_DISABLE_ENV_FILE`
- `COMPOSE_MENU`
- `COMPOSE_EXPERIMENTAL`
- `COMPOSE_PROGRESS`

## 覆盖方式 

| 方法      | 说明                                  |
| ----------- | -------------------------------------------- |
| [`.env` 文件](/manuals/compose/how-tos/environment-variables/variable-interpolation.md) | 位于工作目录。            |
| [Shell](variable-interpolation.md#substitute-from-the-shell)       | 由宿主操作系统的 Shell 定义。  |
| CLI         | 运行时通过 `--env` 或 `-e` 传入。 |

在更改或设置任何环境变量时，请注意[环境变量优先级](envvars-precedence.md)。

## 配置详情

### 项目与文件配置

#### COMPOSE\_PROJECT\_NAME

设置项目名称。该值会与服务名一起作为容器启动时名称的前缀。

例如，若项目名为 `myapp`，且包含 `db` 与 `web` 两个服务，则 Compose 会分别启动名为 `myapp-db-1` 与 `myapp-web-1` 的容器。

Compose 可以通过多种方式设置项目名。各方式的优先级（从高到低）如下：

1. 命令行标志 `-p`
2. `COMPOSE_PROJECT_NAME`
3. 配置文件中的顶层 `name:` 字段（或通过 `-f` 指定的多份配置文件中最后一个 `name:`）
4. 包含配置文件的项目目录的 `basename`（或通过 `-f` 指定的第一份配置文件所在目录的 `basename`）
5. 若未指定配置文件，则为当前目录的 `basename`

项目名只能包含小写字母、数字、短横线和下划线，并且必须以小写字母或数字开头。如果项目目录或当前目录的 `basename` 不满足此约束，你需要使用上述其他方式之一来设置项目名。

另见[命令行选项概览](/reference/cli/docker/compose/_index.md#command-options-overview-and-help)与[使用 `-p` 指定项目名](/reference/cli/docker/compose/_index.md#use--p-to-specify-a-project-name)。

#### COMPOSE\_FILE

指定 Compose 文件路径。支持指定多份 Compose 文件。

- 默认行为：如果未提供，Compose 会在当前目录查找名为 `compose.yaml` 的文件；若未找到，会在父目录中递归向上查找，直到找到为止。
- 当指定多份 Compose 文件时，路径分隔符默认如下：
   - macOS 与 Linux：`:`（冒号）
   - Windows：`;`（分号）
   例如：

      ```console
      COMPOSE_FILE=compose.yaml:compose.prod.yaml
      ```  
   也可通过 [`COMPOSE_PATH_SEPARATOR`](#compose_path_separator) 自定义路径分隔符。  

另见[命令行选项概览](/reference/cli/docker/compose/_index.md#command-options-overview-and-help)与[使用 `-f` 指定一个或多个 Compose 文件的名称与路径](/reference/cli/docker/compose/_index.md#use--f-to-specify-the-name-and-path-of-one-or-more-compose-files)。

#### COMPOSE\_PROFILES

在执行 `docker compose up` 时，指定要启用的一个或多个配置文件（profile）。

匹配这些 profile 的服务会被启动；未声明 profile 的服务也会被启动。

例如，设置 `COMPOSE_PROFILES=frontend` 并执行 `docker compose up`，会同时选择具有 `frontend` profile 的服务与未指定 profile 的服务。

若指定多个 profile，使用逗号分隔。

以下示例会启用同时匹配 `frontend` 与 `debug` 的服务，以及未指定 profile 的服务： 

```console
COMPOSE_PROFILES=frontend,debug
```

另见[在 Compose 中使用 profile](../profiles.md) 与 [`--profile` 命令行选项](/reference/cli/docker/compose/_index.md#use-profiles-to-enable-optional-services)。

#### COMPOSE\_PATH\_SEPARATOR

为 `COMPOSE_FILE` 中列出的条目指定不同的路径分隔符。

- 默认值：
    - 在 macOS 与 Linux 上为 `:`
    - 在 Windows 上为 `;`

#### COMPOSE\_ENV\_FILES

指定在未使用 `--env-file` 时，Compose 应使用哪些环境文件。

当使用多个环境文件时，使用逗号分隔。例如： 

```console
COMPOSE_ENV_FILES=.env.envfile1,.env.envfile2
```

如果未设置 `COMPOSE_ENV_FILES`，且未在 CLI 中提供 `--env-file`，Docker Compose 会采用默认行为：在项目目录中查找 `.env` 文件。

#### COMPOSE\_DISABLE\_ENV\_FILE

用于禁用默认的 `.env` 文件。 

- 支持的取值： 
    - `true` 或 `1`：Compose 忽略 `.env` 文件
    - `false` 或 `0`：Compose 会在项目目录查找 `.env` 文件
- 默认值：`0`

### 环境处理与容器生命周期

#### COMPOSE\_CONVERT\_WINDOWS\_PATHS

启用后，Compose 会在卷定义中将 Windows 风格路径转换为 Unix 风格路径。

- 支持的取值： 
    - `true` 或 `1`：启用
    - `false` 或 `0`：禁用
- 默认值：`0`

#### COMPOSE\_IGNORE\_ORPHANS

启用后，Compose 不再尝试检测项目中的孤立容器。

- 支持的取值： 
   - `true` 或 `1`：启用
   - `false` 或 `0`：禁用
- 默认值：`0`

#### COMPOSE\_REMOVE\_ORPHANS

启用后，在更新服务或堆栈时，Compose 会自动移除孤立容器。孤立容器是指由先前配置创建、但在当前 `compose.yaml` 中已不再定义的容器。

- 支持的取值：
   - `true` 或 `1`：启用自动移除孤立容器
   - `false` 或 `0`：禁用自动移除，Compose 将仅显示警告
- 默认值：`0`

#### COMPOSE\_PARALLEL\_LIMIT

指定与引擎并发调用的最大并行度。

### 输出 

#### COMPOSE\_ANSI

指定何时打印 ANSI 控制字符。 

- 支持的取值：
   - `auto`：Compose 自动检测是否可使用 TTY 模式，否则使用纯文本模式
   - `never`：使用纯文本模式
   - `always` 或 `0`：使用 TTY 模式
- 默认值：`auto`

#### COMPOSE\_STATUS\_STDOUT

启用后，Compose 会将其内部状态与进度消息写入 `stdout`，而非 `stderr`。
默认值为 `false`，以便明确区分 Compose 消息与容器日志的输出流。

- 支持的取值：
   - `true` 或 `1`：启用
   - `false` 或 `0`：禁用
- 默认值：`0`

#### COMPOSE\_PROGRESS

{{< summary-bar feature_name="Compose progress" >}}

在未使用 `--progress` 时，定义进度输出的类型。 

支持的取值包括 `auto`、`tty`、`plain`、`json` 与 `quiet`。
默认值为 `auto`。 

### 使用体验

#### COMPOSE\_MENU

{{< summary-bar feature_name="Compose menu" >}}

启用后，Compose 会显示一个导航菜单，你可以选择在 Docker Desktop 中打开 Compose 堆栈、开启[`watch` 模式](../file-watch.md)，或使用 [Docker Debug](/reference/cli/docker/debug.md)。

- 支持的取值：
   - `true` 或 `1`：启用
   - `false` 或 `0`：禁用
- 默认值：如果通过 Docker Desktop 获取 Docker Compose，默认为 `1`；否则默认为 `0`

#### COMPOSE\_EXPERIMENTAL

{{< summary-bar feature_name="Compose experimental" >}}

这是一个“可退出（opt-out）”变量。关闭后将禁用实验性功能。

- 支持的取值：
   - `true` 或 `1`：启用
   - `false` 或 `0`：禁用
- 默认值：`1`

## Compose V2 中不再支持

以下环境变量在 Compose V2 中无效。
更多信息见[迁移到 Compose V2](/manuals/compose/releases/migrate.md)。

- `COMPOSE_API_VERSION`
    默认情况下，API 版本与服务器协商确定。请使用 `DOCKER_API_VERSION`。
    参见 [Docker CLI 环境变量参考](/reference/cli/docker/#environment-variables)。
- `COMPOSE_HTTP_TIMEOUT`
- `COMPOSE_TLS_VERSION`
- `COMPOSE_FORCE_WINDOWS_HOST`
- `COMPOSE_INTERACTIVE_NO_CLI`
- `COMPOSE_DOCKER_CLI_BUILD`
    使用 `DOCKER_BUILDKIT` 在 BuildKit 与经典构建器之间进行选择。若设置 `DOCKER_BUILDKIT=0`，则 `docker compose build` 将使用经典构建器来构建镜像。

