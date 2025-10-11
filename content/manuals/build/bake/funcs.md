---
title: 函数
weight: 60
description: 了解 Bake 中内置与自定义的 HCL 函数
keywords: 构建, buildx, bake, buildkit, hcl, 函数, 自定义, 内置, 自定义函数, gocty
aliases:
  - /build/customize/bake/hcl-funcs/
  - /build/bake/hcl-funcs/
---

当你需要以比简单拼接或插值更复杂的方式处理构建配置中的值时，HCL 函数非常有用。

## 标准库

Bake 内置支持[标准库函数](/manuals/build/bake/stdlib.md)。

下面示例展示 `add` 函数：

```hcl {title=docker-bake.hcl}
variable "TAG" {
  default = "latest"
}

group "default" {
  targets = ["webapp"]
}

target "webapp" {
  args = {
    buildno = "${add(123, 1)}"
  }
}
```

```console
$ docker buildx bake --print webapp
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
      "args": {
        "buildno": "124"
      }
    }
  }
}
```

## 自定义函数

当内置标准库函数无法满足需求时，你可以创建[自定义函数](https://github.com/hashicorp/hcl/tree/main/ext/userfunc)来精确完成所需逻辑。

下面示例定义一个 `increment` 函数。

```hcl {title=docker-bake.hcl}
function "increment" {
  params = [number]
  result = number + 1
}

group "default" {
  targets = ["webapp"]
}

target "webapp" {
  args = {
    buildno = "${increment(123)}"
  }
}
```

```console
$ docker buildx bake --print webapp
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
      "args": {
        "buildno": "124"
      }
    }
  }
}
```

## 在函数中使用变量

你可以在函数内部引用[变量](./variables)与标准库函数。

不能在函数中引用其他自定义函数。

下面的示例在自定义函数中使用了一个全局变量（`REPO`）。

```hcl {title=docker-bake.hcl}
# docker-bake.hcl
variable "REPO" {
  default = "user/repo"
}

function "tag" {
  params = [tag]
  result = ["${REPO}:${tag}"]
}

target "webapp" {
  tags = tag("v1")
}
```

使用 `--print` 标志打印时可以看到，`tag` 函数使用 `REPO` 的值来设置标签的前缀。

```console
$ docker buildx bake --print webapp
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
      "tags": ["user/repo:v1"]
    }
  }
}
```
