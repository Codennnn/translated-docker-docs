---
title: containerd 镜像存储
weight: 80
description: 如何在 Docker Desktop 中启用 containerd 集成功能
keywords: Docker, containerd, engine, image store, lazy-pull
toc_max: 3
aliases:
- /desktop/containerd/
---

Docker Desktop 正在过渡为使用 containerd 管理镜像与文件系统。本文将介绍 containerd 镜像存储的优势、启用步骤，以及由其带来的新能力。

> [!NOTE]
> 
> Docker Desktop 为“经典镜像存储”和“containerd 镜像存储”分别维护独立的镜像存储。
> 在两者之间切换时，未激活存储中的镜像与容器仍会保留在磁盘上，但会被隐藏，直到你切换回去。

## 什么是 `containerd`？

`containerd` 是一个容器运行时，为容器生命周期管理提供轻量且一致的接口。Docker Engine 在底层已经使用它来创建、启动与停止容器。

Docker Desktop 对 containerd 的集成正在持续推进，并扩展到了镜像存储层，从而提供更高的灵活性与更现代的镜像类型支持。

## 什么是 `containerd` 镜像存储？

镜像存储负责在文件系统中推送、拉取并保存镜像。

经典的 Docker 镜像存储在支持的镜像类型上存在限制。
例如，它不支持包含清单列表（manifest list）的镜像索引（image index）。
当你创建多平台镜像时，镜像索引会汇总并解析该镜像在不同平台上的各个变体。
在构建带有声明（attestations）的镜像时，也需要镜像索引的支持。

containerd 镜像存储扩展了 Docker Engine 可原生交互的镜像类型范围。
虽然这是一项底层架构层面的变更，但它是解锁一系列新用例的前提条件，包括：

- [构建多平台镜像](#build-multi-platform-images)以及带有声明（attestations）的镜像
- 支持使用具有独特特性的 containerd 快照器（snapshotter），
  例如用于在容器启动时懒加载拉取镜像的 [stargz][1]，
  或用于点对点镜像分发的 [nydus][2] 与 [dragonfly][3]
- 运行 [Wasm](wasm.md) 容器的能力

[1]: https://github.com/containerd/stargz-snapshotter
[2]: https://github.com/containerd/nydus-snapshotter
[3]: https://github.com/dragonflyoss/image-service

## 启用 containerd 镜像存储

自 Docker Desktop 4.34 起，默认启用 containerd 镜像存储，但仅适用于全新安装或执行过“恢复出厂设置”的环境。
如果你是从更早版本升级而来，或仍在使用更旧版本，则需要手动切换到 containerd 镜像存储。

在 Docker Desktop 中手动启用该功能：

1. 打开 Docker Desktop 的 **Settings**。
2. 在 **General** 选项卡中，勾选 **Use containerd for pulling and storing images**。
3. 选择 **Apply**。

如需禁用 containerd 镜像存储，取消勾选 **Use containerd for pulling and storing images**。

## 构建多平台镜像

“多平台镜像”指的是面向多种不同架构的一组镜像。
开箱即用的默认构建器并不支持构建多平台镜像。

```console
$ docker build --platform=linux/amd64,linux/arm64 .
[+] Building 0.0s (0/0)
ERROR: Multi-platform build is not supported for the docker driver.
Switch to a different driver, or turn on the containerd image store, and try again.
Learn more at https://docs.docker.com/go/build-multi-platform/
```

启用 containerd 镜像存储后，你可以构建多平台镜像，并将其加载到本地镜像存储：

<script async id="asciicast-ZSUI4Mi2foChLjbevl2dxt5GD" src="https://asciinema.org/a/ZSUI4Mi2foChLjbevl2dxt5GD.js"></script>


