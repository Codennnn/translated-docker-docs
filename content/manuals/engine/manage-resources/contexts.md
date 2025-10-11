---
title: Docker 上下文
description: 了解如何使用上下文在单个客户端管理多个守护进程
keywords: engine, context, cli, daemons, remote
aliases:
  - /engine/context/working-with-contexts/
---

## 简介

本指南介绍如何使用“上下文”（context）让一个 Docker 客户端同时管理多个 Docker 守护进程。

每个上下文都包含与守护进程上资源交互所需的全部信息。`docker context` 命令可帮助你轻松配置与切换上下文。

例如，一个 Docker 客户端可以同时配置两个上下文：

- 本地运行的默认上下文
- 远程的共享上下文

完成配置后，你可以使用 `docker context use <context-name>` 在它们之间切换。

## 前提条件

要跟随本文示例操作，你需要：

- 支持顶级 `context` 命令的 Docker 客户端

运行 `docker context` 验证你的 Docker 客户端是否支持上下文功能。

## 上下文的组成

一个上下文由多项属性构成，包括：

- 名称与描述
- 端点配置
- TLS 信息

使用 `docker context ls` 列出所有可用上下文。

```console
$ docker context ls
NAME        DESCRIPTION                               DOCKER ENDPOINT               ERROR
default *                                             unix:///var/run/docker.sock
```

上述结果展示了一个名为 "default" 的上下文。它通过本地的 Unix 套接字 `/var/run/docker.sock` 与守护进程通信。

`NAME` 列中的星号表示该上下文处于激活状态。也就是说，所有 `docker` 命令默认都会作用在该上下文上，除非你通过 `DOCKER_HOST`、`DOCKER_CONTEXT` 环境变量，或命令行的 `--context`、`--host` 标志进行覆盖。

你可以使用 `docker context inspect` 深入查看详情。以下示例展示如何检查名为 `default` 的上下文：

```console
$ docker context inspect default
[
    {
        "Name": "default",
        "Metadata": {},
        "Endpoints": {
            "docker": {
                "Host": "unix:///var/run/docker.sock",
                "SkipTLSVerify": false
            }
        },
        "TLSMaterial": {},
        "Storage": {
            "MetadataPath": "\u003cIN MEMORY\u003e",
            "TLSPath": "\u003cIN MEMORY\u003e"
        }
    }
]
```

### 创建新上下文

使用 `docker context create` 可以创建新的上下文。

下面的示例创建一个名为 `docker-test` 的上下文，并将其主机端点设置为 TCP 套接字 `tcp://docker:2375`。

```console
$ docker context create docker-test --docker host=tcp://docker:2375
docker-test
Successfully created context "docker-test"
```

新建的上下文会存储在 `~/.docker/contexts/` 目录下的 `meta.json` 文件中。
你创建的每个上下文都会在 `~/.docker/contexts/` 的独立子目录中拥有自己的 `meta.json` 文件。

可以通过 `docker context ls` 和 `docker context inspect <context-name>` 查看新上下文。

```console
$ docker context ls
NAME          DESCRIPTION                             DOCKER ENDPOINT               ERROR
default *                                             unix:///var/run/docker.sock
docker-test                                           tcp://docker:2375
```

当前正在使用的上下文会以星号（"*"）标识。

## 使用其他上下文

你可以通过 `docker context use` 在上下文之间切换。

以下命令将把 `docker` CLI 切换为使用 `docker-test` 上下文：

```console
$ docker context use docker-test
docker-test
Current context is now "docker-test"
```

再次列出上下文，确认星号（"*"）已显示在 `docker-test` 上下文上：

```console
$ docker context ls
NAME            DESCRIPTION                           DOCKER ENDPOINT               ERROR
default                                               unix:///var/run/docker.sock
docker-test *                                         tcp://docker:2375
```

现在，`docker` 命令会指向 `docker-test` 上下文中定义的端点。

也可以通过 `DOCKER_CONTEXT` 环境变量设置当前上下文。该变量会覆盖通过 `docker context use` 设置的值。

根据你的环境，使用下面合适的命令把上下文设置为 `docker-test`：

{{< tabs >}}
{{< tab name="PowerShell" >}}

```ps
> $env:DOCKER_CONTEXT='docker-test'
```

{{< /tab >}}
{{< tab name="Bash" >}}

```console
$ export DOCKER_CONTEXT=docker-test
```

{{< /tab >}}
{{< /tabs >}}

运行 `docker context ls` 验证 `docker-test` 已成为当前激活的上下文。

你也可以使用全局 `--context` 标志临时覆盖上下文。以下命令使用名为 `production` 的上下文：

```console
$ docker --context production container ls
```

## 导出与导入 Docker 上下文

你可以使用 `docker context export` 与 `docker context import` 在不同主机之间导出/导入上下文。

`docker context export` 会把现有上下文导出为一个文件；任何安装了 `docker` 客户端的主机都可以导入它。

### 导出与导入示例

下面示例导出名为 `docker-test` 的上下文，写入到 `docker-test.dockercontext` 文件：

```console
$ docker context export docker-test
Written file "docker-test.dockercontext"
```

你也可以查看导出文件的内容：

```console
$ cat docker-test.dockercontext
```

在另一台主机上使用 `docker context import` 导入该文件，即可创建相同配置的上下文：

```console
$ docker context import docker-test docker-test.dockercontext
docker-test
Successfully imported context "docker-test"
```

可通过 `docker context ls` 验证导入是否成功。

导入命令格式：`docker context import <context-name> <context-file>`。

## 更新上下文

使用 `docker context update` 可以更新已有上下文中的字段。

以下示例把已存在的 `docker-test` 上下文的描述字段更新为 "Test context"：

```console
$ docker context update docker-test --description "Test context"
docker-test
Successfully updated context "docker-test"
```
