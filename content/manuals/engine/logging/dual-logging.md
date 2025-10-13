---
description: >
  了解在使用第三方日志解决方案时，如何在本地读取容器日志。
keywords: >
  docker, 日志, 驱动, 双重日志, dual logging, 缓存, 环形缓冲区,
  配置
title: 使用远程日志驱动配合 docker logs 读取日志
aliases:
  - /config/containers/logging/dual-logging/
---

## 概览

无论配置了哪种日志驱动或插件，你都可以使用 `docker logs` 命令读取容器日志。Docker Engine 会使用 [`local`](drivers/local.md)
日志驱动作为本地缓存，用于读取容器的最新日志。这被称为“双重日志（dual logging）”。默认情况下，该缓存开启了日志文件轮换，每个容器在压缩前最多保留 5 个、每个 20 MB 的日志文件。

你可以在[配置选项](#configuration-options)中自定义这些默认值，或参阅[禁用双重日志缓存](#disable-the-dual-logging-cache)来关闭该功能。

## 前提条件

当所配置的日志驱动不支持读取日志时，Docker Engine 会自动启用双重日志。

下面的示例展示了在未启用与启用双重日志情况下，运行 `docker logs` 命令的表现差异：

### 未启用双重日志能力

当容器配置了 `splunk` 等远程日志驱动且关闭了双重日志时，本地尝试读取容器日志会报错：

- 步骤 1：配置 Docker 守护进程

  ```console
  $ cat /etc/docker/daemon.json
  {
    "log-driver": "splunk",
    "log-opts": {
      "cache-disabled": "true",
      ... (options for "splunk" logging driver)
    }
  }
  ```

- 步骤 2：启动容器

  ```console
  $ docker run -d busybox --name testlog top
  ```

- 步骤 3：读取容器日志

  ```console
  $ docker logs 7d6ac83a89a0
  Error response from daemon: configured logging driver does not support reading
  ```

### 启用双重日志能力

启用双重日志缓存后，即使底层日志驱动不支持读取日志，`docker logs` 也可以读取日志。下面的示例展示了将 `splunk` 作为默认远程日志驱动并启用双重日志缓存的守护进程配置：

- 步骤 1：配置 Docker 守护进程

  ```console
  $ cat /etc/docker/daemon.json
  {
    "log-driver": "splunk",
    "log-opts": {
      ... (options for "splunk" logging driver)
    }
  }
  ```

- 步骤 2：启动容器

  ```console
  $ docker run -d busybox --name testlog top
  ```

- 步骤 3：读取容器日志

  ```console
  $ docker logs 7d6ac83a89a0
  2019-02-04T19:48:15.423Z [INFO]  core: marked as sealed
  2019-02-04T19:48:15.423Z [INFO]  core: pre-seal teardown starting
  2019-02-04T19:48:15.423Z [INFO]  core: stopping cluster listeners
  2019-02-04T19:48:15.423Z [INFO]  core: shutting down forwarding rpc listeners
  2019-02-04T19:48:15.423Z [INFO]  core: forwarding rpc listeners stopped
  2019-02-04T19:48:15.599Z [INFO]  core: rpc listeners successfully shut down
  2019-02-04T19:48:15.599Z [INFO]  core: cluster listeners successfully shut down
  ```

> [!NOTE]
>
> 对于支持读取日志的日志驱动（如 `local`、`json-file`、`journald`），在双重日志能力提供前后，功能并无差异。在这类驱动下，`docker logs` 在两种场景中都可读取日志。

### 配置选项

双重日志缓存支持与[`local` 日志驱动](drivers/local.md)相同的配置项，但需要添加 `cache-` 前缀。你可以为单个容器指定这些选项，也可以通过[守护进程配置文件](/reference/cli/dockerd/#daemon-configuration-file)为新容器设置默认值。

默认情况下，缓存开启了日志轮换，每个容器在压缩前最多保留 5 个、每个 20MB 的日志文件。你可以使用下述配置项自定义这些默认值。

| 选项              | 默认值     | 说明                                                                                                                                              |
| :--------------- | :-------- | :------------------------------------------------------------------------------------------------------------------------------------------------ |
| `cache-disabled` | `"false"` | 是否禁用本地缓存。以字符串形式传递布尔值（`true`、`1`、`0`、`false`）。                                                                               |
| `cache-max-size` | `"20m"`   | 触发轮换前缓存文件的最大尺寸。为正整数加单位修饰符（`k`、`m`、`g`）。                                                                                 |
| `cache-max-file` | `"5"`     | 允许存在的缓存文件最大数量。若轮换产生了多余文件，将删除最旧的文件。为正整数。                                                                          |
| `cache-compress` | `"true"`  | 是否对轮换后的日志文件启用压缩。以字符串形式传递布尔值（`true`、`1`、`0`、`false`）。                                                                  |

## 禁用双重日志缓存

使用 `cache-disabled` 选项可以禁用双重日志缓存。当日志只通过远程日志系统读取，且无需使用 `docker logs` 进行本地调试时，禁用缓存有助于节省存储空间。

你可以在[守护进程配置文件](/reference/cli/dockerd/#daemon-configuration-file)中为单个容器禁用缓存，或为新创建的容器设置为默认禁用。

下面的示例在守护进程配置文件中将 [`splunk`](drivers/splunk.md) 设为默认日志驱动，并禁用缓存：

```console
$ cat /etc/docker/daemon.json
{
  "log-driver": "splunk",
  "log-opts": {
    "cache-disabled": "true",
    ... (options for "splunk" logging driver)
  }
}
```

> [!NOTE]
>
> 对于支持读取日志的日志驱动（如 `local`、`json-file`、`journald`），不会使用双重日志，因此禁用该选项不会产生影响。

## 限制

- 如果容器使用的日志驱动或插件将日志远程发送，且出现网络问题，则不会写入本地缓存。
- 若写入 `logdriver` 失败（例如文件系统已满、写权限被移除等），则缓存写入也会失败，并在守护进程日志中记录；写入缓存不会重试。
- 在默认配置下，为避免文件写入过慢导致阻塞容器的标准 IO，缓存使用了环形缓冲区，可能会导致部分日志丢失。管理员需要在守护进程停止时进行修复。
