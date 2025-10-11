---
description: 如何自动启动容器
keywords: containers, restart, policies, automation, administration
title: 自动启动容器
weight: 10
aliases:
  - /engine/articles/host_integration/
  - /engine/admin/host_integration/
  - /engine/admin/start-containers-automatically/
  - /config/containers/start-containers-automatically/
---

Docker 提供了[重启策略](/reference/cli/docker/container/run.md#restart)，用于控制容器在退出后或 Docker 重启时是否自动启动。重启策略还能确保按正确的顺序启动存在依赖关系的容器。Docker 建议优先使用重启策略，而不要通过进程管理器来启动容器。

需要注意，重启策略与 `dockerd` 命令的 `--live-restore` 标志不同。使用 `--live-restore` 可以在升级 Docker 时保持容器继续运行（其间网络与交互可能会短暂中断）。

## 使用重启策略

在使用 `docker run` 时，通过 [`--restart`](/reference/cli/docker/container/run.md#restart) 选项为容器配置重启策略。`--restart` 的取值可以是：

| 标志                       | 说明                                                                                                                                                                                                                                                                                                                                                                   |
| :------------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `no`                       | 不自动重启容器。（默认）                                                                                                                                                                                                                                                                                                                                              |
| `on-failure[:max-retries]` | 当容器因错误退出（非零退出码）时重启。可通过 `:max-retries` 限制 Docker 守护进程尝试重启的次数。该策略仅在容器失败退出时触发重启；若仅是守护进程本身重启，并不会重启容器。                                                                                                                                            |
| `always`                   | 只要容器停止就始终重启。如果是手动停止，则仅在 Docker 守护进程重启或手动重启容器时才会再次启动。（见[重启策略细节](#重启策略细节)中的第二条）                                                                                                                                                                                                                   |
| `unless-stopped`           | 与 `always` 类似，但如果容器被停止（无论是否手动），即使 Docker 守护进程重启，也不会自动再次启动。                                                                                                                                                                                                                             |

下面的命令启动一个 Redis 容器，并配置为除非显式停止或守护进程重启，否则始终重启：

```console
$ docker run -d --restart unless-stopped redis
```

下面的命令为一个名为 `redis` 的正在运行的容器更改重启策略：

```console
$ docker update --restart unless-stopped redis
```

下面的命令为所有正在运行的容器设置重启策略：

```console
$ docker update --restart unless-stopped $(docker ps -q)
```

### 重启策略细节

使用重启策略时请注意：

- 重启策略仅在容器成功启动后生效。这里的“成功启动”指容器至少运行了 10 秒且已被 Docker 监控。这样可以避免完全无法启动的容器陷入无限重启循环。

- 如果你手动停止了容器，在 Docker 守护进程重启或手动重启容器之前，重启策略将被忽略。这同样是为了避免重启循环。

- 重启策略仅适用于容器。若需为 Swarm 服务配置重启行为，请参考[与服务重启相关的参数](/reference/cli/docker/service/create.md)。

### 前台容器的重启行为

当前台运行容器时，停止容器会导致附着的 CLI 会话一并退出，与容器的重启策略无关。如下例所示：

1. 创建一个 Dockerfile，打印 1 到 5 然后退出：

   ```dockerfile
   FROM busybox:latest
   COPY --chmod=755 <<"EOF" /start.sh
   echo "Starting..."
   for i in $(seq 1 5); do
     echo "$i"
     sleep 1
   done
   echo "Exiting..."
   exit 1
   EOF
   ENTRYPOINT /start.sh
   ```

2. 基于该 Dockerfile 构建镜像：

   ```console
   $ docker build -t startstop .
   ```

3. 以 `always` 为重启策略运行容器。

   容器会向标准输出打印 1..5，随后退出；附着在其上的 CLI 会话也会同时退出。

   ```console
   $ docker run --restart always startstop
   Starting...
   1
   2
   3
   4
   5
   Exiting...
   $
   ```

4. 此时执行 `docker ps` 可以看到容器仍在运行或处于重启中（得益于重启策略）。不过 CLI 会话已经退出，不会在容器首次退出后继续保持。

   ```console
   $ docker ps
   CONTAINER ID   IMAGE       COMMAND                  CREATED         STATUS         PORTS     NAMES
   081991b35afe   startstop   "/bin/sh -c /start.sh"   9 seconds ago   Up 4 seconds             gallant_easley
   ```

5. 你可以在两次重启之间使用 `docker container attach` 重新附着到容器；下一次容器退出时会再次与终端分离。

   ```console
   $ docker container attach 081991b35afe
   4
   5
   Exiting...
   $
   ```

## 使用进程管理器

如果重启策略不符合你的场景（例如宿主机上的其他进程依赖容器），可以考虑使用宿主级进程管理器，例如 [systemd](https://systemd.io/) 或 [supervisor](http://supervisord.org/)。

> [!WARNING]
>
> 不要将 Docker 的重启策略与宿主级进程管理器混用，否则会产生冲突。

若使用进程管理器，请将其配置为调用你平时用于手动启动的命令，例如 `docker start` 或 `docker service`。更多细节请参考对应进程管理器的官方文档。

### 在容器内使用进程管理器

你也可以在容器内部运行进程管理器，用于检查某个进程是否在运行，并在需要时启动/重启它。

> [!WARNING]
>
> 这类工具并不了解 Docker 的上下文，它们只会监控容器内的操作系统进程。Docker 不推荐这种做法，因为它依赖具体平台，并可能随不同发行版或版本而产生差异。
