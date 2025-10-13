---
description: 了解如何使用 Go 模板格式化日志输出
keywords: docker, 日志, 驱动, syslog, Fluentd, gelf, journald
title: 自定义日志驱动的输出
aliases:
  - /engine/reference/logging/log_tags/
  - /engine/admin/logging/log_tags/
  - /config/containers/logging/log_tags/
---

`tag` 日志选项用于指定标识容器日志消息的标签格式。默认情况下，系统会使用容器 ID 的前 12 个字符。若要覆盖该行为，请指定 `tag` 选项：

```console
$ docker run --log-driver=fluentd --log-opt fluentd-address=myhost.local:24224 --log-opt tag="mailer"
```

在为标签赋值时，Docker 支持使用一些特殊的模板占位符：

| 占位符              | 说明                                                 |
| ------------------ | ---------------------------------------------------- |
| `{{.ID}}`          | 容器 ID 的前 12 个字符。                              |
| `{{.FullID}}`      | 完整的容器 ID。                                       |
| `{{.Name}}`        | 容器名称。                                            |
| `{{.ImageID}}`     | 容器所用镜像 ID 的前 12 个字符。                      |
| `{{.ImageFullID}}` | 容器所用镜像的完整 ID。                                |
| `{{.ImageName}}`   | 容器所用镜像名称。                                    |
| `{{.DaemonName}}`  | Docker 程序名称（`docker`）。                         |

例如，指定 `--log-opt tag="{{.ImageName}}/{{.Name}}/{{.ID}}"` 后，`syslog` 的日志行类似：

```text
Aug  7 18:33:19 HOSTNAME hello-world/foobar/5790672ab6a0[9103]: Hello from Docker.
```

在容器启动时，系统会设置标签中的 `container_name` 字段与 `{{.Name}}`。如果你使用 `docker rename` 重命名容器，新的名称不会反映到已有的日志消息中，这些消息仍会使用原始的容器名。
