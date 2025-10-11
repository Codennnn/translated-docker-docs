---
title: 多阶段构建
linkTitle: 多阶段
weight: 10
description: |
  了解多阶段构建的概念与用法，帮助优化构建流程并生成更小的镜像
keywords: build, best practices
aliases:
- /engine/userguide/eng-image/multistage-build/
- /develop/develop-images/multistage-build/
---

如果你正在努力优化 Dockerfile，同时又希望保持其可读性与可维护性，那么多阶段构建（multi-stage builds）将非常有帮助。

## 使用多阶段构建

在多阶段构建中，你可以在 Dockerfile 中使用多个 `FROM` 指令。
每个 `FROM` 指令都可以使用不同的基础镜像，并从该指令开始一个新的构建阶段。
你可以在阶段之间有选择地复制构建产物，只将需要的内容带入下一个阶段，把不需要的内容留在前面的阶段，避免进入最终镜像。

下面这个 Dockerfile 包含两个独立阶段：第一阶段用于编译生成二进制文件；第二阶段只从第一阶段复制该二进制文件。

```dockerfile
# syntax=docker/dockerfile:1
FROM golang:{{% param "example_go_version" %}}
WORKDIR /src
COPY <<EOF ./main.go
package main

import "fmt"

func main() {
  fmt.Println("hello, world")
}
EOF
RUN go build -o /bin/hello ./main.go

FROM scratch
COPY --from=0 /bin/hello /bin/hello
CMD ["/bin/hello"]
```

只需一个 Dockerfile 即可，无需额外的构建脚本。直接运行 `docker build` 即可。

```console
$ docker build -t hello .
```

最终你会得到一个极小的生产镜像，其中只包含二进制文件本身。
用于构建应用的工具链不会进入最终镜像。

它是如何做到的？第二个 `FROM` 指令以 `scratch` 作为基础镜像开启了一个新的构建阶段。
`COPY --from=0` 只从前一阶段复制已构建好的产物。Go SDK 以及其他中间产物都留在前一阶段，不会被保存到最终镜像中。

## 为构建阶段命名

默认情况下，阶段没有名称，你需要用序号来引用它们（第一个 `FROM` 的阶段序号为 0）。
你也可以通过在 `FROM` 指令后添加 `AS <NAME>` 来为阶段命名。
下面的示例在前文基础上为阶段命名，并在 `COPY` 指令中使用该名称。
这样即使将来你调整了 Dockerfile 中指令的顺序，`COPY` 也不会出错。

```dockerfile
# syntax=docker/dockerfile:1
FROM golang:{{% param "example_go_version" %}} AS build
WORKDIR /src
COPY <<EOF /src/main.go
package main

import "fmt"

func main() {
  fmt.Println("hello, world")
}
EOF
RUN go build -o /bin/hello ./main.go

FROM scratch
COPY --from=build /bin/hello /bin/hello
CMD ["/bin/hello"]
```

## 在指定阶段停止构建

构建镜像时，并不一定要执行 Dockerfile 中的所有阶段。你可以指定一个目标阶段（target）。
下面的命令基于上一个示例的 `Dockerfile`，但在名为 `build` 的阶段处停止：

```console
$ docker build --target build -t hello .
```

这种方式适用于以下场景：

- 调试某个特定构建阶段
- 使用包含调试符号或工具的 `debug` 阶段，同时保留一个精简的 `production` 阶段
- 使用 `testing` 阶段向应用注入测试数据，但在生产构建时改用使用真实数据的其他阶段

## 将外部镜像用作阶段

在多阶段构建中，你不仅可以从本 Dockerfile 中较早的阶段复制文件，还可以通过 `COPY --from` 从独立的镜像复制。
可以使用本地镜像名、本地或仓库（registry）可用的标签，或镜像的摘要（ID）。
如有需要，Docker 客户端会先拉取该镜像，然后从中复制相应的产物。语法如下：

```dockerfile
COPY --from=nginx:latest /etc/nginx/nginx.conf /nginx.conf
```

## 复用已有阶段作为新阶段

在使用 `FROM` 指令时，你可以引用之前的阶段，从该阶段继续构建。例如：

```dockerfile
# syntax=docker/dockerfile:1

FROM alpine:latest AS builder
RUN apk --no-cache add build-base

FROM builder AS build1
COPY source1.cpp source.cpp
RUN g++ -o /binary source.cpp

FROM builder AS build2
COPY source2.cpp source.cpp
RUN g++ -o /binary source.cpp
```

## 传统构建器与 BuildKit 的差异

传统的 Docker Engine 构建器会处理 Dockerfile 中直到所选 `--target` 的所有阶段。
即使目标阶段并不依赖某个阶段，也会把它构建出来。

[BuildKit](../buildkit/_index.md) 只会构建目标阶段所依赖的那些阶段。

例如，给定如下 Dockerfile：

```dockerfile
# syntax=docker/dockerfile:1
FROM ubuntu AS base
RUN echo "base"

FROM base AS stage1
RUN echo "stage1"

FROM base AS stage2
RUN echo "stage2"
```

在[启用 BuildKit](../buildkit/_index.md#getting-started) 的情况下，构建该 Dockerfile 的 `stage2` 目标时，只有 `base` 与 `stage2` 会被处理。
由于 `stage2` 不依赖 `stage1`，因此 `stage1` 会被跳过。

```console
$ DOCKER_BUILDKIT=1 docker build --no-cache -f Dockerfile --target stage2 .
[+] Building 0.4s (7/7) FINISHED                                                                    
 => [internal] load build definition from Dockerfile                                            0.0s
 => => transferring dockerfile: 36B                                                             0.0s
 => [internal] load .dockerignore                                                               0.0s
 => => transferring context: 2B                                                                 0.0s
 => [internal] load metadata for docker.io/library/ubuntu:latest                                0.0s
 => CACHED [base 1/2] FROM docker.io/library/ubuntu                                             0.0s
 => [base 2/2] RUN echo "base"                                                                  0.1s
 => [stage2 1/1] RUN echo "stage2"                                                              0.2s
 => exporting to image                                                                          0.0s
 => => exporting layers                                                                         0.0s
 => => writing image sha256:f55003b607cef37614f607f0728e6fd4d113a4bf7ef12210da338c716f2cfd15    0.0s
```

相反，如果未启用 BuildKit，构建同一个目标将会处理所有阶段：

```console
$ DOCKER_BUILDKIT=0 docker build --no-cache -f Dockerfile --target stage2 .
Sending build context to Docker daemon  219.1kB
Step 1/6 : FROM ubuntu AS base
 ---> a7870fd478f4
Step 2/6 : RUN echo "base"
 ---> Running in e850d0e42eca
base
Removing intermediate container e850d0e42eca
 ---> d9f69f23cac8
Step 3/6 : FROM base AS stage1
 ---> d9f69f23cac8
Step 4/6 : RUN echo "stage1"
 ---> Running in 758ba6c1a9a3
stage1
Removing intermediate container 758ba6c1a9a3
 ---> 396baa55b8c3
Step 5/6 : FROM base AS stage2
 ---> d9f69f23cac8
Step 6/6 : RUN echo "stage2"
 ---> Running in bbc025b93175
stage2
Removing intermediate container bbc025b93175
 ---> 09fc3770a9c4
Successfully built 09fc3770a9c4
```

传统构建器仍会处理 `stage1`，即使 `stage2` 并不依赖它。
