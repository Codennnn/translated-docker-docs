---
description: 了解如何在 Docker Engine 中使用 Amazon CloudWatch Logs 日志驱动
keywords: AWS, Amazon, CloudWatch, 日志, 驱动
title: Amazon CloudWatch Logs 日志驱动
aliases:
  - /engine/reference/logging/awslogs/
  - /engine/admin/logging/awslogs/
  - /config/containers/logging/awslogs/
---

`awslogs` 日志驱动会将容器日志发送到
[Amazon CloudWatch Logs](https://aws.amazon.com/cloudwatch/details/#log-monitoring)。
你可以通过 [AWS 管理控制台](https://console.aws.amazon.com/cloudwatch/home#logs:)
或 [AWS SDK 与命令行工具](https://docs.aws.amazon.com/cli/latest/reference/logs/index.html) 查看日志。

## 用法

要将 `awslogs` 作为默认日志驱动，请在 `daemon.json` 中设置 `log-driver` 与 `log-opt`。
该文件在 Linux 上位于 `/etc/docker/`，在 Windows Server 上为
`C:\\ProgramData\\docker\\config\\daemon.json`。关于通过 `daemon.json` 配置 Docker，参见
[daemon.json](/reference/cli/dockerd.md#daemon-configuration-file)。
以下示例将日志驱动设置为 `awslogs`，并设置 `awslogs-region` 选项：

```json
{
  "log-driver": "awslogs",
  "log-opts": {
    "awslogs-region": "us-east-1"
  }
}
```

重启 Docker 以使更改生效。

也可以在 `docker run` 中为特定容器设置日志驱动：

```console
$ docker run --log-driver=awslogs ...
```

如果使用 Docker Compose，可按如下方式声明使用 `awslogs`：

```yaml
myservice:
  logging:
    driver: awslogs
    options:
      awslogs-region: us-east-1
```

## Amazon CloudWatch Logs 选项

你可以在 `daemon.json` 中添加日志选项作为全局默认值，或在启动容器时使用 `--log-opt NAME=VALUE` 指定 CloudWatch Logs 日志驱动的选项。

### awslogs-region

`awslogs` 日志驱动会将日志发送到指定区域。使用 `awslogs-region` 日志选项或 `AWS_REGION` 环境变量设置区域。
默认情况下，如果 Docker 守护进程运行在 EC2 实例上且未设置区域，将使用该实例所在区域。

```console
$ docker run --log-driver=awslogs --log-opt awslogs-region=us-east-1 ...
```

### awslogs-endpoint

默认情况下，Docker 使用 `awslogs-region` 或检测到的区域来构建 CloudWatch Logs API 端点。
使用 `awslogs-endpoint` 可覆盖默认端点。

> [!NOTE]
>
> `awslogs-region` 或检测到的区域决定签名所用的区域；若 `awslogs-endpoint` 指向其他区域，可能出现签名错误。

### awslogs-group

必须为 `awslogs` 指定一个
[日志组（log group）](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/WhatIsCloudWatchLogs.html)。
通过 `awslogs-group` 选项设置：

```console
$ docker run --log-driver=awslogs --log-opt awslogs-region=us-east-1 --log-opt awslogs-group=myLogGroup ...
```

### awslogs-stream

要配置使用哪个
[日志流（log stream）](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/WhatIsCloudWatchLogs.html)，
可使用 `awslogs-stream`。未设置时，默认使用容器 ID 作为日志流。

> [!NOTE]
>
> 同一日志组下的日志流一次应仅供一个容器使用；多个容器并发使用同一日志流会降低日志性能。

### awslogs-create-group

默认情况下，若日志组不存在，日志驱动会返回错误。可将 `awslogs-create-group` 设为 `true`，按需自动创建日志组。该选项默认值为 `false`。

```console
$ docker run \
    --log-driver=awslogs \
    --log-opt awslogs-region=us-east-1 \
    --log-opt awslogs-group=myLogGroup \
    --log-opt awslogs-create-group=true \
    ...
```

> [!NOTE]
>
> 使用 `awslogs-create-group` 前，你的 AWS IAM 策略必须包含 `logs:CreateLogGroup` 权限。

### awslogs-create-stream

默认情况下，日志驱动会创建用于容器日志持久化的 CloudWatch 日志流。

将 `awslogs-create-stream` 设为 `false` 可禁用该行为；此时 Docker 守护进程假定日志流已存在。一个常见场景是由其他进程创建日志流，以避免重复的 API 调用。

若将其设为 `false` 而日志流不存在，则运行时无法将日志持久化到 CloudWatch，并在守护进程日志中出现 `Failed to put log events` 错误。

```console
$ docker run \
    --log-driver=awslogs \
    --log-opt awslogs-region=us-east-1 \
    --log-opt awslogs-group=myLogGroup \
    --log-opt awslogs-stream=myLogStream \
    --log-opt awslogs-create-stream=false \
    ...
```

### awslogs-datetime-format

`awslogs-datetime-format` 用于以 [Python `strftime` 格式](https://strftime.org) 定义多行日志的起始模式。
一条日志消息由匹配该模式的第一行以及后续不匹配的行组成；匹配行同时也充当日志消息的分隔符。

一个典型场景是解析栈追踪等输出，否则这类内容可能被拆分为多条日志；使用合适的模式即可作为单条日志捕获。

当同时设置了 `awslogs-datetime-format` 与 `awslogs-multiline-pattern` 时，前者优先生效。

> [!NOTE]
>
> 多行日志需要对所有日志进行正则解析与匹配，可能对日志性能产生影响。

Consider the following log stream, where new log messages start with a
timestamp:

```console
[May 01, 2017 19:00:01] A message was logged
[May 01, 2017 19:00:04] Another multi-line message was logged
Some random message
with some random words
[May 01, 2017 19:01:32] Another message was logged
```

该格式可表述为 `strftime` 表达式 `[%b %d, %Y %H:%M:%S]`，将其作为 `awslogs-datetime-format` 的值：

```console
$ docker run \
    --log-driver=awslogs \
    --log-opt awslogs-region=us-east-1 \
    --log-opt awslogs-group=myLogGroup \
    --log-opt awslogs-datetime-format='\[%b %d, %Y %H:%M:%S\]' \
    ...
```

解析后在 CloudWatch 中形成如下日志事件：

```console
# First event
[May 01, 2017 19:00:01] A message was logged

# Second event
[May 01, 2017 19:00:04] Another multi-line message was logged
Some random message
with some random words

# Third event
[May 01, 2017 19:01:32] Another message was logged
```

支持以下 `strftime` 代码：

| Code | Meaning                                                          | Example  |
| :--- | :--------------------------------------------------------------- | :------- |
| `%a` | Weekday abbreviated name.                                        | Mon      |
| `%A` | Weekday full name.                                               | Monday   |
| `%w` | Weekday as a decimal number where 0 is Sunday and 6 is Saturday. | 0        |
| `%d` | Day of the month as a zero-padded decimal number.                | 08       |
| `%b` | Month abbreviated name.                                          | Feb      |
| `%B` | Month full name.                                                 | February |
| `%m` | Month as a zero-padded decimal number.                           | 02       |
| `%Y` | Year with century as a decimal number.                           | 2008     |
| `%y` | Year without century as a zero-padded decimal number.            | 08       |
| `%H` | Hour (24-hour clock) as a zero-padded decimal number.            | 19       |
| `%I` | Hour (12-hour clock) as a zero-padded decimal number.            | 07       |
| `%p` | AM or PM.                                                        | AM       |
| `%M` | Minute as a zero-padded decimal number.                          | 57       |
| `%S` | Second as a zero-padded decimal number.                          | 04       |
| `%L` | Milliseconds as a zero-padded decimal number.                    | .123     |
| `%f` | Microseconds as a zero-padded decimal number.                    | 000345   |
| `%z` | UTC offset in the form +HHMM or -HHMM.                           | +1300    |
| `%Z` | Time zone name.                                                  | PST      |
| `%j` | Day of the year as a zero-padded decimal number.                 | 363      |

### awslogs-multiline-pattern（多行模式正则）

`awslogs-multiline-pattern` 使用正则表达式定义多行日志的起始模式。一条日志消息由匹配该模式的第一行以及后续不匹配的行组成；匹配行也充当消息分隔符。

当同时设置了 `awslogs-datetime-format` 时，本选项被忽略。

> [!NOTE]
>
> 多行日志需要对所有日志进行正则解析与匹配，可能对日志性能产生影响。

Consider the following log stream, where each log message should start with the
pattern `INFO`:

```console
INFO A message was logged
INFO Another multi-line message was logged
     Some random message
INFO Another message was logged
```

可使用正则表达式 `^INFO`：

```console
$ docker run \
    --log-driver=awslogs \
    --log-opt awslogs-region=us-east-1 \
    --log-opt awslogs-group=myLogGroup \
    --log-opt awslogs-multiline-pattern='^INFO' \
    ...
```

解析后日志事件如下：

```console
# First event
INFO A message was logged

# Second event
INFO Another multi-line message was logged
     Some random message

# Third event
INFO Another message was logged
```

### tag（标签）

可用 `tag` 作为 `awslogs-stream` 的替代。`tag` 支持 Go 模板占位符，例如 `{{.ID}}`、`{{.FullID}}`、
`{{.Name}}`、`docker.{{.ID}}`。支持的模板替换详见[标签选项文档](log_tags.md)。

当同时设置 `awslogs-stream` 与 `tag` 时，以 `awslogs-stream` 的值为准。

未设置时，默认使用容器 ID 作为日志流。

> [!NOTE]
>
> CloudWatch 日志 API 不支持在日志名中包含 `:`。因此直接使用 `{{ .ImageName }}` 作为标签可能存在问题（镜像名形如 `IMAGE:TAG`，例：`alpine:latest`）。
> 可通过模板语法对其改写为合适的格式，例如将镜像名中的冒号替换为下划线，并拼接容器 ID 的前 12 位：
>
> ```bash
> --log-opt tag='{{ with split .ImageName ":" }}{{join . "_"}}{{end}}-{{.ID}}'
> ```
>
> 输出类似：`alpine_latest-bf0072049c76`

### awslogs-force-flush-interval-seconds（强制刷新间隔秒）

`awslogs` 驱动会周期性地将日志刷新到 CloudWatch。

`awslogs-force-flush-interval-seconds` 用于调整刷新间隔，单位为秒；默认值为 5 秒。

### awslogs-max-buffered-events（最大缓冲事件数）

`awslogs` 驱动会对日志进行缓冲。

`awslogs-max-buffered-events` 用于调整缓冲区的事件数量；默认值为 4K。

## 凭证

要使用 `awslogs` 日志驱动，必须向 Docker 守护进程提供 AWS 凭证。可通过环境变量 `AWS_ACCESS_KEY_ID`、
`AWS_SECRET_ACCESS_KEY`、`AWS_SESSION_TOKEN`，或默认的 AWS 共享凭证文件（root 用户的 `~/.aws/credentials`），
或（当守护进程运行在 EC2 上时）通过 EC2 实例角色提供。

凭证所附的 IAM 策略必须允许 `logs:CreateLogStream` 与 `logs:PutLogEvents` 权限，例如：

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": ["logs:CreateLogStream", "logs:PutLogEvents"],
      "Effect": "Allow",
      "Resource": "*"
    }
  ]
}
```
