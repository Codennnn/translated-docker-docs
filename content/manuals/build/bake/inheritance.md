---
title: Bake 中的继承
linkTitle: 继承
weight: 30
description: 了解如何在 Bake 中从其他目标继承属性
keywords: buildx, buildkit, bake, 继承, 目标, 属性
---

目标可以使用 `inherits` 属性从其他目标继承属性。比如，假设你有一个用于开发环境的镜像构建目标：

```hcl {title=docker-bake.hcl}
target "app-dev" {
  args = {
    GO_VERSION = "{{% param example_go_version %}}"
  }
  tags = ["docker.io/username/myapp:dev"]
  labels = {
    "org.opencontainers.image.source" = "https://github.com/username/myapp"
    "org.opencontainers.image.author" = "moby.whale@example.com"
  }
}
```

你可以创建一个新的目标，复用相同的构建配置，但针对生产环境做一些属性调整。在下面的示例中，`app-release` 目标继承了 `app-dev`，同时覆盖了 `tags` 属性并新增 `platforms` 属性：

```hcl {title=docker-bake.hcl}
target "app-release" {
  inherits = ["app-dev"]
  tags = ["docker.io/username/myapp:latest"]
  platforms = ["linux/amd64", "linux/arm64"]
}
```

## 通用可复用目标

一种常见的继承模式是定义一个公共目标，集中放置项目中全部或多数构建目标需要共享的属性。比如，下面的 `_common` 目标定义了一组通用的构建参数：

```hcl {title=docker-bake.hcl}
target "_common" {
  args = {
    GO_VERSION = "{{% param example_go_version %}}"
    BUILDKIT_CONTEXT_KEEP_GIT_DIR = 1
  }
}
```

随后，你可以在其他目标中继承 `_common`，以应用这些共享属性：

```hcl {title=docker-bake.hcl}
target "lint" {
  inherits = ["_common"]
  dockerfile = "./dockerfiles/lint.Dockerfile"
  output = [{ type = "cacheonly" }]
}

target "docs" {
  inherits = ["_common"]
  dockerfile = "./dockerfiles/docs.Dockerfile"
  output = ["./docs/reference"]
}

target "test" {
  inherits = ["_common"]
  target = "test-output"
  output = ["./test"]
}

target "binaries" {
  inherits = ["_common"]
  target = "binaries"
  output = ["./build"]
  platforms = ["local"]
}
```

## 覆盖继承的属性

当一个目标继承了另一个目标时，它可以覆盖任意继承而来的属性。比如，下面的目标覆盖了继承目标中的 `args` 属性：

```hcl {title=docker-bake.hcl}
target "app-dev" {
  inherits = ["_common"]
  args = {
    GO_VERSION = "1.17"
  }
  tags = ["docker.io/username/myapp:dev"]
}
```

在 `app-dev` 中，`GO_VERSION` 被设置为 `1.17`，从而覆盖了 `_common` 目标中的同名参数。

关于覆盖属性的更多信息，参见[覆盖配置](./overrides.md)。

## 从多个目标继承

`inherits` 属性是一个列表，这意味着你可以同时复用多个其他目标的属性。下面的示例中，`app-release` 同时复用了 `app-dev` 与 `_common` 的属性：

```hcl {title=docker-bake.hcl}
target "_common" {
  args = {
    GO_VERSION = "{{% param example_go_version %}}"
    BUILDKIT_CONTEXT_KEEP_GIT_DIR = 1
  }
}

target "app-dev" {
  inherits = ["_common"]
  args = {
    BUILDKIT_CONTEXT_KEEP_GIT_DIR = 0
  }
  tags = ["docker.io/username/myapp:dev"]
  labels = {
    "org.opencontainers.image.source" = "https://github.com/username/myapp"
    "org.opencontainers.image.author" = "moby.whale@example.com"
  }
}

target "app-release" {
  inherits = ["app-dev", "_common"]
  tags = ["docker.io/username/myapp:latest"]
  platforms = ["linux/amd64", "linux/arm64"]
}
```

当从多个目标继承且出现属性冲突时，`inherits` 列表中靠后的目标优先生效。上面的示例在 `_common` 中定义了 `BUILDKIT_CONTEXT_KEEP_GIT_DIR`，并在 `app-dev` 中进行了覆盖。

`app-release` 同时继承了 `app-dev` 与 `_common`。由于 `app-dev` 中该参数为 0，而 `_common` 中为 1，且 `_common` 在 `inherits` 列表中靠后，因此 `app-release` 中该参数最终为 1（而非 0）。

## 复用目标中的单个属性

如果只想从某个目标复用单个属性，可以使用点号语法引用其他目标的属性。例如，下面的 Bake 文件中，`bar` 目标复用了 `foo` 目标的 `tags` 属性：

```hcl {title=docker-bake.hcl}
target "foo" {
  dockerfile = "foo.Dockerfile"
  tags       = ["myapp:latest"]
}
target "bar" {
  dockerfile = "bar.Dockerfile"
  tags       = target.foo.tags
}
```
