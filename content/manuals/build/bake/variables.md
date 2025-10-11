---
title: Bake 中的变量
linkTitle: 变量
weight: 40
description:
keywords: 构建, buildx, bake, buildkit, hcl, 变量
---

你可以在 Bake 文件中定义并使用变量，用于设置属性值、将其插值到其他值中，以及执行算术运算。变量可以带有默认值，也可以通过环境变量进行覆盖。

## 将变量用作属性值

使用 `variable` 块来定义变量。

```hcl {title=docker-bake.hcl}
variable "TAG" {
  default = "docker.io/username/webapp:latest"
}
```

下面的示例展示如何在目标中使用 `TAG` 变量：

```hcl {title=docker-bake.hcl}
target "webapp" {
  context = "."
  dockerfile = "Dockerfile"
  tags = [ TAG ]
}
```

## 在值中插入变量

Bake 支持将变量插值到字符串值中。你可以使用 `${}` 语法把变量插入到值里。以下示例定义了一个值为 `latest` 的 `TAG` 变量。

```hcl {title=docker-bake.hcl}
variable "TAG" {
  default = "latest"
}
```

要把 `TAG` 变量插入到某个属性的值中，使用 `${TAG}` 语法。

```hcl {title=docker-bake.hcl}
group "default" {
  targets = [ "webapp" ]
}

variable "TAG" {
  default = "latest"
}

target "webapp" {
  context = "."
  dockerfile = "Dockerfile"
  tags = ["docker.io/username/webapp:${TAG}"]
}
```

使用 `--print` 标志打印 Bake 文件时，解析后的构建配置会展示插值后的结果。

```console
$ docker buildx bake --print
```

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
      "tags": ["docker.io/username/webapp:latest"]
    }
  }
}
```

## 校验变量

为验证变量值是否符合预期的类型、取值范围或其他条件，你可以使用 `validation` 块定义自定义校验规则。

在下面的示例中，校验用于对变量值施加数值约束：`PORT` 必须大于等于 1024。

```hcl {title=docker-bake.hcl}
# 定义变量 `PORT`，包含默认值与校验规则
variable "PORT" {
  default = 3000  # 为 `PORT` 指定默认值

  # 校验块：确保 `PORT` 为可接受范围内的有效数字
  validation {
    condition = PORT >= 1024  # 确保 `PORT` 至少为 1024
    error_message = "The variable 'PORT' must be 1024 or greater."  # 非法取值时的错误信息
  }
}
```

如果 `condition` 表达式求值为 `false`，则变量值视为无效，构建会失败并输出 `error_message`。例如，若 `PORT=443`，条件为 `false`，将抛出错误。

在设置校验之前，值会被强制转换为期望的类型，从而确保通过环境变量进行覆盖时能够按预期工作。

### 校验多个条件

若需要评估多个条件，可为同一变量定义多个 `validation` 块。所有条件都必须为 `true`。

示例：

```hcl {title=docker-bake.hcl}
# 定义变量 `VAR`，包含多条校验规则
variable "VAR" {
  # 第一条校验：确保变量非空
  validation {
    condition = VAR != ""
    error_message = "The variable 'VAR' must not be empty."
  }

  # 第二条校验：确保值只包含字母和数字
  validation {
    # VAR 与正则匹配结果必须完全一致：
    condition = VAR == regex("[a-zA-Z0-9]+", VAR)
    error_message = "The variable 'VAR' can only contain letters and numbers."
  }
}
```

本示例强制如下约束：

- 变量不能为空。
- 变量必须匹配特定字符集。

对于 `VAR="hello@world"` 这类无效输入，校验将失败。

### 校验变量依赖

你可以在条件表达式中引用其他 Bake 变量，从而对变量之间的依赖关系进行校验，确保依赖变量在继续之前已正确设置。

示例：

```hcl {title=docker-bake.hcl}
# 定义变量 `FOO`
variable "FOO" {}

# 定义变量 `BAR`，其校验规则依赖 `FOO`
variable "BAR" {
  # 校验块：若使用 `BAR`，必须已设置 `FOO`
  validation {
    condition = FOO != ""  # 检查 `FOO` 非空字符串
    error_message = "The variable 'BAR' requires 'FOO' to be set."
  }
}
```

该配置确保只有当 `FOO` 被赋予非空值时才可使用 `BAR`。如果未设置 `FOO` 就尝试构建，将触发校验错误。

## 取消变量插值

如果希望在解析 Bake 定义时跳过变量插值，请使用双美元符号（`$${VARIABLE}`）。

```hcl {title=docker-bake.hcl}
target "webapp" {
  dockerfile-inline = <<EOF
  FROM alpine
  ARG TARGETARCH
  RUN echo "Building for $${TARGETARCH/amd64/x64}"
  EOF
  platforms = ["linux/amd64", "linux/arm64"]
}
```

```console
$ docker buildx bake --progress=plain
...
#8 [linux/arm64 2/2] RUN echo "Building for arm64"
#8 0.036 Building for arm64
#8 DONE 0.0s

#9 [linux/amd64 2/2] RUN echo "Building for x64"
#9 0.046 Building for x64
#9 DONE 0.1s
...
```

## 在多文件间使用变量

当指定了多个文件时，一个文件可以使用另一个文件中定义的变量。如下例所示，`vars.hcl` 定义了一个默认值为 `docker.io/library/alpine` 的 `BASE_IMAGE` 变量：

```hcl {title=vars.hcl}
variable "BASE_IMAGE" {
  default = "docker.io/library/alpine"
}
```

下面的 `docker-bake.hcl` 定义了 `BASE_LATEST` 变量，它引用了 `BASE_IMAGE`：

```hcl {title=docker-bake.hcl}
variable "BASE_LATEST" {
  default = "${BASE_IMAGE}:latest"
}

target "webapp" {
  contexts = {
    base = BASE_LATEST
  }
}
```

当你使用 `-f` 指定 `vars.hcl` 与 `docker-bake.hcl` 并打印解析后的构建配置时，可以看到 `BASE_LATEST` 被解析为 `docker.io/library/alpine:latest`。

```console
$ docker buildx bake -f vars.hcl -f docker-bake.hcl --print app
```

```json
{
  "target": {
    "webapp": {
      "context": ".",
      "contexts": {
        "base": "docker.io/library/alpine:latest"
      },
      "dockerfile": "Dockerfile"
    }
  }
}
```

## 更多资源

以下资源展示了如何在 Bake 中使用变量：

- 可以使用环境变量覆盖 `variable` 的值。参见[覆盖配置](./overrides.md#environment-variables)。
- 可以在函数中引用和使用全局变量。参见 [HCL 函数](./funcs.md#variables-in-functions)。
- 在计算表达式时可以使用变量值。参见[表达式求值](./expressions.md#expressions-with-variables)。
