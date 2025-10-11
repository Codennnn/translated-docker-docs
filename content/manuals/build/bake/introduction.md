---
title: Bake 简介
linkTitle: 简介
weight: 10
description: 使用 Bake 构建你的项目入门
keywords: bake, 快速开始, 构建, 项目, 简介, 入门
---

Bake 是对 `docker build` 命令的一层抽象，能以更简单且一致的方式为整个团队管理构建配置（CLI 参数、环境变量等）。

Bake 内置于 Buildx CLI。只要安装了 Buildx，就可以通过 `docker buildx bake` 使用 Bake。

## 使用 Bake 构建项目

下面是一个简单的 `docker build` 示例：

```console
$ docker build -f Dockerfile -t myapp:latest .
```

该命令会构建当前目录下的 Dockerfile，并将生成的镜像标记为 `myapp:latest`。

用 Bake 表达同样的构建配置：

```hcl {title=docker-bake.hcl}
target "myapp" {
  context = "."
  dockerfile = "Dockerfile"
  tags = ["myapp:latest"]
}
```

Bake 以结构化方式管理你的构建配置，省去每次记忆 `docker build` 所有 CLI 参数的麻烦。基于这个文件，构建镜像只需运行：

```console
$ docker buildx bake myapp
```

对于简单的构建，`docker build` 与 `docker buildx bake` 的差异并不大。但当构建配置变得复杂时，仅依赖 `docker build` 的 CLI 参数会很难维护；此时 Bake 提供了更结构化的方式来管理复杂性。此外，它还便于在团队内共享构建配置，确保所有人都以一致的配置构建镜像。

## Bake 文件格式

Bake 文件可以使用 HCL、YAML（Docker Compose 文件）或 JSON 编写。总体而言，HCL 表达力更强、灵活性更高，因此本文档与许多使用 Bake 的项目中大多采用 HCL 示例。

目标（target）可设置的属性与 `docker build` 的 CLI 参数高度对应。例如，下面这条 `docker build` 命令：

```console
$ docker build \
  -f Dockerfile \
  -t myapp:latest \
  --build-arg foo=bar \
  --no-cache \
  --platform linux/amd64,linux/arm64 \
  .
```

其在 Bake 中的等价写法：

```hcl {title=docker-bake.hcl}
target "myapp" {
  context = "."
  dockerfile = "Dockerfile"
  tags = ["myapp:latest"]
  args = {
    foo = "bar"
  }
  no-cache = true
  platforms = ["linux/amd64", "linux/arm64"]
}
```

> [!TIP]
>
> 想在 VS Code 中获得更好的 Bake 文件编辑体验？
> 试试 [Docker VS Code 扩展（Beta）](https://marketplace.visualstudio.com/items?itemName=docker.docker)，提供语法检查、代码导航与漏洞扫描。

## 后续步骤

继续了解 Bake 的用法：

- 了解如何在 Bake 中定义并使用 [目标（target）](./targets.md)
- 查看目标可配置的全部属性，请参考
  [Bake 文件参考](/build/bake/reference/)。
