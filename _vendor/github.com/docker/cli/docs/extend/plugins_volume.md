---
title: Docker 卷插件
description: "如何使用外部卷插件管理数据"
keywords: "Examples, Usage, volume, docker, data, volumes, plugin, api"
---

Docker Engine 的卷插件（volume plugins）使 Engine 部署能够与外部存储系统（例如 Amazon EBS）集成，并让数据卷在单个 Docker 主机的生命周期之外依然持久存在。更多信息参见[插件文档](legacy_plugins.md)。

## 变更日志

### 1.13.0

- 当作为 v2 插件架构的一部分使用时，插件返回的路径中涉及的挂载点，必须挂载在插件配置的 `PropagatedMount` 指定的目录之下（[#26398](https://github.com/docker/docker/pull/26398)）。

### 1.12.0

- 在 `VolumeDriver.Get` 响应中新增 `Status` 字段（[#21006](https://github.com/docker/docker/pull/21006#)）。
- 新增 `VolumeDriver.Capabilities`，用于获取卷驱动的能力（[#22077](https://github.com/docker/docker/pull/22077)）。

### 1.10.0

- 新增 `VolumeDriver.Get`，用于获取卷的详细信息（[#16534](https://github.com/docker/docker/pull/16534)）。
- 新增 `VolumeDriver.List`，用于列出该驱动所管理的所有卷（[#16534](https://github.com/docker/docker/pull/16534)）。

### 1.8.0

- 初始支持卷驱动插件（[#14659](https://github.com/docker/docker/pull/14659)）。

## 命令行变更

要让容器访问一个卷，可在 `docker container run` 命令上使用 `--volume` 与 `--volume-driver` 选项。`--volume`（或 `-v`）选项接收一个卷名与挂载路径，`--volume-driver` 选项接收驱动类型。

```console
$ docker volume create --driver=flocker volumename

$ docker container run -it --volume volumename:/data busybox sh
```

### `--volume`

`--volume`（或 `-v`）选项的取值格式为 `<volume_name>:<mountpoint>`，两部分使用冒号（`:`）分隔。

- 卷名是该卷的可读名称，且不能以 `/` 开头。本文其余部分将其记为 `volume_name`。
- `Mountpoint` 是卷可用的位置：在 v1 中为宿主机路径，在 v2 中为插件内路径。

### `volumedriver`

与 `volumename` 搭配指定 `volumedriver`，可以使用诸如 [Flocker](https://github.com/ScatterHQ/flocker) 之类的插件来管理单机之外的卷，例如 EBS 上的卷。

## 创建 VolumeDriver

容器创建端点（`/containers/create`）接受一个 `string` 类型的 `VolumeDriver` 字段，用于指定驱动名称。如果未指定，则默认为 `"local"`（本地卷的默认驱动）。

## 卷插件协议

当插件在激活时注册为 `VolumeDriver` 后，必须向 Docker 守护进程提供宿主机文件系统上的可写路径。Docker 守护进程会将这些路径提供给容器使用，并通过将这些路径 bind-mount 到容器中来提供卷。

> [!NOTE]
> 卷插件不应向 `/var/lib/docker/` 目录写入数据（包括 `/var/lib/docker/volumes`）。`/var/lib/docker/` 目录保留给 Docker 使用。

### `/VolumeDriver.Create`

请求：

```json
{
    "Name": "volume_name",
    "Opts": {}
}
```

指示插件按用户给定的卷名创建卷。实际在文件系统上创建该卷可以延后至调用 `Mount` 时再进行。`Opts` 是从用户请求透传的、驱动特定的可选项映射。

响应：

```json
{
    "Err": ""
}
```

发生错误时在字符串中返回错误信息。

### `/VolumeDriver.Remove`

请求：

```json
{
    "Name": "volume_name"
}
```

从磁盘上删除指定卷。当用户执行 `docker rm -v` 以移除与容器关联的卷时，会触发此请求。

响应：

```json
{
    "Err": ""
}
```

发生错误时在字符串中返回错误信息。

### `/VolumeDriver.Mount`

请求：

```json
{
    "Name": "volume_name",
    "ID": "b87d7442095999a92b65b3d9691e697b61713829cc0ffd1bb72e4ccd51aa4d6c"
}
```

Docker 要求插件按用户指定的卷名提供一个卷。`Mount` 在每次容器启动时调用一次。如果同一 `volume_name` 被多次请求，插件可能需要追踪每次新的挂载请求，并在首次挂载时进行资源准备，在最后一次对应的卸载请求时回收资源。

`ID` 是请求挂载方的唯一标识。

响应：

- v1

  ```json
  {
      "Mountpoint": "/path/to/directory/on/host",
      "Err": ""
  }
  ```

- v2

  ```json
  {
      "Mountpoint": "/path/under/PropagatedMount",
      "Err": ""
  }
  ```

`Mountpoint` 为卷可用的位置：v1 为宿主机路径，v2 为插件内路径。

`Err` 为空字符串或包含错误信息。

### `/VolumeDriver.Path`

请求：

```json
{
    "Name": "volume_name"
}
```

请求给定 `volume_name` 对应卷的路径。

响应：

- v1

  ```json
  {
      "Mountpoint": "/path/to/directory/on/host",
      "Err": ""
  }
  ```

- v2

  ```json
  {
      "Mountpoint": "/path/under/PropagatedMount",
      "Err": ""
  }
  ```

返回卷可用的位置（v1：宿主机路径；v2：插件内路径），以及发生错误时的错误字符串。

`Mountpoint` 为可选字段；若未提供，Docker 可能在之后再次查询。

### `/VolumeDriver.Unmount`

请求：

```json
{
    "Name": "volume_name",
    "ID": "b87d7442095999a92b65b3d9691e697b61713829cc0ffd1bb72e4ccd51aa4d6c"
}
```

Docker 不再使用该命名卷。`Unmount` 在每次容器停止时调用一次。插件可以据此判断是否可以回收该卷的资源。

`ID` 是请求挂载方的唯一标识。

响应：

```json
{
    "Err": ""
}
```

发生错误时在字符串中返回错误信息。

### `/VolumeDriver.Get`

请求：

```json
{
    "Name": "volume_name"
}
```

获取关于 `volume_name` 的信息。

响应：

- v1

  ```json
  {
    "Volume": {
      "Name": "volume_name",
      "Mountpoint": "/path/to/directory/on/host",
      "Status": {}
    },
    "Err": ""
  }
  ```

- v2

  ```json
  {
    "Volume": {
      "Name": "volume_name",
      "Mountpoint": "/path/under/PropagatedMount",
      "Status": {}
    },
    "Err": ""
  }
  ```

发生错误时在字符串中返回错误信息。`Mountpoint` 与 `Status` 为可选字段。

### /VolumeDriver.List

请求：

```json
{}
```

获取该插件已登记的卷列表。

响应：

- v1

  ```json
  {
    "Volumes": [
      {
        "Name": "volume_name",
        "Mountpoint": "/path/to/directory/on/host"
      }
    ],
    "Err": ""
  }
  ```

- v2

  ```json
  {
    "Volumes": [
      {
        "Name": "volume_name",
        "Mountpoint": "/path/under/PropagatedMount"
      }
    ],
    "Err": ""
  }
  ```

发生错误时在字符串中返回错误信息。`Mountpoint` 为可选字段。

### /VolumeDriver.Capabilities

请求：

```json
{}
```

获取该驱动支持的能力列表。

驱动并非必须实现 `Capabilities`；若未实现，则使用默认值。

响应：

```json
{
  "Capabilities": {
    "Scope": "global"
  }
}
```

支持的作用域（Scope）为 `global` 与 `local`。对于任何其它取值，都将忽略而使用 `local`。`Scope` 使集群管理器可以用不同方式处理卷。例如，`global` 表示集群管理器仅需创建一次该卷（而非在每台 Docker 主机上都创建）。未来可能会加入更多能力。
