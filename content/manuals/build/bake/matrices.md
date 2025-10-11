---
title: 矩阵目标
weight: 70
description: 了解如何在 Bake 中定义与使用矩阵目标，将单个目标派生为多个不同变体
keywords: build, buildx, bake, buildkit, matrix, hcl, json
---

矩阵策略允许你根据所指定的参数，将单个目标派生为多个不同的变体。这与 [GitHub Actions 的矩阵策略](https://docs.github.com/en/actions/using-jobs/using-a-matrix-for-your-jobs)类似。使用矩阵可以减少 Bake 定义中的重复。

`matrix` 属性是“参数名 → 值列表”的映射。Bake 会针对值的每一种组合生成并构建一个独立目标。

每个生成的目标都必须具有唯一名称。可通过 `name` 属性指定目标名称的生成方式。

下面的示例把 `app` 目标解析为 `app-foo` 与 `app-bar`，并使用矩阵值定义[目标构建阶段](/build/bake/reference/#targettarget)。

```hcl {title=docker-bake.hcl}
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

## 多维轴

你可以在矩阵中指定多个键，从多个维度派生目标。使用多个矩阵键时，Bake 会构建每一种可能的变体。

下面的示例会构建四个目标：

- `app-foo-1-0`
- `app-foo-2-0`
- `app-bar-1-0`
- `app-bar-2-0`

```hcl {title=docker-bake.hcl}
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

## 每个矩阵条目的多字段值

如果希望不止基于单个值来区分矩阵条目，可以将映射（map）作为矩阵的值。Bake 会为每个映射生成一个目标，并可通过点号访问嵌套字段。

下面的示例会构建两个目标：

- `app-foo-1-0`
- `app-bar-2-0`

```hcl {title=docker-bake.hcl}
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
