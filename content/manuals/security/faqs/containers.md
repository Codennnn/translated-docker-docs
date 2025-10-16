---
title: 容器安全常见问题
linkTitle: 容器
description: 关于 Docker 容器安全与隔离的常见问题
keywords: 容器安全, Docker Desktop 隔离, 增强容器隔离, 文件共享
weight: 20
tags: [FAQ]
aliases:
- /faq/security/containers/
---

## Docker Desktop 中，容器如何与宿主机隔离？

Docker Desktop 会在一个定制的 Linux 虚拟机中运行所有容器（原生 Windows 容器除外）。即使容器以 root 身份运行，这种方式也能在容器与宿主机之间提供强隔离。

需要重点注意：

- 通过 Docker Desktop 设置共享的宿主机文件，容器可以访问
- 在 Docker Desktop 虚拟机内，容器默认以 root 运行，但其能力受限
- 特权容器（`--privileged`、`--pid=host`、`--cap-add`）在虚拟机内拥有更高权限，可访问虚拟机内部及 Docker Engine

启用增强容器隔离（ECI）后，每个容器都会在 Docker Desktop 虚拟机中的独立 Linux 用户命名空间内运行。即便是特权容器，其权限也仅限于容器边界，而不是虚拟机本身。ECI 通过多种高级技术防止容器突破 Docker Desktop 虚拟机与 Docker Engine 的安全边界。

## 容器可以访问宿主机文件系统的哪些部分？

容器仅能访问以下宿主机文件：

1. 通过 Docker Desktop 设置共享的文件
1. 显式绑定挂载到容器内的文件（例如：`docker run -v /path/to/host/file:/mnt`）

## 以 root 运行的容器能否访问宿主机上管理员拥有的文件？

不能。宿主机文件共享通过用户态文件服务器实现（以 Docker Desktop 用户身份在 `com.docker.backend` 中运行），因此容器只能访问 Docker Desktop 用户本就拥有访问权限的文件。
