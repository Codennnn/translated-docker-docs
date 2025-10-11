---
title: Bake 目标
linkTitle: 目标
weight: 20
description: 了解如何在 Bake 中定义与使用目标
keywords: bake, target, targets, buildx, docker, buildkit, default
---

在 Bake 文件中，目标（target）表示一次构建调用。它承载了你通常通过 `docker build` 命令行参数（flags）传入的全部配置信息。

```hcl {title=docker-bake.hcl}
target "webapp" {
  dockerfile = "webapp.Dockerfile"
  tags = ["docker.io/username/webapp:latest"]
  context = "https://github.com/username/webapp"
}
```

使用 Bake 构建某个目标时，将目标名称传给 `bake` 命令即可：

```console
$ docker buildx bake webapp
```

你也可以一次性构建多个目标，只需传入多个目标名称：

```console
$ docker buildx bake webapp api tests
```

## 默认目标

当你未在 `docker buildx bake` 中显式指定目标时，Bake 会构建名为 `default` 的目标。

```hcl {title=docker-bake.hcl}
target "default" {
  dockerfile = "webapp.Dockerfile"
  tags = ["docker.io/username/webapp:latest"]
  context = "https://github.com/username/webapp"
}
```

要构建该默认目标，直接运行不带任何参数的 `docker buildx bake`：

```console
$ docker buildx bake
```

## 目标可配置项

目标可设置的属性与 `docker build` 的 CLI 标志非常相近，同时还包含少数 Bake 专属属性。

目标可设置属性的完整列表见 [Bake 参考](/build/bake/reference#target)。

## 目标分组

你可以使用 `group` 块将多个目标分组，便于一次性构建：

```hcl {title=docker-bake.hcl}
group "all" {
  targets = ["webapp", "api", "tests"]
}

target "webapp" {
  dockerfile = "webapp.Dockerfile"
  tags = ["docker.io/username/webapp:latest"]
  context = "https://github.com/username/webapp"
}

target "api" {
  dockerfile = "api.Dockerfile"
  tags = ["docker.io/username/api:latest"]
  context = "https://github.com/username/api"
}

target "tests" {
  dockerfile = "tests.Dockerfile"
  contexts = {
    webapp = "target:webapp"
    api = "target:api"
  }
  output = ["type=local,dest=build/tests"]
  context = "."
}
```

要构建某个分组中的全部目标，把该分组名传给 `bake` 命令：

```console
$ docker buildx bake all
```

## 目标与分组的模式匹配

Bake 在指定目标或目标分组时支持类 shell 的通配符，使你无需逐一列出每个目标：

支持的模式：

- `*`：匹配任意长度的任意字符序列
- `?`：匹配任意单个字符
- `[abc]`：匹配方括号内给定集合中的任意一个字符

> [!NOTE]
>
> 请始终为通配符模式加上引号。若不加引号，shell 会先在当前目录对通配符做文件名展开，通常会导致错误。

示例： 

```console
# 匹配所有以 'foo-' 开头的目标
$ docker buildx bake "foo-*"

# 匹配所有目标
$ docker buildx bake "*"

# 匹配：foo-baz、foo-caz、foo-daz 等
$ docker buildx bake "foo-?az"

# 匹配：foo-bar、boo-bar
$ docker buildx bake "[fb]oo-bar"

# 匹配：mtx-a-b-d、mtx-a-b-e、mtx-a-b-f
$ docker buildx bake "mtx-a-b-*"
``` 

你也可以组合多个模式：

```console
$ docker buildx bake "foo*" "tests"
```

## 更多资源

想进一步了解 Bake 的能力，请参阅：

- 在 Bake 中使用[变量](./variables.md)，让构建配置更灵活。
- 在 [矩阵](./matrices.md)中了解如何用矩阵一次构建多组不同配置的镜像。
- 前往 [Bake 文件参考](/build/bake/reference/)，查看 Bake 文件支持的全部属性及语法。
