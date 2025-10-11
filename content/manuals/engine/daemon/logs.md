---
title: 查看守护进程日志
description: 了解如何读取 Docker 守护进程的事件日志
keywords: docker, daemon, configuration, troubleshooting, logging
aliases:
  - /config/daemon/logs/
---

守护进程日志有助于诊断问题。日志保存位置取决于操作系统配置和所用的日志子系统，可能位于以下位置：

| 操作系统                            | 日志位置                                                                                                                                 |
| :--------------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------- |
| Linux                              | 使用 `journalctl -xu docker.service`（或根据发行版查看 `/var/log/syslog` 或 `/var/log/messages`）                                           |
| macOS（`dockerd` 日志）             | `~/Library/Containers/com.docker.docker/Data/log/vm/dockerd.log`                                                                         |
| macOS（`containerd` 日志）          | `~/Library/Containers/com.docker.docker/Data/log/vm/containerd.log`                                                                      |
| Windows（WSL2）（`dockerd` 日志）    | `%LOCALAPPDATA%\Docker\log\vm\dockerd.log`                                                                                               |
| Windows（WSL2）（`containerd` 日志） | `%LOCALAPPDATA%\Docker\log\vm\containerd.log`                                                                                            |
| Windows（Windows 容器）             | 查看 Windows 事件日志                                                                                                                     |

在 macOS 上查看 `dockerd` 日志：打开终端，使用 `tail -f` 持续跟随输出；按 `CTRL+c` 终止：

```console
$ tail -f ~/Library/Containers/com.docker.docker/Data/log/vm/dockerd.log
2021-07-28T10:21:21Z dockerd time="2021-07-28T10:21:21.497642089Z" level=debug msg="attach: stdout: begin"
2021-07-28T10:21:21Z dockerd time="2021-07-28T10:21:21.497714291Z" level=debug msg="attach: stderr: begin"
2021-07-28T10:21:21Z dockerd time="2021-07-28T10:21:21.499798390Z" level=debug msg="Calling POST /v1.41/containers/35fc5ec0ffe1ad492d0a4fbf51fd6286a087b89d4dd66367fa3b7aec70b46a40/wait?condition=removed"
2021-07-28T10:21:21Z dockerd time="2021-07-28T10:21:21.518403686Z" level=debug msg="Calling GET /v1.41/containers/35fc5ec0ffe1ad492d0a4fbf51fd6286a087b89d4dd66367fa3b7aec70b46a40/json"
2021-07-28T10:21:21Z dockerd time="2021-07-28T10:21:21.527074928Z" level=debug msg="Calling POST /v1.41/containers/35fc5ec0ffe1ad492d0a4fbf51fd6286a087b89d4dd66367fa3b7aec70b46a40/start"
2021-07-28T10:21:21Z dockerd time="2021-07-28T10:21:21.528203579Z" level=debug msg="container mounted via layerStore: &{/var/lib/docker/overlay2/6e76ffecede030507fcaa576404e141e5f87fc4d7e1760e9ce5b52acb24
...
^C
```

## 启用调试日志

启用调试有两种方法。推荐在 `daemon.json` 中将 `debug` 设为 `true`，该方法适用于所有 Docker 平台。

1.  编辑位于 `/etc/docker/` 的 `daemon.json`（如不存在需新建）。在 macOS 或 Windows 上不要直接编辑该文件，
    请在 Docker Desktop 设置中修改。

2.  如果文件为空，添加以下内容：

    ```json
    {
      "debug": true
    }
    ```

    如果已有其他 JSON 配置，仅需添加 `"debug": true`。若该键不是最后一项，注意在行尾添加逗号。
    同时检查 `log-level`（如已设置）是否为 `info` 或 `debug`。`info` 为默认值，可选：`debug`、`info`、`warn`、`error`、`fatal`。

3.  向守护进程发送 `HUP` 信号以重新加载配置。在 Linux 上使用：

    ```console
    $ sudo kill -SIGHUP $(pidof dockerd)
    ```

    在 Windows 上，重启 Docker。

你也可以停止 Docker 守护进程，并使用 `-D` 标志手动启动以开启调试。
但这样可能绕过宿主机启动脚本设置的环境，反而增加调试难度。

## 强制输出堆栈跟踪

当守护进程无响应时，可以向其发送 `SIGUSR1` 信号，强制记录完整的堆栈跟踪。

- **Linux**：

  ```console
  $ sudo kill -SIGUSR1 $(pidof dockerd)
  ```

- **Windows Server**：

  下载 [docker-signal](https://github.com/moby/docker-signal)。

  获取 dockerd 的进程 ID：`Get-Process dockerd`。

  运行可执行文件并传入 `--pid=<daemon 的 PID>`。

这会强制写入堆栈跟踪，但不会停止守护进程。日志中会显示堆栈内容，或指向保存堆栈的文件路径。

守护进程在处理完 `SIGUSR1` 并输出堆栈后会继续运行。堆栈可用于分析守护进程中所有 goroutine 与线程的状态。

## 查看堆栈跟踪

可以通过以下方式查看 Docker 守护进程日志：

- 在使用 `systemctl` 的 Linux 系统上执行 `journalctl -u docker.service`
- 对于较旧的 Linux 系统，可查看 `/var/log/messages`、`/var/log/daemon.log` 或 `/var/log/docker.log`

> [!NOTE]
>
> 在 Docker Desktop for Mac / Windows 上，无法手动生成堆栈跟踪。
> 如遇问题，可点击任务栏中的 Docker 图标，选择 **Troubleshoot** 以发送诊断信息。

在 Docker 日志中查找类似如下的消息：

```text
...goroutine stacks written to /var/run/docker/goroutine-stacks-2017-06-02T193336z.log
```

Docker 保存堆栈跟踪和转储的位置取决于操作系统与配置。你可以直接从这些信息中获取诊断线索，
或将其提供给 Docker 以协助定位问题。
