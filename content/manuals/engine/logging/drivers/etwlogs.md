---
description: 了解如何在 Docker Engine 中使用 Windows 事件跟踪（ETW）日志驱动
keywords: ETW, docker, 日志, 驱动
title: ETW 日志驱动
aliases:
  - /engine/admin/logging/etwlogs/
  - /config/containers/logging/etwlogs/
---

Windows 事件跟踪（Event Tracing for Windows，ETW）日志驱动会将容器日志转发为 ETW 事件。
ETW 是 Windows 上用于应用跟踪的通用框架。每个 ETW 事件都包含一条消息，其中既有日志内容，也包含上下文信息。客户端可以创建 ETW 监听器来监听这些事件。

该日志驱动在 Windows 中注册的 ETW 提供程序（provider）具有如下 GUID 标识：`{a3693192-9ed6-46d2-a981-f8226c8363bd}`。
客户端创建 ETW 监听器并注册监听来自此提供程序的事件。提供程序与监听器的创建顺序无关：在提供程序注册到系统之前，客户端就可以创建监听器并开始监听其事件。

## 用法

下面演示如何使用大多数 Windows 安装中自带的 logman 工具来监听这些事件：

1. `logman start -ets DockerContainerLogs -p "{a3693192-9ed6-46d2-a981-f8226c8363bd}" 0x0 -o trace.etl`
2. 通过在 Docker run 命令中添加 `--log-driver=etwlogs`，使用 etwlogs 驱动运行容器，并产生一些日志。
3. `logman stop -ets DockerContainerLogs`
4. 以上步骤会生成包含事件的 etl 文件。可通过运行 `tracerpt -y trace.etl` 将其转换为可读格式。

每条 ETW 事件包含一条结构化的消息字符串，格式如下：

```text
container_name: %s, image_name: %s, container_id: %s, image_id: %s, source: [stdout | stderr], log: %s
```

各字段含义如下：

| 字段              | 说明                                           |
| ---------------- | ---------------------------------------------- |
| `container_name` | 容器启动时的名称。                             |
| `image_name`     | 容器所用镜像的名称。                           |
| `container_id`   | 完整的 64 位容器 ID。                           |
| `image_id`       | 容器所用镜像的完整 ID。                         |
| `source`         | `stdout` 或 `stderr`。                          |
| `log`            | 容器日志消息内容。                             |

下面是一个事件消息示例（为便于阅读进行了格式化）：

```yaml
container_name: backstabbing_spence,
image_name: windowsservercore,
container_id: f14bb55aa862d7596b03a33251c1be7dbbec8056bbdead1da8ec5ecebbe29731,
image_id: sha256:2f9e19bd998d3565b4f345ac9aaf6e3fc555406239a4fb1b1ba879673713824b,
source: stdout,
log: Hello world!
```

客户端可以解析该消息字符串，从中获取日志消息及其上下文信息。时间戳也包含在 ETW 事件中。

> [!NOTE]
>
> 此 ETW 提供程序仅输出消息字符串，而非专门结构化的 ETW 事件。因此，无需在系统中注册清单文件即可读取与解析这些 ETW 事件。
