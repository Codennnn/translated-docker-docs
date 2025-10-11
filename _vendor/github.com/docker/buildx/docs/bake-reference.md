---
title: Bake 文件参考
---

Bake 文件用于定义通过 `docker buildx bake` 运行的构建工作流。

## 文件格式

你可以使用以下文件格式编写 Bake 文件：

- HashiCorp 配置语言（HCL）
- JSON
- YAML（Compose 文件）

默认情况下，Bake 按如下顺序查找配置文件：

1. `compose.yaml`
2. `compose.yml`
3. `docker-compose.yml`
4. `docker-compose.yaml`
5. `docker-bake.json`
6. `docker-bake.hcl`
7. `docker-bake.override.json`
8. `docker-bake.override.hcl`

你可以使用 `--file` 显式指定文件位置：

```console
$ docker buildx bake --file ../docker/bake.hcl --print
```

如果未显式指定文件，Bake 会在当前工作目录中查找。若找到多个 Bake 文件，将按查找顺序合并为一个定义。也就是说，如果项目同时包含 `compose.yaml` 与 `docker-bake.hcl`，Bake 会先加载 `compose.yaml`，再加载 `docker-bake.hcl`。

当合并的文件包含重复的属性定义时，具体是合并还是被后者覆盖，取决于属性类型。以下属性会被最后出现的定义覆盖：

- `target.cache-to`
- `target.dockerfile-inline`
- `target.dockerfile`
- `target.outputs`
- `target.platforms`
- `target.pull`
- `target.tags`
- `target.target`

例如，如果 `compose.yaml` 与 `docker-bake.hcl` 都定义了 `tags` 属性，则以 `docker-bake.hcl` 为准。

```console
$ cat compose.yaml
services:
  webapp:
    build:
      context: .
      tags:
        - bar
$ cat docker-bake.hcl
target "webapp" {
  tags = ["foo"]
}
$ docker buildx bake --print webapp
{
  "group": {
    "default": {
      "targets": [
        "webapp"
      ]
    }
  },
  "target": {
    "webapp": {
      "context": ".",
      "dockerfile": "Dockerfile",
      "tags": [
        "foo"
      ]
    }
  }
}
```

其他属性会被合并。比如，若 `compose.yaml` 与 `docker-bake.hcl` 都对 `labels` 定义了不同的键值，则两者都会被包含；若存在相同的键，则后者覆盖前者。

```console
$ cat compose.yaml
services:
  webapp:
    build:
      context: .
      labels: 
        com.example.foo: "foo"
        com.example.name: "Alice"
$ cat docker-bake.hcl
target "webapp" {
  labels = {
    "com.example.bar" = "bar"
    "com.example.name" = "Bob"
  }
}
$ docker buildx bake --print webapp
{
  "group": {
    "default": {
      "targets": [
        "webapp"
      ]
    }
  },
  "target": {
    "webapp": {
      "context": ".",
      "dockerfile": "Dockerfile",
      "labels": {
        "com.example.foo": "foo",
        "com.example.bar": "bar",
        "com.example.name": "Bob"
      }
    }
  }
}
```

## 语法

Bake 文件支持以下属性类型：

- `target`：构建目标
- `group`：构建目标集合
- `variable`：构建参数与变量
- `function`：自定义 Bake 函数

在 Bake 文件中，以分层块的形式定义属性；每个属性可以包含一个或多个具体字段（attributes）。

下面的片段展示了一个简单 Bake 文件的 JSON 表达形式。该文件包含三个属性：一个变量、一个分组与一个目标。

```json
{
  "variable": {
    "TAG": {
      "default": "latest"
    }
  },
  "group": {
    "default": {
      "targets": ["webapp"]
    }
  },
  "target": {
    "webapp": {
      "dockerfile": "Dockerfile",
      "tags": ["docker.io/username/webapp:${TAG}"]
    }
  }
}
```

在 JSON 表达中，属性是对象（object），其字段为对象上的键值。

下面是同一份 Bake 文件的 HCL 版本：

```hcl
variable "TAG" {
  default = "latest"
}

group "default" {
  targets = ["webapp"]
}

target "webapp" {
  dockerfile = "Dockerfile"
  tags = ["docker.io/username/webapp:${TAG}"]
}
```

HCL 是撰写 Bake 文件的首选格式。除了语法差异外，HCL 还支持 JSON 与 YAML 不具备的特性。

本文档中的示例均使用 HCL。

## 目标（Target）

一个目标对应一次 `docker build` 调用。来看下面这条构建命令：

```console
$ docker build \
  --file=Dockerfile.webapp \
  --tag=docker.io/username/webapp:latest \
  https://github.com/username/webapp
```

可以在 Bake 文件中这样表达：

```hcl
target "webapp" {
  dockerfile = "Dockerfile.webapp"
  tags = ["docker.io/username/webapp:latest"]
  context = "https://github.com/username/webapp"
}
```

下表列出了可用于目标的全部属性：

| Name                                            | Type    | 描述                                                                 |
|-------------------------------------------------|---------|----------------------------------------------------------------------|
| [`args`](#targetargs)                           | Map     | 构建参数                                                             |
| [`annotations`](#targetannotations)             | List    | 导出器注解                                                           |
| [`attest`](#targetattest)                       | List    | 构建证明（attestations）                                            |
| [`cache-from`](#targetcache-from)               | List    | 外部缓存来源                                                         |
| [`cache-to`](#targetcache-to)                   | List    | 外部缓存目标                                                         |
| [`call`](#targetcall)                           | String  | 指定目标要调用的前端方法                                             |
| [`context`](#targetcontext)                     | String  | 指定路径或 URL 下的一组构建文件                                      |
| [`contexts`](#targetcontexts)                   | Map     | 额外的构建上下文                                                     |
| [`description`](#targetdescription)             | String  | 目标描述                                                             |
| [`dockerfile-inline`](#targetdockerfile-inline) | String  | 内联 Dockerfile 字符串                                               |
| [`dockerfile`](#targetdockerfile)               | String  | Dockerfile 路径                                                      |
| [`entitlements`](#targetentitlements)           | List    | 构建过程运行所需的权限                                               |
| [`extra-hosts`](#targetextra-hosts)             | List    | 自定义主机名到 IP 的映射                                             |
| [`inherits`](#targetinherits)                   | List    | 从其他目标继承属性                                                   |
| [`labels`](#targetlabels)                       | Map     | 镜像元数据                                                           |
| [`matrix`](#targetmatrix)                       | Map     | 定义变量集，将目标分叉为多个变体                                     |
| [`name`](#targetname)                           | String  | 在矩阵策略中覆盖目标名称                                             |
| [`no-cache-filter`](#targetno-cache-filter)     | List    | 对特定阶段禁用构建缓存                                               |
| [`no-cache`](#targetno-cache)                   | Boolean | 完全禁用构建缓存                                                     |
| [`output`](#targetoutput)                       | List    | 输出目标                                                             |
| [`platforms`](#targetplatforms)                 | List    | 目标平台                                                             |
| [`pull`](#targetpull)                           | Boolean | 始终拉取镜像                                                         |
| [`secret`](#targetsecret)                       | List    | 暴露给构建的机密                                                     |
| [`shm-size`](#targetshm-size)                   | List    | `/dev/shm` 大小                                                      |
| [`ssh`](#targetssh)                             | List    | 暴露给构建的 SSH agent 套接字或密钥                                  |
| [`tags`](#targettags)                           | List    | 镜像名称与标签                                                       |
| [`target`](#targettarget)                       | String  | 目标构建阶段                                                         |
| [`ulimits`](#targetulimits)                     | List    | Ulimit 选项                                                          |

### `target.args`

使用 `args` 属性为该目标定义构建参数。
其效果等同于在构建命令中传递 [`--build-arg`][build-arg] 标志。

```hcl
target "default" {
  args = {
    VERSION = "0.0.0+unknown"
  }
}
```

你可以将 `args` 的某个键设置为 `null`。
这样会强制该目标使用 Dockerfile 中对应 `ARG` 指令的取值。

```hcl
variable "GO_VERSION" {
  default = "1.20.3"
}

target "webapp" {
  dockerfile = "webapp.Dockerfile"
  tags = ["docker.io/username/webapp"]
}

target "db" {
  args = {
    GO_VERSION = null
  }
  dockerfile = "db.Dockerfile"
  tags = ["docker.io/username/db"]
}
```

### `target.annotations`

`annotations` 属性用于为 bake 构建出的镜像添加注解（annotation）。
其取值是注解列表，格式为 `KEY=VALUE`。

```hcl
target "default" {
  output = [{ type = "image", name = "foo" }]
  annotations = ["org.opencontainers.image.authors=dvdksn"]
}
```

默认情况下，注解会添加到镜像清单（manifest）。
你可以在注解前添加前缀来指定注解层级，前缀为一个逗号分隔的层级列表。
下面的示例会同时把注解添加到镜像索引与清单。

```hcl
target "default" {
  output = [
    {
      type = "image"
      name = "foo"
    }
  ]
  annotations = ["index,manifest:org.opencontainers.image.authors=dvdksn"]
}
```

关于支持的层级，参见
[指定注解层级](https://docs.docker.com/build/building/annotations/#specifying-annotation-levels)。

### `target.attest`

`attest` 属性可为目标应用[构建证明（attestations）][attestations]。
该属性接收证明参数的长格式 CSV 表达。

```hcl
target "default" {
  attest = [
    {
      type = "provenance"
      mode = "max"
    },
    {
      type = "sbom"
    }
  ]
}
```

### `target.cache-from`

构建缓存来源。
构建器会从你指定的位置导入缓存。
该功能使用 [Buildx 缓存后端][cache-backends]，其行为与 [`--cache-from`][cache-from] 标志一致。
此属性为列表，可指定多个缓存来源。

```hcl
target "app" {
  cache-from = [
    {
      type = "s3"
      region = "eu-west-1"
      bucket = "mybucket"
    },
    {
      type = "registry"
      ref = "user/repo:cache"
    }
  ]
}
```

### `target.cache-to`

构建缓存导出目标。
构建器会将构建缓存导出到你指定的位置。
该功能使用 [Buildx 缓存后端][cache-backends]，其行为与 [`--cache-to`][cache-to] 标志一致。
此属性为列表，可指定多个导出目标。

```hcl
target "app" {
  cache-to = [
    {
      type = "s3"
      region = "eu-west-1"
      bucket = "mybucket"
    },
    {
      type = "inline"
    }
  ]
}
```

### `target.call`

指定要使用的前端方法。
通过前端方法，你可以只执行构建检查而不实际构建等。
其行为与 `--call` 标志一致。

```hcl
target "app" {
  call = "check"
}
```

可选值：

- `build`：执行构建（默认）
- `check`：评估该目标的[构建检查](https://docs.docker.com/build/checks/)
- `outline`：展示该目标的构建参数及其默认值（若有）
- `targets`：列出已加载定义中的全部 Bake 目标及其[描述](#targetdescription)

关于前端方法的更多信息，参见命令参考
[`docker buildx build --call`](https://docs.docker.com/reference/cli/docker/buildx/build/#call)。

### `target.context`

指定该目标使用的构建上下文位置。
可接受 URL 或目录路径。
其行为与构建命令位置参数中的[构建上下文][context]一致。

```hcl
target "app" {
  context = "./src/www"
}
```

默认解析为当前工作目录（`"."`）。

```console
$ docker buildx bake --print -f - <<< 'target "default" {}'
[+] Building 0.0s (0/0)
{
  "target": {
    "default": {
      "context": ".",
      "dockerfile": "Dockerfile"
    }
  }
}
```

### `target.contexts`

额外的构建上下文。
其行为与 [`--build-context`][build-context] 标志一致。
该属性为映射（map），键名会成为可在构建中引用的命名上下文。

你可以指定不同类型的上下文，例如本地目录、Git URL，甚至其他 Bake 目标。
Bake 会基于取值的模式自动判断上下文类型。

| 上下文类型       | 示例                                      |
| --------------- | ----------------------------------------- |
| 容器镜像         | `docker-image://alpine@sha256:0123456789` |
| Git URL         | `https://github.com/user/proj.git`        |
| HTTP URL        | `https://example.com/files`               |
| 本地目录         | `../path/to/src`                          |
| Bake 目标       | `target:base`                             |

#### 固定镜像版本

```hcl
# docker-bake.hcl
target "app" {
  contexts = {
    alpine = "docker-image://alpine:3.13"
  }
}
```

```Dockerfile
# Dockerfile
FROM alpine
RUN echo "Hello world"
```

#### 使用本地目录

```hcl
# docker-bake.hcl
target "app" {
  contexts = {
    src = "../path/to/source"
  }
}
```

```Dockerfile
# Dockerfile
FROM scratch AS src
FROM golang
COPY --from=src . .
```

#### 使用其他目标作为基础

> [!NOTE]
> 一般应优先使用常规的多阶段构建而非此选项。
> 当你有多个难以合并到同一份的 Dockerfile 时，可考虑使用该特性。

```hcl
# docker-bake.hcl
target "base" {
  dockerfile = "baseapp.Dockerfile"
}

target "app" {
  contexts = {
    baseapp = "target:base"
  }
}
```

```Dockerfile
# Dockerfile
FROM baseapp
RUN echo "Hello world"
```

### `target.description`

为目标定义面向人的描述，说明其用途或功能。

```hcl
target "lint" {
  description = "Runs golangci-lint to detect style errors"
  args = {
    GOLANGCI_LINT_VERSION = null
  }
  dockerfile = "lint.Dockerfile"
}
```

结合 `docker buildx bake --list=targets` 使用时，这一属性可在列出可用目标时提供更丰富的信息。

### `target.dockerfile-inline`

将字符串值作为该构建目标的内联 Dockerfile 使用。

```hcl
target "default" {
  dockerfile-inline = "FROM alpine\nENTRYPOINT [\"echo\", \"hello\"]"
}
```

当同时指定 `dockerfile-inline` 与 `dockerfile` 时，前者优先生效。
若两者同时存在，Bake 使用内联版本。

### `target.dockerfile`

指定用于构建的 Dockerfile 名称。
其行为与 `docker build` 命令的 [`--file`][file] 标志一致。

```hcl
target "default" {
  dockerfile = "./src/www/Dockerfile"
}
```

默认解析为 `"Dockerfile"`。

```console
$ docker buildx bake --print -f - <<< 'target "default" {}'
[+] Building 0.0s (0/0)
{
  "target": {
    "default": {
      "context": ".",
      "dockerfile": "Dockerfile"
    }
  }
}
```

### `target.entitlements`

Entitlements 是构建过程运行所需的权限集合。

当前支持的权限包括：

- `network.host`: Allows the build to use commands that access the host network. In Dockerfile, use [`RUN --network=host`](https://docs.docker.com/reference/dockerfile/#run---networkhost) to run a command with host network enabled.

- `security.insecure`: Allows the build to run commands in privileged containers that are not limited by the default security sandbox. Such container may potentially access and modify system resources. In Dockerfile, use [`RUN --security=insecure`](https://docs.docker.com/reference/dockerfile/#run---security) to run a command in a privileged container.

```hcl
target "integration-tests" {
  # 该目标需要特权容器以运行嵌套容器
  entitlements = ["security.insecure"]
}
```

启用 Entitlements 包含两个步骤：首先，目标需声明其所需的权限；其次，在调用 `bake` 命令时，用户需通过传入 `--allow` 标志或在交互式终端提示时确认来授予权限。此流程可确保用户知悉其授予构建过程的潜在不安全权限。

### `target.extra-hosts`

使用 `extra-hosts` 为该目标定义自定义的主机名到 IP 的映射。
其行为与构建命令的 [`--add-host`][add-host] 标志一致。

```hcl
target "default" {
  extra-hosts = {
    my_hostname = "8.8.8.8"
  }
}
```

### `target.inherits`

一个目标可以从其他目标继承属性。
使用 `inherits` 在目标之间进行引用与复用。

如下示例中，`app-dev` 指定了镜像名称与标签；
`app-release` 通过 `inherits` 复用该标签配置。

```hcl
variable "TAG" {
  default = "latest"
}

target "app-dev" {
  tags = ["docker.io/username/myapp:${TAG}"]
}

target "app-release" {
  inherits = ["app-dev"]
  platforms = ["linux/amd64", "linux/arm64"]
}
```

`inherits` 是列表类型，这意味着可以同时复用多个其他目标的属性。
在下例中，`app-release` 同时复用了 `app-dev` 与 `_release` 的属性。

```hcl
target "app-dev" {
  args = {
    GO_VERSION = "1.20"
    BUILDX_EXPERIMENTAL = 1
  }
  tags = ["docker.io/username/myapp"]
  dockerfile = "app.Dockerfile"
  labels = {
    "org.opencontainers.image.source" = "https://github.com/username/myapp"
  }
}

target "_release" {
  args = {
    BUILDKIT_CONTEXT_KEEP_GIT_DIR = 1
    BUILDX_EXPERIMENTAL = 0
  }
}

target "app-release" {
  inherits = ["app-dev", "_release"]
  platforms = ["linux/amd64", "linux/arm64"]
}
```

当从多个目标继承且发生冲突时，`inherits` 列表中靠后的目标优先生效。
上例在 `app-release` 中两次定义了 `BUILDX_EXPERIMENTAL`，
由于 `_release` 在继承链中靠后，最终取值为 `0`：

```console
$ docker buildx bake --print app-release
[+] Building 0.0s (0/0)
{
  "group": {
    "default": {
      "targets": [
        "app-release"
      ]
    }
  },
  "target": {
    "app-release": {
      "context": ".",
      "dockerfile": "app.Dockerfile",
      "args": {
        "BUILDKIT_CONTEXT_KEEP_GIT_DIR": "1",
        "BUILDX_EXPERIMENTAL": "0",
        "GO_VERSION": "1.20"
      },
      "labels": {
        "org.opencontainers.image.source": "https://github.com/username/myapp"
      },
      "tags": [
        "docker.io/username/myapp"
      ],
      "platforms": [
        "linux/amd64",
        "linux/arm64"
      ]
    }
  }
}
```

### `target.labels`

为构建结果设置镜像标签（label）。
其行为与 `docker build` 的 `--label` 标志一致。

```hcl
target "default" {
  labels = {
    "org.opencontainers.image.source" = "https://github.com/username/myapp"
    "com.docker.image.source.entrypoint" = "Dockerfile"
  }
}
```

`labels` 的某个键可以设置为 `null`。
此时会使用 Dockerfile 中定义的标签取值。

### `target.matrix`

矩阵策略（matrix）可基于你指定的参数，将单个目标分叉为多个不同变体。
其工作方式与 [GitHub Actions 的矩阵策略] 类似。
这有助于减少 bake 定义中的重复。

`matrix` 是一个将参数名映射到取值列表的映射。
Bake 会对所有组合分别生成独立目标进行构建。

每个生成的目标都**必须**有唯一名称。
可通过 `name` 属性指定命名解析方式。

下例将 `app` 解析为 `app-foo` 与 `app-bar`，并使用矩阵值设置[目标构建阶段](#targettarget)。

```hcl
target "app" {
  name = "app-${tgt}"
  matrix = {
    tgt = ["foo", "bar"]
  }
  target = tgt
}
```

```console
$ docker buildx bake --print app
[+] Building 0.0s (0/0)
{
  "group": {
    "app": {
      "targets": [
        "app-foo",
        "app-bar"
      ]
    },
    "default": {
      "targets": [
        "app"
      ]
    }
  },
  "target": {
    "app-bar": {
      "context": ".",
      "dockerfile": "Dockerfile",
      "target": "bar"
    },
    "app-foo": {
      "context": ".",
      "dockerfile": "Dockerfile",
      "target": "foo"
    }
  }
}
```

#### 多维矩阵

你可以在矩阵中指定多个键，使目标在多维上进行分叉。
当使用多个矩阵键时，Bake 会构建所有可能的变体。

下例将生成四个目标：

- `app-foo-1-0`
- `app-foo-2-0`
- `app-bar-1-0`
- `app-bar-2-0`

```hcl
target "app" {
  name = "app-${tgt}-${replace(version, ".", "-")}"
  matrix = {
    tgt = ["foo", "bar"]
    version = ["1.0", "2.0"]
  }
  target = tgt
  args = {
    VERSION = version
  }
}
```

#### 每个矩阵目标包含多个值

如果希望矩阵项不仅由单一值区分，可以使用映射（map）作为矩阵取值。
Bake 会为每个映射创建一个目标，你可通过点号语法访问嵌套字段。

下例将生成两个目标：

- `app-foo-1-0`
- `app-bar-2-0`

```hcl
target "app" {
  name = "app-${item.tgt}-${replace(item.version, ".", "-")}"
  matrix = {
    item = [
      {
        tgt = "foo"
        version = "1.0"
      },
      {
        tgt = "bar"
        version = "2.0"
      }
    ]
  }
  target = item.tgt
  args = {
    VERSION = item.version
  }
}
```

### `target.name`

为使用矩阵策略的目标指定命名解析方式。
下例将 `app` 解析为 `app-foo` 与 `app-bar`。

```hcl
target "app" {
  name = "app-${tgt}"
  matrix = {
    tgt = ["foo", "bar"]
  }
  target = tgt
}
```

### `target.network`

为整个构建请求指定网络模式。
这将覆盖 Dockerfile 中所有 `RUN` 指令的默认网络模式。
可选值为 `default`、`host` 与 `none`。

通常，更好的做法是在 Dockerfile 中按步骤使用 `RUN --network=<value>` 设置网络模式。
这样你可以为单个构建步骤设置网络模式，构建者不必在命令中额外传参即可获得一致行为。

若你在 Bake 文件中将网络模式设为 `host`，在调用 `bake` 命令时还必须授予 `network.host` 权限。
因为 `host` 网络模式需要更高的权限，且可能带来安全风险。
你可以通过向 `docker buildx bake` 传递 `--allow=network.host` 来授予，或在交互式终端提示时确认。

```hcl
target "app" {
  # 确保该构建无法访问互联网
  network = "none"
}
```

### `target.no-cache-filter`

为指定阶段禁用构建缓存。
其行为与 `docker build` 的 `--no-cache-filter` 标志一致。
下例禁用了阶段 `foo` 的缓存。

```hcl
target "default" {
  no-cache-filter = ["foo"]
}
```

### `target.no-cache`

在构建镜像时不使用缓存。
其行为与 `docker build` 的 `--no-cache` 标志一致。

```hcl
target "default" {
  no-cache = 1
}
```

### `target.output`

配置构建产物的导出方式。
其行为与 [`--output`][output] 标志一致。
下例将目标配置为仅导出缓存：

```hcl
target "default" {
  output = [{ type = "cacheonly" }]
}
```

### `target.platforms`

为构建目标设置目标平台。
其行为与 [`--platform`][platform] 标志一致。
下例为三种架构创建多平台构建。

```hcl
target "default" {
  platforms = ["linux/amd64", "linux/arm64", "linux/arm/v7"]
}
```

### `target.pull`

配置在构建该目标时是否尝试拉取镜像。
其行为与 `docker build` 的 `--pull` 标志一致。
下例强制构建器始终拉取该目标引用的所有镜像。

```hcl
target "default" {
  pull = true
}
```

### `target.secret`

定义要暴露给构建目标的机密（secret）。
其行为与 [`--secret`][secret] 标志一致。

```hcl
variable "HOME" {
  default = null
}

target "default" {
  secret = [
    {
      type = "env"
      id = "KUBECONFIG"
    },
    {
      type = "file"
      id = "aws"
      src = "${HOME}/.aws/credentials"
    }
  ]
}
```

这允许你在 Dockerfile 中[挂载该机密][run_mount_secret]。

```dockerfile
RUN --mount=type=secret,id=aws,target=/root/.aws/credentials \
    aws cloudfront create-invalidation ...
RUN --mount=type=secret,id=KUBECONFIG,env=KUBECONFIG \
    helm upgrade --install
```

### `target.shm-size`

在使用 `RUN` 指令时，设置分配给构建容器的共享内存大小。

格式为 `<数字><单位>`。`数字` 必须大于 `0`。单位可选，取 `b`（字节）、`k`（KB）、`m`（MB）或 `g`（GB）。
若省略单位，系统默认使用字节。

其行为与 `docker build` 的 `--shm-size` 标志一致。

```hcl
target "default" {
  shm-size = "128m"
}
```

> [!NOTE]
> 大多数情况下，建议交由构建器自动决定合适的配置。
> 只有在复杂构建场景下需要进行特定性能调优时，才考虑手动调整。

### `target.ssh`

定义要暴露给构建的 SSH agent 套接字或密钥。
其行为与 [`--ssh`][ssh] 标志一致。
在构建过程中需要访问私有仓库时非常有用。

```hcl
target "default" {
  ssh = [{ id = "default" }]
}
```

```dockerfile
FROM alpine
RUN --mount=type=ssh \
    apk add git openssh-client \
    && install -m 0700 -d ~/.ssh \
    && ssh-keyscan github.com >> ~/.ssh/known_hosts \
    && git clone git@github.com:user/my-private-repo.git
```

### `target.tags`

该构建目标使用的镜像名称与标签。
其行为与 [`--tag`][tag] 标志一致。

```hcl
target "default" {
  tags = [
    "org/repo:latest",
    "myregistry.azurecr.io/team/image:v1"
  ]
}
```

### `target.target`

设置要构建的目标阶段。
其行为与 [`--target`][target] 标志一致。

```hcl
target "default" {
  target = "binaries"
}
```

### `target.ulimits`

Ulimits 可在执行 `RUN` 指令时覆盖构建容器的默认 ulimit 配置。
其格式为 `<类型>=<软限制>[:<硬限制>]`，例如：

```hcl
target "app" {
  ulimits = [
    "nofile=1024:1024"
  ]
}
```

> [!NOTE]
> 若未提供 `硬限制`，则 `软限制` 将同时作为两者的取值。
> 若未设置任何 `ulimits`，将继承守护进程上的默认配置。

> [!NOTE]
> In most cases, it is recommended to let the builder automatically determine
> the appropriate configurations. Manual adjustments should only be considered
> when specific performance tuning is required for complex build scenarios.

## 分组（Group）

分组允许你一次调用多个构建（目标）。

```hcl
group "default" {
  targets = ["db", "webapp-dev"]
}

target "webapp-dev" {
  dockerfile = "Dockerfile.webapp"
  tags = ["docker.io/username/webapp:latest"]
}

target "db" {
  dockerfile = "Dockerfile.db"
  tags = ["docker.io/username/db"]
}
```

当分组与目标同名时，分组优先生效。
下面的 bake 文件将构建名为 `default` 的分组；
同名的 `default` 目标会被忽略。

```hcl
target "default" {
  dockerfile-inline = "FROM ubuntu"
}

group "default" {
  targets = ["alpine", "debian"]
}
target "alpine" {
  dockerfile-inline = "FROM alpine"
}
target "debian" {
  dockerfile-inline = "FROM debian"
}
```

## 变量（Variable）

HCL 文件格式支持定义变量块。
你可以将变量作为 Dockerfile 中的构建参数，
或在 Bake 文件的属性值中进行插值使用。

```hcl
variable "TAG" {
  type = string
  default = "latest"
  description: "Tag to use for build"
}

target "webapp-dev" {
  dockerfile = "Dockerfile.webapp"
  tags = ["docker.io/username/webapp:${TAG}"]
}
```

你可以在 Bake 文件中为变量指定默认值，
也可以赋值为 `null`。若为 `null`，Buildx 将改用 Dockerfile 中的默认值。

你还可以通过 `description` 字段为变量添加用途说明。
结合 `docker buildx bake --list=variables` 选项时，这将使变量列表输出更加信息丰富。

你可以通过环境变量覆盖 Bake 文件中设置的变量默认值。
例如，下面将 `TAG` 变量设置为 `dev`，覆盖上例的默认值 `latest`。

```console
$ TAG=dev docker buildx bake webapp-dev
```

变量也可以显式指定类型。
若提供类型，系统会用其校验默认值（若存在）以及覆盖值。
在需要覆盖的复杂类型场景中尤为有用。
上例可以扩展以便应用任意数量的标签：
```hcl
variable "TAGS" {
  default = ["latest"]
  type = list(string)
}

target "webapp-dev" {
  dockerfile = "Dockerfile.webapp"
  tags = [for tag in TAGS: "docker.io/username/webapp:${tag}"]
}
```

下例展示了无需修改文件或编写自定义函数/解析逻辑即可生成三个标签：
```console
$ TAGS=dev,latest,2 docker buildx bake webapp-dev
```

### 变量类型（Typing）

可用的原始类型：
* `string`
* `number`
* `bool`

类型以关键字形式书写，必须为字面量：
```hcl
variable "OK" {
  type = string
}

# 不能写成实际的字符串
variable "BAD" {
  type = "string"
}

# 不能是表达式的结果
variable "ALSO_BAD" {
  type = lower("string")
}
```
为原始类型显式标注类型有助于表达意图（尤其未提供默认值时），
但通常即使不显式标注类型，bake 也能按预期工作。

复杂类型通过“类型构造器”表达，包括：
* `tuple([<type>,...])`
* `list(<type>)`
* `set(<type>)`
* `map(<type>)`
* `object({<attr>=<type>},...})`

以下给出每种类型的示例，以及（可选）默认值的写法：
```hcl
# 以结构化方式表达 "1.2.3-alpha"
variable "MY_VERSION" {
  type = tuple([number, number, number, string])
  default = [1, 2, 3, "alpha"]
}

# 矩阵构建中使用的 JDK 版本
variable "JDK_VERSIONS" {
  type = list(number)
  default = [11, 17, 21]
}

# 更好的表达方式；同时具备集合的语义，并可使用基于集合的函数
variable "JDK_VERSIONS" {
  type = set(number)
  default = [11, 17, 21]
}

# 借助 lookup()，将 "feature" 转换为标签
variable "FEATURE_TO_NAME" {
  type = map(string)
  default = {featureA = "slim", featureB = "tiny"}
}

# 将分支名称映射到仓库位置
variable "PUSH_DESTINATION" {
  type = object({branch = string, registry = string})
  default = {branch = "main", registry = "prod-registry.invalid.com"}
}

# 结合复合类型让上例更实用
variable "PUSH_DESTINATIONS" {
  type = list(object({branch = string, registry = string}))
  default = [
    {branch = "develop", registry = "test-registry.invalid.com"},
    {branch = "main", registry = "prod-registry.invalid.com"},
  ]
}
```
注意：即便未显式标注类型，上述各示例中的默认值依然是合法的。
如果省略类型，前三个都会被视为 `tuple`，你只能使用作用于 `tuple` 的函数，无法添加元素等；
同样地，第三与第四个都会被视为 `object`，并受其语义与限制约束。
简言之，缺省类型时：用 `[]` 包裹的值为 `tuple`，用 `{}` 包裹的值为 `object`。
为复杂类型显式标注类型不仅允许使用适用于该类型的函数，也是允许进行覆盖的前提。

> [!NOTE]
> 更多细节参见 [HCL 类型表达式][typeexpr]。

### 覆盖变量

如[变量简介](#variable)所述，原始类型（`string`、`number`、`bool`）无需声明类型即可被覆盖，且通常行为符合预期。
（当未显式声明类型且默认值不含 `{}` 或 `[]` 时，变量被视为原始类型；
既未声明类型也无默认值的变量视为 `string`。）
当然，覆盖也可与显式类型并用；这在某些边界场景很有帮助，例如希望 `VAR=true` 被视为 `string`（否则可能因上下文不同被视为 `string` 或 `bool`）。
复杂类型仅在提供了类型时才能被覆盖。
覆盖依旧通过环境变量进行，但值可通过 CSV 或 JSON 提供。

#### 使用 CSV 进行覆盖

这是推荐方式，适合交互式使用。
实践中常见且适合覆盖的复杂类型为 `list` 与 `set`，
因此它们（以及 `tuple`，尽管更偏结构但在此更像集合）均获得完整的 CSV 支持。


`map` 与 `object` 的 CSV 支持有限，复合类型不支持。
对于这类高级场景，可改用[使用 JSON](#json-overrides) 的机制。

#### 使用 JSON 进行覆盖

也可以通过 JSON 提供覆盖。
对于某些复杂类型，这可能是唯一可行的方法；若覆盖值本身已是 JSON（例如来源于 JSON API），也更为便捷。
当值包含引号或逗号，难以用 CSV 表达时，也可使用 JSON。
要启用 JSON 覆盖，只需在变量名后追加 `_JSON`。
在下例中，CSV 无法表示第二个值，因此需使用 JSON：
```hcl
variable "VALS" {
  type = list(string)
  default = ["some", "list"]
}
```
```console
$ cat data.json
["hello","with,comma","with\"quote"]
$ VALS_JSON=$(< data.json) docker buildx bake

# CSV equivalent, though the second value cannot be expressed at all 
$ VALS='hello,"with""quote"' docker buildx bake
```

This example illustrates some precedence and usage rules:
```hcl
variable "FOO" {
  type = string
  default = "foo"
}

variable "FOO_JSON" {
  type = string
  default = "foo"
}
```

The variable `FOO` can *only* be overridden using CSV because `FOO_JSON`, which would typically used for a JSON override,
is already a defined variable.
Since `FOO_JSON` is an actual variable, setting that environment variable would be expected to a CSV value.
A JSON override *is* possible for this variable, using environment variable `FOO_JSON_JSON`.

```console
# 以下三种写法等价，均将变量 FOO 设为 bar
$ FOO=bar docker buildx bake <...>
$ FOO='bar' docker buildx bake <...>
$ FOO="bar" docker buildx bake <...>

# 仅设置变量 FOO_JSON；不影响 FOO
$ FOO_JSON=bar docker buildx bake <...>

# 这也会设置 FOO_JSON，但因非有效 JSON 而失败
$ FOO_JSON_JSON=bar docker buildx bake <...>

# 以下写法等价
$ cat data.json
"bar"
$ FOO_JSON_JSON=$(< data.json) docker buildx bake <...>
$ FOO_JSON_JSON='"bar"' docker buildx bake <...>
$ FOO_JSON=bar docker buildx bake <...>

# 这会设置两个不同变量，均以 CSV 指定（FOO=bar，FOO_JSON="baz"）
$ FOO=bar FOO_JSON='"baz"' docker buildx bake <...>

# 这两者指向同一变量，其中 FOO_JSON_JSON 具有更高优先级并按 JSON 读取（FOO_JSON=baz）
$ FOO_JSON=bar FOO_JSON_JSON='"baz"' docker buildx bake <...>
```

### 内置变量

以下变量为内置变量，无需定义即可在 Bake 中使用：

| 变量                  | 说明                                                                                |
| --------------------- | ----------------------------------------------------------------------------------- |
| `BAKE_CMD_CONTEXT`    | 远程 Bake 文件构建时保存主上下文。                                                  |
| `BAKE_LOCAL_PLATFORM` | 返回当前平台的默认平台标识（例如 `linux/amd64`）。                                  |

### 使用环境变量作为默认值

你可以将 Bake 变量的默认值设为某个环境变量的值：

```hcl
variable "HOME" {
  default = "$HOME"
}
```

### 将变量插入到属性中

若要在属性的字符串值中插入变量，
必须使用花括号包裹变量。
以下写法不可用：

```hcl
variable "HOME" {
  default = "$HOME"
}

target "default" {
  ssh = ["default=$HOME/.ssh/id_rsa"]
}
```

在需要插入的位置，用花括号包裹变量：

```diff
  variable "HOME" {
    default = "$HOME"
  }

  target "default" {
-   ssh = ["default=$HOME/.ssh/id_rsa"]
+   ssh = ["default=${HOME}/.ssh/id_rsa"]
  }
```

在将变量插入属性之前，
必须先在 bake 文件中声明该变量，
如下例所示。

```console
$ cat docker-bake.hcl
target "default" {
  dockerfile-inline = "FROM ${BASE_IMAGE}"
}
$ docker buildx bake
[+] Building 0.0s (0/0)
docker-bake.hcl:2
--------------------
   1 |     target "default" {
   2 | >>>   dockerfile-inline = "FROM ${BASE_IMAGE}"
   3 |     }
   4 |
--------------------
ERROR: docker-bake.hcl:2,31-41: Unknown variable; There is no variable named "BASE_IMAGE"., and 1 other diagnostic(s)
$ cat >> docker-bake.hcl

variable "BASE_IMAGE" {
  default = "alpine"
}

$ docker buildx bake
[+] Building 0.6s (5/5) FINISHED
```

## 函数（Function）

HCL 文件可使用由 [go-cty][go-cty] 提供的[通用函数集][bake_stdlib]：

```hcl
# docker-bake.hcl
target "webapp-dev" {
  dockerfile = "Dockerfile.webapp"
  tags = ["docker.io/username/webapp:latest"]
  args = {
    buildno = "${add(123, 1)}"
  }
}
```

此外，还支持[用户自定义函数][userfunc]：

```hcl
# docker-bake.hcl
function "increment" {
  params = [number]
  result = number + 1
}

target "webapp-dev" {
  dockerfile = "Dockerfile.webapp"
  tags = ["docker.io/username/webapp:latest"]
  args = {
    buildno = "${increment(123)}"
  }
}
```

> [!NOTE]
> 更多细节参见[用户自定义 HCL 函数][hcl-funcs]。

<!-- external links -->

[add-host]: https://docs.docker.com/reference/cli/docker/buildx/build/#add-host
[attestations]: https://docs.docker.com/build/attestations/
[bake_stdlib]: https://github.com/docker/buildx/blob/master/docs/bake-stdlib.md
[build-arg]: https://docs.docker.com/reference/cli/docker/image/build/#build-arg
[build-context]: https://docs.docker.com/reference/cli/docker/buildx/build/#build-context
[cache-backends]: https://docs.docker.com/build/cache/backends/
[cache-from]: https://docs.docker.com/reference/cli/docker/buildx/build/#cache-from
[cache-to]: https://docs.docker.com/reference/cli/docker/buildx/build/#cache-to
[context]: https://docs.docker.com/reference/cli/docker/buildx/build/#build-context
[file]: https://docs.docker.com/reference/cli/docker/image/build/#file
[go-cty]: https://github.com/zclconf/go-cty/tree/main/cty/function/stdlib
[hcl-funcs]: https://docs.docker.com/build/bake/hcl-funcs/
[output]: https://docs.docker.com/reference/cli/docker/buildx/build/#output
[platform]: https://docs.docker.com/reference/cli/docker/buildx/build/#platform
[run_mount_secret]: https://docs.docker.com/reference/dockerfile/#run---mounttypesecret
[secret]: https://docs.docker.com/reference/cli/docker/buildx/build/#secret
[ssh]: https://docs.docker.com/reference/cli/docker/buildx/build/#ssh
[tag]: https://docs.docker.com/reference/cli/docker/image/build/#tag
[target]: https://docs.docker.com/reference/cli/docker/image/build/#target
[typeexpr]: https://github.com/hashicorp/hcl/tree/main/ext/typeexpr
[userfunc]: https://github.com/hashicorp/hcl/tree/main/ext/userfunc
