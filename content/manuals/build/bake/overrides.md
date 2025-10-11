---
title: 覆盖配置
description: 了解如何在 Bake 文件中覆盖配置，以使用不同属性进行构建。
keywords: build, buildx, bake, buildkit, hcl, json, overrides, configuration
aliases:
  - /build/bake/configuring-build/
---

Bake 支持从文件加载构建定义，但有时你需要更灵活的方式来配置这些定义。例如，你可能希望在特定环境下，或针对某个特定目标构建时，覆盖某项属性。

以下属性可以被覆盖：

- `args`
- `attest`
- `cache-from`
- `cache-to`
- `context`
- `contexts`
- `dockerfile`
- `entitlements`
- `labels`
- `network`
- `no-cache`
- `output`
- `platform`
- `pull`
- `secrets`
- `ssh`
- `tags`
- `target`

要覆盖这些属性，你可以使用以下方法：

- [文件覆盖](#file-overrides)
- [命令行覆盖](#command-line)
- [环境变量覆盖](#environment-variables)

## 文件覆盖

你可以加载多个 Bake 文件来为目标定义构建配置。这对于将配置拆分到不同文件以便更好地组织管理，或根据加载的文件有条件地覆盖配置，非常有用。

### 默认文件查找顺序

你可以使用 `--file` 或 `-f` 指定要加载的文件。
如果未指定，Bake 会按以下顺序进行查找：

1. `compose.yaml`
2. `compose.yml`
3. `docker-compose.yml`
4. `docker-compose.yaml`
5. `docker-bake.json`
6. `docker-bake.hcl`
7. `docker-bake.override.json`
8. `docker-bake.override.hcl`

如果找到多个 Bake 文件，所有文件都会被加载并合并为单一定义。文件按上述查找顺序进行合并。

```console
$ docker buildx bake --print
[+] Building 0.0s (1/1) FINISHED                                                                                                                                                                                            
 => [internal] load local bake definitions                                                                                                                                                                             0.0s
 => => reading compose.yaml 45B / 45B                                                                                                                                                                                  0.0s
 => => reading docker-bake.hcl 113B / 113B                                                                                                                                                                             0.0s
 => => reading docker-bake.override.hcl 65B / 65B
```

当合并的文件包含重复的属性定义时，具体会被合并或由最后出现的定义覆盖，取决于该属性的合并策略。

Bake 会按发现顺序尝试加载所有文件。若多个文件定义了相同的目标，这些属性会被合并或被覆盖；发生覆盖时，以最后加载的定义为准。

例如，给定以下文件：

```hcl {title=docker-bake.hcl}
variable "TAG" {
  default = "foo"
}

target "default" {
  tags = ["username/my-app:${TAG}"]
}
```

```hcl {title=docker-bake.override.hcl}
variable "TAG" {
  default = "bar"
}
```

由于 `docker-bake.override.hcl` 在默认查找顺序中最后加载，`TAG` 变量被覆盖为 `bar`。

```console
$ docker buildx bake --print
{
  "target": {
    "default": {
      "context": ".",
      "dockerfile": "Dockerfile",
      "tags": ["username/my-app:bar"]
    }
  }
}
```

### 手动指定文件覆盖

你可以通过 `--file` 标志显式指定要加载的文件，并以此按需应用覆盖文件。

例如，你可以创建一个文件来定义特定环境的一组配置，并只在该环境构建时加载它。下面的示例展示了如何加载一个将 `TAG` 变量设置为 `bar` 的 `overrides.hcl` 文件。随后，`TAG` 变量会在 `default` 目标中被使用。

```hcl {title=docker-bake.hcl}
variable "TAG" {
  default = "foo"
}

target "default" {
  tags = ["username/my-app:${TAG}"]
}
```

```hcl {title=overrides.hcl}
variable "TAG" {
  default = "bar"
}
```

在不使用 `--file` 的情况下打印构建配置时，可以看到 `TAG` 变量为默认值 `foo`。

```console
$ docker buildx bake --print
{
  "target": {
    "default": {
      "context": ".",
      "dockerfile": "Dockerfile",
      "tags": [
        "username/my-app:foo"
      ]
    }
  }
}
```

使用 `--file` 加载 `overrides.hcl` 后，`TAG` 变量会被覆盖为 `bar`。

```console
$ docker buildx bake -f docker-bake.hcl -f overrides.hcl --print
{
  "target": {
    "default": {
      "context": ".",
      "dockerfile": "Dockerfile",
      "tags": [
        "username/my-app:bar"
      ]
    }
  }
}
```

## 命令行

你也可以使用命令行中的[`--set` 标志](/reference/cli/docker/buildx/bake.md#set)覆盖目标配置：

```hcl
# docker-bake.hcl
target "app" {
  args = {
    mybuildarg = "foo"
  }
}
```

```console
$ docker buildx bake --set app.args.mybuildarg=bar --set app.platform=linux/arm64 app --print
```

```json
{
  "group": {
    "default": {
      "targets": ["app"]
    }
  },
  "target": {
    "app": {
      "context": ".",
      "dockerfile": "Dockerfile",
      "args": {
        "mybuildarg": "bar"
      },
      "platforms": ["linux/arm64"]
    }
  }
}
```

同时支持 [https://golang.org/pkg/path/#Match](https://golang.org/pkg/path/#Match) 中定义的模式匹配语法：

```console
$ docker buildx bake --set foo*.args.mybuildarg=value  # overrides build arg for all targets starting with "foo"
$ docker buildx bake --set *.platform=linux/arm64      # overrides platform for all targets
$ docker buildx bake --set foo*.no-cache               # bypass caching only for targets starting with "foo"
```

通过 `--set` 可覆盖的属性包括：

- `args`
- `attest`
- `cache-from`
- `cache-to`
- `context`
- `contexts`
- `dockerfile`
- `entitlements`
- `labels`
- `network`
- `no-cache`
- `output`
- `platform`
- `pull`
- `secrets`
- `ssh`
- `tags`
- `target`

## 环境变量

你也可以使用环境变量来覆盖配置。

Bake 允许使用环境变量覆盖 `variable` 块的值。只有 `variable` 块可以被环境变量覆盖。
这意味着你需要先在 Bake 文件中定义变量，然后设置同名环境变量来覆盖它。

下面的示例展示了如何在 Bake 文件中定义带默认值的 `TAG` 变量，并通过环境变量进行覆盖。

```hcl
variable "TAG" {
  default = "latest"
}

target "default" {
  context = "."
  dockerfile = "Dockerfile"
  tags = ["docker.io/username/webapp:${TAG}"]
}
```

```console
$ export TAG=$(git rev-parse --short HEAD)
$ docker buildx bake --print webapp
```

此时，`TAG` 变量被覆盖为环境变量的值，即 `git rev-parse --short HEAD` 生成的短提交哈希。

```json
{
  "group": {
    "default": {
      "targets": ["webapp"]
    }
  },
  "target": {
    "webapp": {
      "context": ".",
      "dockerfile": "Dockerfile",
      "tags": ["docker.io/username/webapp:985e9e9"]
    }
  }
}
```

### 类型转换

支持使用环境变量覆盖非字符串类型变量。通过环境变量传入的值会先被转换为合适的类型。

下面的示例定义了 `PORT` 变量：`backend` 目标直接使用该值，`frontend` 目标使用 `PORT` 自增 1 的结果。

```hcl
variable "PORT" {
  default = 3000
}

group "default" {
  targets = ["backend", "frontend"]
}

target "backend" {
  args = {
    PORT = PORT
  }
}

target "frontend" {
  args = {
    PORT = add(PORT, 1)
  }
}
```

使用环境变量覆盖 `PORT` 时，值会先被转换为期望的类型（整数），然后再执行 `frontend` 目标中的表达式。

```console
$ PORT=7070 docker buildx bake --print
```

```json
{
  "group": {
    "default": {
      "targets": [
        "backend",
        "frontend"
      ]
    }
  },
  "target": {
    "backend": {
      "context": ".",
      "dockerfile": "Dockerfile",
      "args": {
        "PORT": "7070"
      }
    },
    "frontend": {
      "context": ".",
      "dockerfile": "Dockerfile",
      "args": {
        "PORT": "7071"
      }
    }
  }
}
```
