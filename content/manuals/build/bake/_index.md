---
title: Bake
weight: 50
keywords: build, buildx, bake, buildkit, hcl, json, compose
aliases:
  - /build/customize/bake/
---

Bake 是 Docker Buildx 的一项功能，你可以用声明式文件来定义构建配置，
而不必编写复杂的 CLI 命令表达式。它还支持在一次调用中并发运行多个构建。

Bake 文件可以使用 HCL、JSON 或 YAML 格式编写；其中 YAML 是对 Docker Compose 文件的扩展。
下面是一个 HCL 格式的 Bake 文件示例：

```hcl {title=docker-bake.hcl}
group "default" {
  targets = ["frontend", "backend"]
}

target "frontend" {
  context = "./frontend"
  dockerfile = "frontend.Dockerfile"
  args = {
    NODE_VERSION = "22"
  }
  tags = ["myapp/frontend:latest"]
}

target "backend" {
  context = "./backend"
  dockerfile = "backend.Dockerfile"
  args = {
    GO_VERSION = "{{% param "example_go_version" %}}"
  }
  tags = ["myapp/backend:latest"]
}
```

`group` 块定义了一组可以并发构建的目标。
每个 `target` 块都定义了一个具有独立配置的构建目标，例如构建上下文、Dockerfile 与标签。

基于上述 Bake 文件进行构建，你可以运行：

```console
$ docker buildx bake
```

该命令会执行名为 `default` 的分组，从而并发构建 `frontend` 与 `backend` 两个目标。

## 入门

想要快速上手 Bake，请参阅[Bake 简介](./introduction.md)。
