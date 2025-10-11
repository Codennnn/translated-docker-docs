---
title: 构建缓存失效
description: 深入了解 Docker 构建缓存的失效机制
keywords: build, buildx, buildkit, cache, invalidation, cache miss
---

在构建镜像时，Docker 会按顺序逐条执行 Dockerfile 中的指令。
对于每条指令，[构建器](/manuals/build/builders/_index.md) 都会检查是否可以复用构建缓存。

## 通用规则

构建缓存失效的基本规则如下：

- 构建器会先检查基础镜像是否已被缓存。之后的每条指令都会与已缓存的层进行比对；
  如果没有完全匹配的缓存层，则判定缓存失效。

- 大多数情况下，将 Dockerfile 指令与对应的缓存层进行比对即可；
  但某些指令需要额外的检查与说明。

- 对于 `ADD`、`COPY` 指令，以及带有绑定挂载的 `RUN` 指令（`RUN --mount=type=bind`），
  构建器会基于文件元数据计算缓存校验和，以判断缓存是否有效。
  在查找缓存时，只要相关文件的元数据发生变化，就会导致缓存失效。

  计算缓存校验和时不会考虑文件的修改时间（`mtime`）。
  如果仅复制文件的 `mtime` 发生变化，缓存不会因此失效。

- 除 `ADD` 与 `COPY` 外，缓存检查不会查看容器内的文件来判断是否命中缓存。
  例如，处理 `RUN apt-get -y update` 时，并不会检查容器内更新的文件来决定是否命中缓存；
  此时仅使用命令字符串本身进行匹配。

一旦某步缓存失效，其后的 Dockerfile 指令都会生成新的镜像层，不再复用缓存。

如果你的构建包含多层，且希望提高缓存复用率，建议尽可能将不常变化的指令放在前面，
将变化频繁的指令放在后面。

## RUN 指令

`RUN` 指令的缓存不会在不同构建之间自动失效。
例如，你在 Dockerfile 中添加了安装 `curl` 的步骤：

```dockerfile
FROM alpine:{{% param "example_alpine_version" %}} AS install
RUN apk add curl
```

这并不意味着镜像中的 `curl` 总是最新版本。即使一周后重新构建，你仍会得到与之前相同的包。
若要强制重新执行该 `RUN` 指令，你可以：

- 确保其之前的某一层发生变化
- 在构建前清理缓存，使用 [`docker builder prune`](/reference/cli/docker/builder/prune.md)
- 使用 `--no-cache` 或 `--no-cache-filter` 选项

`--no-cache-filter` 可用于指定某个构建阶段，使其缓存失效：

```console
$ docker build --no-cache-filter install .
```

## 构建机密（secrets）

构建机密（secret）的内容不参与构建缓存。
仅修改 secret 的值不会导致缓存失效。

如果修改 secret 后希望强制使缓存失效，可以传入一个构建参数（值任意），并在修改 secret 时同步修改该参数；
构建参数的变化会导致缓存失效。

```dockerfile
FROM alpine
ARG CACHEBUST
RUN --mount=type=secret,id=TOKEN,env=TOKEN \
    some-command ...
```

```console
$ TOKEN="tkn_pat123456" docker build --secret id=TOKEN --build-arg CACHEBUST=1 .
```

secret 的属性（如 ID、挂载路径）会参与缓存校验和的计算，改变这些属性会导致缓存失效。
