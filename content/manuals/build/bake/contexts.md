---
title: 在 Bake 中使用额外上下文
linkTitle: 上下文
weight: 80
description: |
  当你需要固定镜像版本，
  或引用其他目标的输出时，额外的上下文会很有用
keywords: build, buildx, bake, buildkit, hcl
aliases:
  - /build/customize/bake/build-contexts/
  - /build/bake/build-contexts/
---

除定义构建上下文的主键 `context` 外，每个目标还可以通过 `contexts` 键（映射）定义具名的额外上下文。其取值对应于[构建命令](/reference/cli/docker/buildx/build.md#build-context)中的 `--build-context` 标志。

在 Dockerfile 中，可以在 `FROM` 指令或 `--from` 标志中使用这些上下文。

支持的上下文来源包括：

- 本地文件系统目录
- 容器镜像
- Git URL
- HTTP URL
- Bake 文件中其他目标的名称

## 固定 alpine 镜像

```dockerfile {title=Dockerfile}
# syntax=docker/dockerfile:1
FROM alpine
RUN echo "Hello world"
```

```hcl {title=docker-bake.hcl}
target "app" {
  contexts = {
    alpine = "docker-image://alpine:3.13"
  }
}
```

## 使用第二个源码目录

```dockerfile {title=Dockerfile}
FROM golang
COPY --from=src . .
```

```hcl {title=docker-bake.hcl}
# 运行 `docker buildx bake app` 后，`src` 不会指向之前的某个构建阶段，
# 而是指向客户端文件系统（不属于构建上下文）。
target "app" {
  contexts = {
    src = "../path/to/source"
  }
}
```

## 将目标用作构建上下文

若要将某个目标的构建结果作为另一个目标的构建上下文，请使用带有 `target:` 前缀的目标名称。

```dockerfile {title=baseapp.Dockerfile}
FROM scratch
```
```dockerfile {title=Dockerfile}
# syntax=docker/dockerfile:1
FROM baseapp
RUN echo "Hello world"
```

```hcl {title=docker-bake.hcl}
target "base" {
  dockerfile = "baseapp.Dockerfile"
}

target "app" {
  contexts = {
    baseapp = "target:base"
  }
}
```

在大多数情况下，推荐使用一个包含多个目标的多阶段 Dockerfile 来实现类似行为。仅在存在多个难以合并为单一文件的 Dockerfile 时，才建议采用本节方式。

## 去重上下文传输

> [!NOTE]
>
> 自 Buildx 0.17.0 起，Bake 会自动为共享相同上下文的目标去重上下文传输。除 Buildx 版本要求外，构建器还需运行 BuildKit 0.16.0 或更高版本，且 Dockerfile 语法需为 `docker/dockerfile:1.10` 或更高。
>
> 若已满足上述要求，无需按本节所述手动去重上下文传输。
>
> - 检查 Buildx 版本：运行 `docker buildx version`。
> - 检查 BuildKit 版本：运行 `docker buildx inspect --bootstrap`，查看 `BuildKit version` 字段。
> - 检查 Dockerfile 语法版本：查看 Dockerfile 中的 `syntax`[解析指令](/reference/dockerfile.md#syntax)。若未指定，则使用当前 BuildKit 随附的默认版本。要显式设置，请在 Dockerfile 顶部添加 `#syntax=docker/dockerfile:1.10`。

当你通过分组并发构建目标时，构建上下文会为每个目标独立加载。若同一分组内的多个目标使用了相同上下文，该上下文会被重复传输多次。这可能对构建时长产生显著影响，具体取决于构建配置。例如，假设 Bake 文件定义了如下目标分组：

```hcl {title=docker-bake.hcl}
group "default" {
  targets = ["target1", "target2"]
}

target "target1" {
  target = "target1"
  context = "."
}

target "target2" {
  target = "target2"
  context = "."
}
```

在上述场景中，构建 `default` 分组时，上下文 `.` 会被传输两次：一次用于 `target1`，一次用于 `target2`。

如果构建上下文很小，且使用本地构建器，重复传输的影响可能不大。但当构建上下文较大、目标数量较多，或需要通过网络将上下文传输到远程构建器时，上下文传输会成为性能瓶颈。

为避免多次传输相同上下文，你可以定义一个仅加载上下文文件的具名上下文，并让需要这些文件的每个目标引用它。如下示例通过具名目标 `ctx` 提供上下文，供 `target1` 与 `target2` 复用：

```hcl {title=docker-bake.hcl}
group "default" {
  targets = ["target1", "target2"]
}

target "ctx" {
  context = "."
  target = "ctx"
}

target "target1" {
  target = "target1"
  contexts = {
    ctx = "target:ctx"
  }
}

target "target2" {
  target = "target2"
  contexts = {
    ctx = "target:ctx"
  }
}
```

具名上下文 `ctx` 对应一个 Dockerfile 阶段，它会从其上下文（`.`）复制文件。Dockerfile 中的其他阶段即可引用名为 `ctx` 的上下文，例如通过 `--mount=from=ctx` 挂载其文件。

```dockerfile {title=Dockerfile}
FROM scratch AS ctx
COPY --link . .

FROM golang:alpine AS target1
WORKDIR /work
RUN --mount=from=ctx \
    go build -o /out/client ./cmd/client \

FROM golang:alpine AS target2
WORKDIR /work
RUN --mount=from=ctx \
    go build -o /out/server ./cmd/server
```
