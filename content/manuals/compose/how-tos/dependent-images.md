---
description: 为具有共享定义的服务构建镜像
keywords: compose, build
title: 构建依赖镜像
weight: 50
---

{{< summary-bar feature_name="Compose dependent images" >}}

为缩短推送/拉取时间并减小镜像体积，Compose 应用的常见做法是尽量让各服务共享基础层。通常会为所有服务选择相同的操作系统基础镜像；当多个镜像需要相同的系统包时，还可以进一步共享镜像层。接下来要解决的问题是：如何避免在每个服务中重复编写完全相同的 Dockerfile 指令。

为便于说明，本文假设你希望所有服务都基于 `alpine` 作为基础镜像，并安装系统包 `openssl`。

## 多阶段 Dockerfile

推荐做法是将共享的声明集中到一个 Dockerfile 中，并利用多阶段构建特性，让各服务的镜像都从这段共享声明开始构建。

Dockerfile:

```dockerfile
FROM alpine as base
RUN /bin/sh -c apk add --update --no-cache openssl

FROM base as service_a
# 构建服务 a
...

FROM base as service_b
# 构建服务 b
...
```

Compose 文件：

```yaml
services:
  a:
     build:
       target: service_a
  b:
     build:
       target: service_b
```

## 将其他服务的镜像用作基础镜像

一个常见模式是在某个服务中复用另一个服务的镜像作为基础镜像。由于 Compose 不解析 Dockerfile，它无法自动识别这种服务间依赖关系，也就无法据此自动安排正确的构建顺序。

a.Dockerfile:

```dockerfile
FROM alpine
RUN /bin/sh -c apk add --update --no-cache openssl
```

b.Dockerfile:

```dockerfile
FROM service_a
# 构建服务 b
```

Compose 文件：

```yaml
services:
  a:
     image: service_a 
     build:
       dockerfile: a.Dockerfile
  b:
     image: service_b
     build:
       dockerfile: b.Dockerfile
```

早期的 Docker Compose v1 会按顺序构建镜像，因此这种模式可以开箱即用。Compose v2 使用 BuildKit 优化构建、并行构建镜像，因此需要显式声明依赖。

推荐做法是将依赖的基础镜像声明为额外的构建上下文：

Compose 文件：

```yaml
services:
  a:
     image: service_a
     build: 
       dockerfile: a.Dockerfile
  b:
     image: service_b
     build:
       dockerfile: b.Dockerfile
       additional_contexts:
         # `FROM service_a` 将被解析为对服务 "a" 的依赖，必须先构建服务 a
         service_a: "service:a"
```

通过 `additional_contexts` 属性，你可以在不显式命名镜像的情况下引用由其他服务构建的镜像：

b.Dockerfile:

```dockerfile

FROM base_image  
# `base_image` 不会解析为实际镜像。它用于指向一个已命名的额外构建上下文

# 构建服务 b
```

Compose 文件：

```yaml
services:
  a:
     build: 
       dockerfile: a.Dockerfile
       # 构建出的镜像将被打标签为 <project_name>_a
  b:
     build:
       dockerfile: b.Dockerfile
       additional_contexts:
         # `FROM base_image` 将被解析为对服务 "a" 的依赖，必须先构建服务 a
         base_image: "service:a"
```

## 使用 Bake 构建

使用 [Bake](/manuals/build/bake/_index.md) 可以一次性传递所有服务的完整构建定义，并以最高效率编排构建流程。

要启用该功能，请在环境中设置 `COMPOSE_BAKE=true` 后运行 Compose：

```console
$ COMPOSE_BAKE=true docker compose build
[+] Building 0.0s (0/1)                                                         
 => [internal] load local bake definitions                                 0.0s
...
[+] Building 2/2 manifest list sha256:4bd2e88a262a02ddef525c381a5bdb08c83  0.0s
 ✔ service_b  Built                                                        0.7s 
 ✔ service_a  Built    
```

Bake 也可以作为默认构建器。你可以通过编辑 `$HOME/.docker/config.json` 配置文件来启用：
```json
{
  ...
  "plugins": {
    "compose": {
      "build": "bake"
    }
  }
  ...
}
```

## 更多资源

- [Docker Compose 构建参考](/reference/cli/docker/compose/build.md)
- [了解多阶段 Dockerfile](/manuals/build/building/multi-stage.md)
