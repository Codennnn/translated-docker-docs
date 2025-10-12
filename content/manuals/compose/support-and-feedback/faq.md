---
description: 回答关于 Docker Compose 的常见问题，包括 v1 与 v2、常用命令、关闭行为与开发环境等。
keywords: docker compose faq, docker compose questions, docker-compose vs docker compose, docker compose json, docker compose stop delay, run multiple docker compose
title: Docker Compose 常见问题
linkTitle: 常见问题
weight: 10
tags: [FAQ]
aliases:
- /compose/faq/
---

### `docker compose` 与 `docker-compose` 有何区别？

Docker Compose 命令行二进制的第一个版本发布于 2014 年，使用 Python 编写，通过 `docker-compose` 调用。通常，Compose v1 的项目会在 `compose.yaml` 中包含顶级的 `version` 字段，取值范围 2.0 到 3.8，表示特定的文件格式版本。

Docker Compose 的第二个版本于 2020 年发布，使用 Go 编写，通过 `docker compose` 调用。Compose v2 会忽略 `compose.yaml` 中顶级的 `version` 字段。

更多信息参见《[Compose 的发展历程](/manuals/compose/intro/history.md)》。

### `up`、`run` 与 `start` 有何区别？

通常你会使用 `docker compose up`。`up` 用于启动或重启 `compose.yaml` 中定义的所有服务。在默认的“附加”（attached）模式下，你可以看到所有容器的日志；在“分离”（detached）模式（`-d`）下，Compose 在启动容器后退出，但容器会在后台继续运行。

`docker compose run` 用于运行一次性（one-off）或临时（adhoc）任务。该命令需要指定要运行的服务名，并只会启动该服务及其依赖的服务容器。可用 `run` 来运行测试，或执行管理任务，例如向数据卷容器添加或移除数据。`run` 的行为类似 `docker run -ti`，会打开一个交互式终端并返回容器内进程的退出状态码。

`docker compose start` 只用于重新启动已创建但停止的容器，不会创建新容器。

### 为什么服务需要 10 秒才会重新创建或停止？

`docker compose stop` 会先向容器发送 `SIGTERM` 以尝试停止容器，然后等待[默认 10 秒超时](/reference/cli/docker/compose/stop.md)。超时后会发送 `SIGKILL` 强制终止容器。如果你总在等待这个超时时间，说明容器在收到 `SIGTERM` 时没有正常退出。

关于容器内[进程处理信号](https://medium.com/@gchudnov/trapping-signals-in-docker-containers-7a57fdda7d86)的问题已有大量讨论。

要解决这个问题，可以尝试：

- 确保在 Dockerfile 中使用 `CMD` 与 `ENTRYPOINT` 的 exec 形式。

  例如，使用 `["program", "arg1", "arg2"]`，而不是 `"program arg1 arg2"`。
  字符串形式会导致 Docker 通过 `bash` 运行你的进程，而 `bash` 无法正确处理信号。
  Compose 始终使用 JSON 形式，因此即便你在 Compose 文件中覆盖了命令或入口点，也无需担心。

- 如果可行，请修改你的应用，为 `SIGTERM` 增加显式的信号处理。

- 将 `stop_signal` 设置为应用能够正确处理的信号：

  ```yaml
  services:
    web:
      build: .
      stop_signal: SIGINT
  ```

- 如果无法修改应用，可将其封装在轻量级 init 系统（如 [s6](https://skarnet.org/software/s6/)）或信号代理（如 [dumb-init](https://github.com/Yelp/dumb-init) 或 [tini](https://github.com/krallin/tini)）中，它们能正确处理 `SIGTERM`。

### 如何在同一主机上运行一个 Compose 项目的多个副本？

Compose 使用项目名为项目的容器及其他资源创建唯一标识。若要运行同一项目的多个副本，可通过命令行选项 `-p` 或设置 [`COMPOSE_PROJECT_NAME` 环境变量](/manuals/compose/how-tos/environment-variables/envvars.md#compose_project_name) 指定自定义项目名。

### Compose 文件可以用 JSON 替代 YAML 吗？

可以。[YAML 是 JSON 的超集](https://stackoverflow.com/a/1729545/444646)，因此任何 JSON 文件都应是有效的 YAML。要在 Compose 中使用 JSON 文件，请显式指定文件名，例如：

```console
$ docker compose -f compose.json up
```

### 该用 `COPY`/`ADD` 打包代码，还是使用卷挂载？

你可以在 `Dockerfile` 中使用 `COPY` 或 `ADD` 指令将代码添加到镜像中。如果需要随镜像迁移代码（例如发送到其他环境：生产、CI 等），这种方式很有用。

如果希望修改代码后立即生效（例如开发阶段你的服务支持热重载或实时刷新），可以使用 `volume` 挂载代码。

在某些场景下，你也可以两者结合：镜像中通过 `COPY` 包含代码；开发阶段再通过 Compose 文件中的 `volume` 挂载宿主机上的代码。卷会覆盖镜像中对应目录的内容。
