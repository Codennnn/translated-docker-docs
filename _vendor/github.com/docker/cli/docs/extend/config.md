---
description: "如何使用托管插件系统开发与使用插件"
keywords: "API, Usage, plugins, documentation, developer"
title: 插件 V2 的配置 V1 版本
---

本文概述了 V0 插件配置的格式。

插件配置描述了 Docker Engine 插件的各个组成部分。
插件配置可使用以下媒体类型序列化为 JSON 格式：

| 配置类型 | 媒体类型                                 |
|------------|------------------------------------------|
| config     | `application/vnd.docker.plugin.v1+json`  |

## 配置字段说明

Config 提供了在注册表中使用 V0 插件格式时可访问的基础字段。

- `description` string

  插件的描述

- `documentation` string

  指向插件文档的链接

- `interface` PluginInterface

  插件实现的接口，结构体包含以下字段：

  - `types` string array

    `types` 表示插件当前实现了哪些接口。

    支持的类型：

    - `docker.volumedriver/1.0`

    - `docker.networkdriver/1.0`

    - `docker.ipamdriver/1.0`

    - `docker.authz/1.0`

    - `docker.logdriver/1.0`

    - `docker.metricscollector/1.0`

  - `socket` string

    与插件通信所用套接字的名称。该套接字会在 `/run/docker/plugins` 下创建。

- `entrypoint` string array

   插件入口点，参见 [`ENTRYPOINT`](https://docs.docker.com/reference/dockerfile/#entrypoint)

- `workdir` string

   插件工作目录，参见 [`WORKDIR`](https://docs.docker.com/reference/dockerfile/#workdir)

- `network` PluginNetwork

  插件的网络配置，结构体包含以下字段：

  - `type` string

    网络类型。

    支持的类型：

    - `bridge`
    - `host`
    - `none`

- `mounts` PluginMount array

  插件的挂载配置，结构体包含以下字段。
  参见 [`MOUNTS`](https://github.com/opencontainers/runtime-spec/blob/master/config.md#mounts)。

  - `name` string

    挂载名称。

  - `description` string

    挂载描述。

  - `source` string

    挂载源。

  - `destination` string

    挂载目标。

  - `type` string

    挂载类型。

  - `options` string array

    挂载选项。

- `ipchost` Boolean

   访问宿主机 IPC 命名空间。

- `pidhost` Boolean

   访问宿主机 PID 命名空间。

- `propagatedMount` string

   以 rshared 方式挂载的路径，使其子挂载对 Docker 可见。对卷插件很有用。该路径会绑定挂载到插件 rootfs 之外，以便升级时保留其内容。

- `env` PluginEnv array

  插件的环境变量，结构体包含以下字段：

  - `name` string

    环境变量名。

  - `description` string

    环境变量描述。

  - `value` string

    环境变量值。

- `args` PluginArgs

  插件的参数配置，结构体包含以下字段：

  - `name` string

    参数名。

  - `description` string

    参数描述。

  - `value` string array

    参数的取值。

- `linux` PluginLinux

  - `capabilities` string array

    插件的能力（仅 Linux）。参见[`此处`](https://github.com/opencontainers/runc/blob/master/libcontainer/SPEC.md#security)的列表。

  - `allowAllDevices` Boolean

    若从宿主机绑定挂载 `/dev`，且 allowAllDevices 为 true，则插件将拥有对宿主机所有设备的 `rwm` 访问。

  - `devices` PluginDevice array

    插件的设备（仅 Linux），结构体包含以下字段。
    参见 [`DEVICES`](https://github.com/opencontainers/runtime-spec/blob/master/config-linux.md#devices)。

    - `name` string

      设备名称。

    - `description` string

      设备描述。

    - `path` string

      设备路径。

## 配置示例

以下示例展示了 'tiborvass/sample-volume-plugin' 的插件配置：

```json
{
  "Args": {
    "Description": "",
    "Name": "",
    "Settable": null,
    "Value": null
  },
  "Description": "A sample volume plugin for Docker",
  "Documentation": "https://docs.docker.com/engine/extend/plugins/",
  "Entrypoint": [
    "/usr/bin/sample-volume-plugin",
    "/data"
  ],
  "Env": [
    {
      "Description": "",
      "Name": "DEBUG",
      "Settable": [
        "value"
      ],
      "Value": "0"
    }
  ],
  "Interface": {
    "Socket": "plugin.sock",
    "Types": [
      "docker.volumedriver/1.0"
    ]
  },
  "Linux": {
    "Capabilities": null,
    "AllowAllDevices": false,
    "Devices": null
  },
  "Mounts": null,
  "Network": {
    "Type": ""
  },
  "PropagatedMount": "/data",
  "User": {},
  "Workdir": ""
}
```
