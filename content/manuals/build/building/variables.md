---
title: 构建变量
linkTitle: 变量
weight: 20
description: 使用构建参数与环境变量来配置构建
keywords: build, args, variables, parameters, env, environment variables, config
aliases:
- /build/buildkit/color-output-controls/
- /build/building/env-vars/
- /build/guide/build-args/
---

在 Docker 构建流程中，构建参数（`ARG`）与环境变量（`ENV`）都用于向构建过程传递信息。
你可以用它们对构建进行参数化，从而让构建更加灵活、可配置。

> [!WARNING]
>
> 不要使用构建参数或环境变量向构建传递机密信息（secrets），它们可能会暴露在最终镜像中。
> 建议使用 Secret 挂载或 SSH 挂载，以更安全的方式在构建期间提供机密。
>
> 参见：[构建机密](./secrets.md)。

## 相同点与差异

构建参数与环境变量有不少相似之处：
- 它们都在 Dockerfile 中声明，并可通过 `docker build` 的标志进行设置；
- 都能用于让构建具备参数化能力。
但它们的用途并不相同。

### 构建参数（Build arguments）

构建参数是服务于 Dockerfile 本身的变量。
你可以用它来参数化 Dockerfile 指令中的取值。
例如，可用构建参数来指定要安装的依赖版本。

构建参数只有在被指令使用时才会对构建产生影响。
由镜像创建的容器中默认无法访问这些参数，除非你明确地把它们从 Dockerfile 传递到镜像的文件系统或配置信息中。
构建参数可能会保留在镜像元数据中（例如来源证明和镜像历史），因此不适合存放机密信息。

使用构建参数能让 Dockerfile 更加灵活、也更易维护。

关于如何使用构建参数，参见：[``ARG`` 使用示例](#arg-usage-example)。

### 环境变量（Environment variables）

环境变量会传入构建执行环境，并会在由该镜像创建的容器中持续存在。

环境变量主要用于：

- 配置构建的执行环境
- 为容器设置默认环境变量

被设置后的环境变量可以直接影响你的构建执行过程，以及应用程序的行为或配置。

你不能在构建命令行直接覆盖或设置环境变量的值。
环境变量的值必须在 Dockerfile 中声明。
你可以将环境变量与构建参数结合，使环境变量的值能够在构建时进行配置。

关于如何使用环境变量来配置构建，参见：[``ENV`` 使用示例](#env-usage-example)。

## `ARG` 使用示例

构建参数通常用于指定构建中所用组件的版本，例如镜像变体或软件包版本。

将版本作为构建参数声明，可以在不手动修改 Dockerfile 的情况下使用不同版本进行构建。
这同样让 Dockerfile 更易维护，因为你可以在文件顶部集中声明版本。

构建参数也能帮助你在多处复用同一个值。
例如，如果构建中用了多种 `alpine` 变体，你可以确保各处使用相同的 `alpine` 版本：

- `golang:1.22-alpine${ALPINE_VERSION}`
- `python:3.12-alpine${ALPINE_VERSION}`
- `nginx:1-alpine${ALPINE_VERSION}`

下面的示例通过构建参数定义了 `node` 与 `alpine` 的版本：

```dockerfile
# syntax=docker/dockerfile:1

ARG NODE_VERSION="{{% param example_node_version %}}"
ARG ALPINE_VERSION="{{% param example_alpine_version %}}"

FROM node:${NODE_VERSION}-alpine${ALPINE_VERSION} AS base
WORKDIR /src

FROM base AS build
COPY package*.json ./
RUN npm ci
RUN npm run build

FROM base AS production
COPY package*.json ./
RUN npm ci --omit=dev && npm cache clean --force
COPY --from=build /src/dist/ .
CMD ["node", "app.js"]
```

在这个示例中，构建参数带有默认值。
你可以选择在构建时是否进行覆盖。
若需覆盖默认值，可使用 `--build-arg`：

```console
$ docker build --build-arg NODE_VERSION=current .
```

关于构建参数的更多信息，参见：

- [`ARG` Dockerfile reference](/reference/dockerfile.md#arg)
- [`docker build --build-arg` reference](/reference/cli/docker/buildx/build.md#build-arg)

## `ENV` 使用示例

使用 `ENV` 声明环境变量后，该变量会对构建阶段中后续的所有指令可用。
下面的示例在使用 `npm` 安装依赖前，将 `NODE_ENV` 设置为 `production`。
设置该变量后，`npm` 会忽略仅用于本地开发的依赖包。

```dockerfile
# syntax=docker/dockerfile:1

FROM node:20
WORKDIR /app
COPY package*.json ./
ENV NODE_ENV=production
RUN npm ci && npm cache clean --force
COPY . .
CMD ["node", "app.js"]
```

默认情况下，环境变量不能在构建时直接配置。
如果想在构建时改变某个 `ENV` 的值，可以结合环境变量与构建参数来实现：

```dockerfile
# syntax=docker/dockerfile:1

FROM node:20
ARG NODE_ENV=production
ENV NODE_ENV=$NODE_ENV
WORKDIR /app
COPY package*.json ./
RUN npm ci && npm cache clean --force
COPY . .
CMD ["node", "app.js"]
```

配合上述 Dockerfile，你可以用 `--build-arg` 覆盖 `NODE_ENV` 的默认值：

```console
$ docker build --build-arg NODE_ENV=development .
```

请注意，环境变量会在容器运行时继续生效，因此它们可能会对应用的运行产生意料之外的影响。

关于在构建中使用环境变量的更多信息，参见：

- [`ENV` Dockerfile reference](/reference/dockerfile.md#env)

## 作用域（Scoping）

在 Dockerfile 的全局作用域中声明的构建参数，并不会自动继承到各个构建阶段。
它们只在全局作用域内可用。

```dockerfile
# syntax=docker/dockerfile:1

# 下述构建参数在全局作用域中声明：
ARG NAME="joe"

FROM alpine
# 下面这条指令无法访问 $NAME 构建参数，
# 因为该参数只在全局作用域声明，而非当前阶段。
RUN echo "hello ${NAME}!"
```

在上面的示例中，由于 `NAME` 构建参数超出了作用域，`echo` 的输出会是 `hello !`。
若想在某个阶段中使用全局构建参数，必须在该阶段显示声明并“消费”它：

```dockerfile
# syntax=docker/dockerfile:1

# 在全局作用域声明构建参数
ARG NAME="joe"

FROM alpine
# 在构建阶段中“消费”该构建参数
ARG NAME
RUN echo $NAME
```

一旦在某个阶段中声明或消费了构建参数，它会被该阶段的子阶段自动继承。

```dockerfile
# syntax=docker/dockerfile:1
FROM alpine AS base
# 在构建阶段中声明构建参数
ARG NAME="joe"

# 基于 "base" 创建一个新的阶段
FROM base AS build
# 由于在父阶段中声明了 NAME 构建参数，
# 因此在此阶段可直接使用
RUN echo "hello $NAME!"
```

下图进一步示意了在多阶段构建中，构建参数与环境变量的继承关系：

{{< figure src="../../images/build-variables.svg" class="invertible" >}}

## 预定义构建参数

本节介绍默认在所有构建中可用的预定义构建参数。

### 多平台构建参数

多平台构建参数用于描述构建平台与目标平台。

构建平台指运行构建器（BuildKit 守护进程）的宿主系统的操作系统、架构及平台变体。

- `BUILDPLATFORM`
- `BUILDOS`
- `BUILDARCH`
- `BUILDVARIANT`

目标平台参数表示该次构建的目标平台，其值由 `docker build` 命令的 `--platform` 标志指定。

- `TARGETPLATFORM`
- `TARGETOS`
- `TARGETARCH`
- `TARGETVARIANT`

这些参数通常用于多平台构建中的交叉编译。
它们在 Dockerfile 的全局作用域中可用，但不会自动被各阶段继承。
若要在阶段内使用，必须在该阶段中显式声明：

```dockerfile
# syntax=docker/dockerfile:1

# 预定义构建参数可在全局作用域中使用
FROM --platform=$BUILDPLATFORM golang
# 如需在阶段内使用，需通过 ARG 声明以便继承
ARG TARGETOS
RUN GOOS=$TARGETOS go build -o ./exe .
```

关于多平台构建参数的更多信息，参见：[多平台参数](/reference/dockerfile.md#automatic-platform-args-in-the-global-scope)

### 代理参数

代理相关的构建参数用于为构建指定代理服务器。
你无需在 Dockerfile 中声明或引用这些参数。
在构建时通过 `--build-arg` 指定即可生效。

默认情况下，代理参数不会写入到构建缓存以及 `docker history` 的输出中。
如果你在 Dockerfile 中引用了这些参数，代理配置将会进入构建缓存。

构建器支持以下代理参数（不区分大小写）：

- `HTTP_PROXY`
- `HTTPS_PROXY`
- `FTP_PROXY`
- `NO_PROXY`
- `ALL_PROXY`

配置构建代理示例：

```console
$ docker build --build-arg HTTP_PROXY=https://my-proxy.example.com .
```

关于代理参数的更多信息，参见：[代理参数](/reference/dockerfile.md#predefined-args)。

## 构建工具配置变量

下列环境变量用于启用、禁用或调整 Buildx 与 BuildKit 的行为。
注意：这些变量不是用来配置构建容器的；它们不会在构建内部可见，也与 Dockerfile 的 `ENV` 指令无关。
这些变量用于配置 Buildx 客户端或 BuildKit 守护进程。

| 变量                                                                         | 类型              | 描述                                                             |
|-----------------------------------------------------------------------------|-------------------|------------------------------------------------------------------|
| [BUILDKIT_COLORS](#buildkit_colors)                                         | String            | 配置终端输出文本的颜色。                                         |
| [BUILDKIT_HOST](#buildkit_host)                                             | String            | 指定用于远程构建器的主机地址。                                   |
| [BUILDKIT_PROGRESS](#buildkit_progress)                                     | String            | 配置进度输出的类型。                                             |
| [BUILDKIT_TTY_LOG_LINES](#buildkit_tty_log_lines)                           | String            | TTY 模式下活动步骤显示的日志行数。                               |
| [BUILDX_BAKE_FILE](#buildx_bake_file)                                       | String            | 指定 `docker buildx bake` 的构建定义文件。                       |
| [BUILDX_BAKE_FILE_SEPARATOR](#buildx_bake_file_separator)                   | String            | 指定 `BUILDX_BAKE_FILE` 的路径分隔符。                           |
| [BUILDX_BAKE_GIT_AUTH_HEADER](#buildx_bake_git_auth_header)                 | String            | 远程 Bake 文件的 HTTP 认证方案。                                 |
| [BUILDX_BAKE_GIT_AUTH_TOKEN](#buildx_bake_git_auth_token)                   | String            | 远程 Bake 文件的 HTTP 认证令牌。                                 |
| [BUILDX_BAKE_GIT_SSH](#buildx_bake_git_ssh)                                 | String            | 远程 Bake 文件的 SSH 认证。                                      |
| [BUILDX_BUILDER](#buildx_builder)                                           | String            | 指定要使用的构建器实例。                                         |
| [BUILDX_CONFIG](#buildx_config)                                             | String            | 指定配置、状态与日志的存放位置。                                 |
| [BUILDX_CPU_PROFILE](#buildx_cpu_profile)                                   | String            | 在指定路径生成 `pprof` CPU 性能剖析文件。                        |
| [BUILDX_EXPERIMENTAL](#buildx_experimental)                                 | Boolean           | 启用实验性功能。                                                 |
| [BUILDX_GIT_CHECK_DIRTY](#buildx_git_check_dirty)                           | Boolean           | 启用 Git 脏工作区检测。                                          |
| [BUILDX_GIT_INFO](#buildx_git_info)                                         | Boolean           | 在来源证明中移除 Git 信息。                                      |
| [BUILDX_GIT_LABELS](#buildx_git_labels)                                     | String \| Boolean | 为镜像添加基于 Git 的来源标签。                                  |
| [BUILDX_MEM_PROFILE](#buildx_mem_profile)                                   | String            | 在指定路径生成 `pprof` 内存剖析文件。                            |
| [BUILDX_METADATA_PROVENANCE](#buildx_metadata_provenance)                   | String \| Boolean | 自定义写入元数据文件的来源信息。                                 |
| [BUILDX_METADATA_WARNINGS](#buildx_metadata_warnings)                       | String            | 在元数据文件中包含构建告警。                                     |
| [BUILDX_NO_DEFAULT_ATTESTATIONS](#buildx_no_default_attestations)           | Boolean           | 关闭默认来源证明。                                               |
| [BUILDX_NO_DEFAULT_LOAD](#buildx_no_default_load)                           | Boolean           | 关闭默认将镜像加载到本地镜像存储。                               |
| [EXPERIMENTAL_BUILDKIT_SOURCE_POLICY](#experimental_buildkit_source_policy) | String            | 指定 BuildKit 源策略文件。                                       |

BuildKit 还支持一些额外的配置参数，参见：[BuildKit 内置构建参数](/reference/dockerfile.md#buildkit-built-in-build-args)。

你可以用不同的方式为环境变量表达布尔值。例如，`true`、`1`、`T` 都会被解析为 true。
解析逻辑依赖 Go 标准库的 `strconv.ParseBool` 函数。详情参见其[参考文档](https://pkg.go.dev/strconv#ParseBool)。

<!-- vale Docker.HeadingSentenceCase = NO -->

### BUILDKIT_COLORS

用于改变终端输出的颜色。将 `BUILDKIT_COLORS` 设置为如下格式的 CSV 字符串：

```console
$ export BUILDKIT_COLORS="run=123,20,245:error=yellow:cancel=blue:warning=white"
```

颜色值可以是任意有效的 RGB 十六进制值，或使用[BuildKit 预定义颜色](https://github.com/moby/buildkit/blob/master/util/progress/progressui/colors.go)中的名称。

按照 [no-color.org](https://no-color.org/) 的建议，只要设置了 `NO_COLOR`，就会关闭彩色输出。

### BUILDKIT_HOST

{{< summary-bar feature_name="Buildkit host" >}}

使用 `BUILDKIT_HOST` 可以指定一个 BuildKit 守护进程地址，作为远程构建器使用。
其效果等同于在 `docker buildx create` 中以位置参数的形式传入该地址。

用法：

```console
$ export BUILDKIT_HOST=tcp://localhost:1234
$ docker buildx create --name=remote --driver=remote
```

如果同时设置了 `BUILDKIT_HOST` 环境变量并在命令中提供了位置参数，以位置参数为准。

### BUILDKIT_PROGRESS

设置 BuildKit 进度输出的类型。可选值：

- `auto` (default)
- `plain`
- `tty`
- `quiet`
- `rawjson`

用法：

```console
$ export BUILDKIT_PROGRESS=plain
```

### BUILDKIT_TTY_LOG_LINES

在 TTY 模式下，你可以通过设置 `BUILDKIT_TTY_LOG_LINES` 调整活动步骤可见的日志行数（默认 `6`）。

```console
$ export BUILDKIT_TTY_LOG_LINES=8
```

### EXPERIMENTAL_BUILDKIT_SOURCE_POLICY

允许你指定一个[BuildKit 源策略](https://github.com/moby/buildkit/blob/master/docs/build-repro.md#reproducing-the-pinned-dependencies)文件，用于基于固定依赖实现可复现构建。

```console
$ export EXPERIMENTAL_BUILDKIT_SOURCE_POLICY=./policy.json
```

示例：

```json
{
  "rules": [
    {
      "action": "CONVERT",
      "selector": {
        "identifier": "docker-image://docker.io/library/alpine:latest"
      },
      "updates": {
        "identifier": "docker-image://docker.io/library/alpine:latest@sha256:4edbd2beb5f78b1014028f4fbb99f3237d9561100b6881aabbf5acce2c4f9454"
      }
    },
    {
      "action": "CONVERT",
      "selector": {
        "identifier": "https://raw.githubusercontent.com/moby/buildkit/v0.10.1/README.md"
      },
      "updates": {
        "attrs": {"http.checksum": "sha256:6e4b94fc270e708e1068be28bd3551dc6917a4fc5a61293d51bb36e6b75c4b53"}
      }
    },
    {
      "action": "DENY",
      "selector": {
        "identifier": "docker-image://docker.io/library/golang*"
      }
    }
  ]
}
```

### BUILDX_BAKE_FILE

{{< summary-bar feature_name="Buildx bake file" >}}

为 `docker buildx bake` 指定一个或多个构建定义文件。

该环境变量可替代命令行标志 `-f` / `--file`。

如需指定多个文件，可使用系统路径分隔符分隔（Linux/macOS 使用 `:`，Windows 使用 `;`）：

```console
export BUILDX_BAKE_FILE=file1.hcl:file2.hcl
```

或者使用由 [BUILDX_BAKE_FILE_SEPARATOR](#buildx_bake_file_separator) 定义的自定义分隔符：

```console
export BUILDX_BAKE_FILE_SEPARATOR=@
export BUILDX_BAKE_FILE=file1.hcl@file2.hcl
```

当同时设置了 `BUILDX_BAKE_FILE` 与 `-f` 时，只会使用通过 `-f` 指定的文件。

若列表中的某个文件不存在或无效，bake 会返回错误。

### BUILDX_BAKE_FILE_SEPARATOR

{{< summary-bar feature_name="Buildx bake file separator" >}}

控制 `BUILDX_BAKE_FILE` 环境变量中文件路径的分隔符。

当文件路径本身包含默认分隔符，或你希望在不同平台间统一分隔符时，这很有用。

```console
export BUILDX_BAKE_PATH_SEPARATOR=@
export BUILDX_BAKE_FILE=file1.hcl@file2.hcl
```

### BUILDX_BAKE_GIT_AUTH_HEADER

{{< summary-bar feature_name="Buildx bake Git auth token" >}}

当在私有 Git 仓库中使用远程 Bake 定义时，设置 HTTP 认证方案。
其效果等同于 [`GIT_AUTH_HEADER` 机密](./secrets#http-authentication-scheme)，
但可在 Bake 加载远程定义前完成预检认证。支持的取值包括 `bearer`（默认）与 `basic`。

用法：

```console
$ export BUILDX_BAKE_GIT_AUTH_HEADER=basic
```

### BUILDX_BAKE_GIT_AUTH_TOKEN

{{< summary-bar feature_name="Buildx bake Git auth token" >}}

当在私有 Git 仓库中使用远程 Bake 定义时，设置 HTTP 认证令牌。
其效果等同于 [`GIT_AUTH_TOKEN` 机密](./secrets#git-authentication-for-remote-contexts)，
但可在 Bake 加载远程定义前完成预检认证。

用法：

```console
$ export BUILDX_BAKE_GIT_AUTH_TOKEN=$(cat git-token.txt)
```

### BUILDX_BAKE_GIT_SSH

{{< summary-bar feature_name="Buildx bake Git SSH" >}}

允许你提供一组 SSH agent 套接字路径，在使用私有仓库中的远程 Bake 定义时转发给 Bake，用于对 Git 服务器进行认证。
这与构建时的 SSH 挂载类似，但可在 Bake 解析构建定义前完成预检认证。

通常不必设置该变量，因为 Bake 默认会使用 `SSH_AUTH_SOCK` 指向的 agent 套接字。
只有当你想使用其他路径的套接字时才需要设置。
该变量支持以逗号分隔的多路径字符串。

用法：

```console
$ export BUILDX_BAKE_GIT_SSH=/run/foo/listener.sock,~/.creds/ssh.sock
```

### BUILDX_BUILDER

覆盖当前配置的构建器实例。等同于命令行标志 `docker buildx --builder`。

用法：

```console
$ export BUILDX_BUILDER=my-builder
```

### BUILDX_CONFIG

使用 `BUILDX_CONFIG` 指定用于构建配置、状态与日志的目录。该目录的查找顺序如下：

- `$BUILDX_CONFIG`
- `$DOCKER_CONFIG/buildx`
- `~/.docker/buildx` (default)

用法：

```console
$ export BUILDX_CONFIG=/usr/local/etc
```

### BUILDX_CPU_PROFILE

{{< summary-bar feature_name="Buildx CPU profile" >}}

设置后，Buildx 会在指定位置生成 `pprof` CPU 性能剖析文件。

> [!NOTE]
> 此属性仅对 Buildx 的开发与调试有用。
> 这些剖析数据并不适用于分析实际构建的性能。

用法：

```console
$ export BUILDX_CPU_PROFILE=buildx_cpu.prof
```

### BUILDX_EXPERIMENTAL

启用实验性构建功能。

用法：

```console
$ export BUILDX_EXPERIMENTAL=1
```

### BUILDX_GIT_CHECK_DIRTY

{{< summary-bar feature_name="Buildx Git check dirty" >}}

设置为 true 时，在[来源证明](/manuals/build/metadata/attestations/slsa-provenance.md)中检查源码管理信息是否处于脏状态。

用法：

```console
$ export BUILDX_GIT_CHECK_DIRTY=1
```

### BUILDX_GIT_INFO

{{< summary-bar feature_name="Buildx Git info" >}}

设置为 false 时，从[来源证明](/manuals/build/metadata/attestations/slsa-provenance.md)中移除源码管理信息。

用法：

```console
$ export BUILDX_GIT_INFO=0
```

### BUILDX_GIT_LABELS

{{< summary-bar feature_name="Buildx Git labels" >}}

为构建出的镜像添加基于 Git 信息的来源标签。包括：

- `com.docker.image.source.entrypoint`：Dockerfile 相对项目根目录的位置
- `org.opencontainers.image.revision`：Git 提交修订版本
- `org.opencontainers.image.source`：仓库的 SSH 或 HTTPS 地址

示例：

```json
  "Labels": {
    "com.docker.image.source.entrypoint": "Dockerfile",
    "org.opencontainers.image.revision": "5734329c6af43c2ae295010778cd308866b95d9b",
    "org.opencontainers.image.source": "git@github.com:foo/bar.git"
  }
```

用法：

- 设置 `BUILDX_GIT_LABELS=1`，包含 `entrypoint` 与 `revision` 两个标签。
- 设置 `BUILDX_GIT_LABELS=full`，包含所有标签。

如果仓库处于脏状态，`revision` 标签值会追加 `-dirty` 后缀。

### BUILDX_MEM_PROFILE

{{< summary-bar feature_name="Buildx mem profile" >}}

设置后，Buildx 会在指定位置生成 `pprof` 内存剖析文件。

> [!NOTE]
> 此属性仅对 Buildx 的开发与调试有用。
> 这些剖析数据并不适用于分析实际构建的性能。

用法：

```console
$ export BUILDX_MEM_PROFILE=buildx_mem.prof
```

### BUILDX_METADATA_PROVENANCE

{{< summary-bar feature_name="Buildx metadata provenance" >}}

默认情况下，Buildx 会通过 [`--metadata-file`](/reference/cli/docker/buildx/build/#metadata-file) 在元数据文件中写入最小化的来源信息。
该环境变量允许你自定义写入的来源信息级别：
* `min`：最小化来源信息（默认）
* `max`：完整来源信息
* `disabled`、`false` 或 `0`：不写入来源信息

### BUILDX_METADATA_WARNINGS

{{< summary-bar feature_name="Buildx metadata warnings" >}}

默认情况下，Buildx 不会通过 [`--metadata-file`](/reference/cli/docker/buildx/build/#metadata-file) 将构建告警写入元数据文件。
将该变量设置为 `1` 或 `true` 可启用写入。

### BUILDX_NO_DEFAULT_ATTESTATIONS

{{< summary-bar feature_name="Buildx no default" >}}

默认情况下，BuildKit v0.11 及更高版本会为你构建的镜像添加[来源证明](/manuals/build/metadata/attestations/slsa-provenance.md)。
设置 `BUILDX_NO_DEFAULT_ATTESTATIONS=1` 可禁用默认来源证明。

用法：

```console
$ export BUILDX_NO_DEFAULT_ATTESTATIONS=1
```

### BUILDX_NO_DEFAULT_LOAD

当使用 `docker` 驱动进行构建时，构建完成后镜像会自动加载到本地镜像存储。
设置 `BUILDX_NO_DEFAULT_LOAD` 可以禁用该行为。

用法：

```console
$ export BUILDX_NO_DEFAULT_LOAD=1
```

<!-- vale Docker.HeadingSentenceCase = YES -->
