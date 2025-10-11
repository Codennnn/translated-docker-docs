---
title: 在 Bake 中使用表达式
linkTitle: 表达式
weight: 50
description: 了解 Bake 的高级特性，如自定义函数
keywords: build, buildx, bake, buildkit, hcl, expressions, evaluation, math, arithmetic, conditionals
aliases:
  - /build/bake/advanced/
---

HCL 格式的 Bake 文件支持表达式求值，可用于执行算术运算、按条件设置值等。

## 算术运算

表达式中可以进行算术运算。下面的示例演示两个数相乘：

```hcl {title=docker-bake.hcl}
sum = 7*6

target "default" {
  args = {
    answer = sum
  }
}
```

使用 `--print` 打印 Bake 文件，可看到构建参数 `answer` 的求值结果：

```console
$ docker buildx bake --print
```

```json
{
  "target": {
    "default": {
      "context": ".",
      "dockerfile": "Dockerfile",
      "args": {
        "answer": "42"
      }
    }
  }
}
```

## 三元运算符

可以使用三元运算符按条件设置值。

下面的示例使用内置的 `notequal`[函数](./funcs.md)，仅当变量非空时才追加一个标签：

```hcl {title=docker-bake.hcl}
variable "TAG" {}

target "default" {
  context="."
  dockerfile="Dockerfile"
  tags = [
    "my-image:latest",
    notequal("",TAG) ? "my-image:${TAG}": ""
  ]
}
```

在该示例中，`TAG` 为空字符串，因此最终构建配置仅包含硬编码的 `my-image:latest` 标签。

```console
$ docker buildx bake --print
```

```json
{
  "target": {
    "default": {
      "context": ".",
      "dockerfile": "Dockerfile",
      "tags": ["my-image:latest"]
    }
  }
}
```

## 搭配变量的表达式

表达式可以与[变量](./variables.md)一起使用，以条件化地设置值，或进行算术运算。

下面的示例根据变量的取值来设置参数：当变量 `FOO` 大于 5 时，将构建参数 `v1` 设为 "higher"，否则设为 "lower"；当变量 `IS_FOO` 为 true 时，将构建参数 `v2` 设为 "yes"，否则设为 "no"。

```hcl {title=docker-bake.hcl}
variable "FOO" {
  default = 3
}

variable "IS_FOO" {
  default = true
}

target "app" {
  args = {
    v1 = FOO > 5 ? "higher" : "lower"
    v2 = IS_FOO ? "yes" : "no"
  }
}
```

使用 `--print` 打印该目标，可看到 `v1` 与 `v2` 两个构建参数的求值结果：

```console
$ docker buildx bake --print app
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
        "v1": "lower",
        "v2": "yes"
      }
    }
  }
}
```
