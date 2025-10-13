---
description: 了解如何为 Docker 守护进程配置日志驱动
keywords: docker, 日志, 驱动
title: 配置日志驱动
aliases:
  - /config/containers/logging/logentries/
  - /engine/reference/logging/overview/
  - /engine/reference/logging/
  - /engine/admin/reference/logging/
  - /engine/admin/logging/logentries/
  - /engine/admin/logging/overview/
  - /config/containers/logging/configure/
  - /config/containers/
---

Docker 提供了多种日志机制，帮助你获取正在运行的容器与服务的日志信息。这些机制被称为“日志驱动”。每个 Docker 守护进程都有一个默认的日志驱动，除非你为容器单独指定，否则容器都会使用该默认驱动。

默认情况下，Docker 使用 [`json-file` 日志驱动](drivers/json-file.md)，它会以 JSON 格式在本地缓存容器日志。除了使用 Docker 内置的日志驱动外，你还可以编写并使用[日志驱动插件](plugins.md)。

> [!TIP]
>
> 建议使用 `local` 日志驱动来避免磁盘被日志耗尽。默认情况下不执行日志轮换，因此对于日志量很大的容器，默认的
> [`json-file` 日志驱动](drivers/json-file.md) 可能会占用大量磁盘空间，最终导致磁盘耗尽。
>
> Docker 将不带日志轮换的 json-file 作为默认值，是为了与旧版本保持向后兼容，以及适配将 Docker 作为 Kubernetes 运行时的场景。
>
> 对于其他场景，推荐使用 `local` 日志驱动：它默认执行日志轮换，并采用更高效的文件格式。参见下文的
> [配置默认日志驱动](#configure-the-default-logging-driver) 了解如何将 `local` 设为默认；更多细节见
> [本地文件日志驱动](drivers/local.md)。

## 配置默认日志驱动

若要为 Docker 守护进程指定默认的日志驱动，请在 `daemon.json` 配置文件中，将 `log-driver` 的值设置为目标驱动名称。详见
[`dockerd` 参考文档](/reference/cli/dockerd/#daemon-configuration-file)中的“daemon configuration file”章节。

默认日志驱动为 `json-file`。下面的示例将默认日志驱动设置为 [`local` 日志驱动](drivers/local.md)：

```json
{
  "log-driver": "local"
}
```

如果目标日志驱动支持配置项，可以在 `daemon.json` 文件中通过 `log-opts`（JSON 对象）进行设置。如下示例为 `json-file` 日志驱动设置了 4 个配置项：

```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3",
    "labels": "production_status",
    "env": "os,customer"
  }
}
```

重启 Docker 后，新创建的容器会使用更新的配置。已存在的容器不会自动应用新的日志配置。

> [!NOTE]
>
> `daemon.json` 中的 `log-opts` 配置项必须以字符串形式提供。因此布尔与数值类型（如上例中的 `max-file`）也需要用引号（`"`）包裹。

如果未显式指定日志驱动，默认值为 `json-file`。
要查看当前 Docker 守护进程的默认日志驱动，可运行 `docker info` 并查找 `Logging Driver`。在 Linux、macOS 或 Windows 的 PowerShell 中可使用：

```console
$ docker info --format '{{.LoggingDriver}}'

json-file
```

> [!NOTE]
>
> 在守护进程配置中修改默认日志驱动或其选项，仅会影响修改之后创建的容器。现有容器仍保留它们创建时的日志驱动配置。若要更新某个容器的日志驱动，需要使用期望的选项重新创建该容器。参见下文[为容器配置日志驱动](#configure-the-logging-driver-for-a-container)了解如何查看容器的日志驱动配置。

## 为容器配置日志驱动

启动容器时，可以通过 `--log-driver` 为其指定一个不同于守护进程默认值的日志驱动。若该驱动支持配置项，可通过一个或多个 `--log-opt <NAME>=<VALUE>` 进行设置。即使容器使用默认日志驱动，也可以使用不同的配置项。

下面的示例使用 `none` 日志驱动启动一个 Alpine 容器：

```console
$ docker run -it --log-driver none alpine ash
```

要查看某个正在运行容器当前使用的日志驱动，可运行如下 `docker inspect` 命令（将 `<CONTAINER>` 替换为容器名或 ID）：

```console
$ docker inspect -f '{{.HostConfig.LogConfig.Type}}' <CONTAINER>

json-file
```

## 配置从容器到日志驱动的日志投递模式

Docker 提供两种从容器向日志驱动投递日志的模式：

- （默认）直接、阻塞式地从容器写入到驱动
- 非阻塞式：将日志写入到每个容器各自的中间缓冲区，由驱动异步消费

`non-blocking` 模式可以避免因日志回压导致应用阻塞。当 `STDERR` 或 `STDOUT` 被阻塞时，应用可能会以不可预期的方式失败。

> [!WARNING]
>
> 当缓冲区被写满时，新日志将不会入队。相较于阻塞应用的日志写入流程，丢弃部分日志通常是更可取的策略。

`mode` 日志选项用于控制采用 `blocking`（默认）还是 `non-blocking` 的投递模式。

当 `mode` 为 `non-blocking` 时，`max-buffer-size` 用于设置中间缓冲区的大小。默认值为 `1m`，即 1 MB（约一百万字节）。允许的格式字符串参见 [`go-units` 包的 `FromHumanSize()`](https://pkg.go.dev/github.com/docker/go-units#FromHumanSize)，例如：`1KiB` 表示 1024 字节，`2g` 表示 2×10^9 字节。

下面的示例以非阻塞模式并设置 4 MB 缓冲区启动一个 Alpine 容器：

```console
$ docker run -it --log-opt mode=non-blocking --log-opt max-buffer-size=4m alpine ping 127.0.0.1
```

### 在日志驱动中使用环境变量或标签

部分日志驱动会将容器的 `--env|-e` 或 `--label` 的值添加到容器日志中。如下示例在使用守护进程默认日志驱动（此处为 `json-file`）的同时，设置了环境变量 `os=ubuntu`：

```console
$ docker run -dit --label production_status=testing -e os=ubuntu alpine sh
```

若日志驱动支持，上述设置会在日志输出中添加额外字段。以下为 `json-file` 日志驱动生成的输出片段：

```json
"attrs":{"production_status":"testing","os":"ubuntu"}
```

## 支持的日志驱动

Docker 支持以下日志驱动。各驱动的配置项（如适用）见对应文档链接。若你使用[日志驱动插件](plugins.md)，可获得更多选项。

| 驱动                                  | 说明                                                                                                        |
| :------------------------------------ | :---------------------------------------------------------------------------------------------------------- |
| `none`                                | 不为容器提供日志，`docker logs` 不会返回任何输出。                                                          |
| [`local`](drivers/local.md)           | 以定制格式存储日志，尽量降低开销。                                                                          |
| [`json-file`](drivers/json-file.md)   | 以 JSON 格式存储日志。Docker 的默认日志驱动。                                                               |
| [`syslog`](drivers/syslog.md)         | 将日志写入 `syslog`。主机上必须运行 `syslog` 守护进程。                                                     |
| [`journald`](drivers/journald.md)     | 将日志写入 `journald`。主机上必须运行 `journald` 守护进程。                                                 |
| [`gelf`](drivers/gelf.md)             | 将日志写入 GELF 端点（例如 Graylog 或 Logstash）。                                                           |
| [`fluentd`](drivers/fluentd.md)       | 将日志写入 `fluentd`（forward 输入）。主机上必须运行 `fluentd` 守护进程。                                   |
| [`awslogs`](drivers/awslogs.md)       | 将日志写入 Amazon CloudWatch Logs。                                                                          |
| [`splunk`](drivers/splunk.md)         | 使用 HTTP 事件收集器（HEC）将日志写入 `splunk`。                                                             |
| [`etwlogs`](drivers/etwlogs.md)       | 将日志作为 Windows 事件跟踪（ETW）事件写入。仅适用于 Windows 平台。                                          |
| [`gcplogs`](drivers/gcplogs.md)       | 将日志写入 Google Cloud Platform (GCP) Logging。                                                             |

## 日志驱动的限制

- 读取日志时需要解压轮换后的日志文件，这会暂时增加磁盘使用（直至读取完成），并在解压过程中增加 CPU 使用。
- 日志可占用的最大空间受 Docker 数据目录所在主机存储容量的限制。
