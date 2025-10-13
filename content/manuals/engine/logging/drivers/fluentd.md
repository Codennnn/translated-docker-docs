---
description: 了解如何使用 fluentd 日志驱动
keywords: Fluentd, docker, 日志, 驱动
title: Fluentd 日志驱动
aliases:
  - /engine/reference/logging/fluentd/
  - /reference/logging/fluentd/
  - /engine/admin/logging/fluentd/
  - /config/containers/logging/fluentd/
---

`fluentd` 日志驱动会将容器日志以结构化数据的形式发送到
[Fluentd](https://www.fluentd.org) 收集器。随后，你可以借助
[Fluentd 的各类输出插件](https://www.fluentd.org/plugins) 将这些日志写入不同的目的地。

除了日志消息本身，`fluentd` 日志驱动还会在结构化日志中附带以下元数据：

| 字段              | 说明                                                                                                                                             |
| :---------------- | :----------------------------------------------------------------------------------------------------------------------------------------------- |
| `container_id`    | 完整的 64 位容器 ID。                                                                                                                            |
| `container_name`  | 容器启动时的名称。若之后通过 `docker rename` 重命名容器，日志条目中不会反映该新名称。                                                              |
| `source`          | `stdout` 或 `stderr`                                                                                                                             |
| `log`             | 容器日志内容                                                                                                                                      |

## 用法

部分选项可通过在命令中多次指定 `--log-opt` 来配置：

- `fluentd-address`：指定连接 Fluentd 守护进程的套接字地址，例如 `fluentdhost:24224` 或 `unix:///path/to/fluentd.sock`。
- `tag`：为发送到 Fluentd 的消息指定标签。支持部分 Go 模板占位符，例如 `{{.ID}}`、`{{.FullID}}`、`{{.Name}}`、`docker.{{.ID}}`。

要将 `fluentd` 作为默认日志驱动，请在 `daemon.json` 中设置 `log-driver` 与 `log-opt`。
该文件在 Linux 上位于 `/etc/docker/`，在 Windows Server 上为
`C:\\ProgramData\\docker\\config\\daemon.json`。关于通过 `daemon.json` 配置 Docker，参见
[daemon.json](/reference/cli/dockerd.md#daemon-configuration-file)。

以下示例将日志驱动设置为 `fluentd`，并设置 `fluentd-address` 选项：

```json
{
  "log-driver": "fluentd",
  "log-opts": {
    "fluentd-address": "fluentdhost:24224"
  }
}
```

重启 Docker 以使更改生效。

> [!NOTE]
>
> 在 `daemon.json` 配置文件中，`log-opts` 选项必须以字符串形式提供。
> 因此布尔值和数值（例如 `fluentd-async` 或 `fluentd-max-retries` 的取值）必须使用引号（`"`）包裹。

也可以在 `docker run` 中为特定容器设置日志驱动：

```console
$ docker run --log-driver=fluentd ...
```

在使用该日志驱动前，请先启动一个 Fluentd 守护进程。默认情况下，日志驱动会通过 `localhost:24224` 连接。
可使用 `fluentd-address` 选项连接到其他地址：

```console
$ docker run --log-driver=fluentd --log-opt fluentd-address=fluentdhost:24224
```

如果容器无法连接到 Fluentd 守护进程，除非启用 `fluentd-async` 选项，否则容器会立即退出。

## 选项

可以使用 `--log-opt NAME=VALUE` 为 Fluentd 日志驱动指定其他选项。

### fluentd-address

默认情况下，日志驱动连接到 `localhost:24224`。通过 `fluentd-address` 可以指定其他地址。
支持 `tcp`（默认）与 `unix` 套接字。

```console
$ docker run --log-driver=fluentd --log-opt fluentd-address=fluentdhost:24224
$ docker run --log-driver=fluentd --log-opt fluentd-address=tcp://fluentdhost:24224
$ docker run --log-driver=fluentd --log-opt fluentd-address=unix:///path/to/fluentd.sock
```

上述示例中前两行等价，因为默认协议为 `tcp`。

### tag

默认情况下，Docker 使用容器 ID 的前 12 个字符作为日志标签。
若需自定义标签格式，参见[日志标签选项文档](log_tags.md)。

### labels、labels-regex、env 与 env-regex

`labels` 与 `env` 选项分别接收一个以逗号分隔的键列表。
当 `label` 与 `env` 键发生冲突时，以 `env` 的值为准。这两个选项都会在日志条目的附加属性中加入额外字段。

`env-regex` 与 `labels-regex` 的作用与 `env`、`labels` 类似且兼容，它们的取值是用于匹配与日志相关的环境变量和标签的正则表达式。
通常用于更高级的[日志标签选项](log_tags.md)。

### fluentd-async

Docker 会在后台连接 Fluentd。在连接建立之前，消息会被缓冲。默认值为 `false`。

### fluentd-async-reconnect-interval

启用 `fluentd-async` 时，`fluentd-async-reconnect-interval` 用于设置重新建立到 `fluentd-address` 连接的间隔（毫秒）。
当该地址解析为一个或多个 IP（例如 Consul 的服务地址）时，此选项尤为有用。

### fluentd-buffer-limit

设置内存中的事件缓冲上限。日志记录会在内存中保存，直到达到该数量。
缓冲区满时，记录日志的调用会失败。默认值为 1048576。
参考：https://github.com/fluent/fluent-logger-golang/tree/master#bufferlimit

### fluentd-retry-wait

两次重试之间的等待时间。默认 1 秒。

### fluentd-max-retries

最大重试次数。默认值为 `4294967295`（2**32 - 1）。

### fluentd-sub-second-precision

以纳秒精度生成日志事件。默认值为 `false`。

### fluentd-write-timeout

设置向 `fluentd` 守护进程写入的超时时间。默认情况下不设置超时，写入将无限期阻塞。

## 借助 Docker 管理 Fluentd 守护进程

关于 `Fluentd` 本身，请参见[项目主页](https://www.fluentd.org)与[官方文档](https://docs.fluentd.org)。

要使用该日志驱动，你需要在某台主机上启动 `fluentd` 守护进程。推荐使用
[Fluentd 的 Docker 镜像](https://hub.docker.com/r/fluent/fluentd/)。
如果你希望先在每台主机上聚合多个容器的日志，然后再将它们转发到另一台 Fluentd 节点进行集中存储，该镜像会非常有用。

### 测试容器日志输出

1.  编写一个配置文件（`test.conf`）用于输出接收到的日志：

    ```text
    <source>
      @type forward
    </source>

    <match *>
      @type stdout
    </match>
    ```

2.  使用该配置文件启动 Fluentd 容器：

    ```console
    $ docker run -it -p 24224:24224 -v /path/to/conf/test.conf:/fluentd/etc/test.conf -e FLUENTD_CONF=test.conf fluent/fluentd:latest
    ```

3.  使用 `fluentd` 日志驱动启动一个或多个容器：

    ```console
    $ docker run --log-driver=fluentd your/application
    ```
