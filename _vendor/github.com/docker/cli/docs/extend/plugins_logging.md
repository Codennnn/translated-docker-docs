---
title: Docker 日志驱动插件
description: "日志驱动插件。"
keywords: "Examples, Usage, plugins, docker, documentation, user guide, logging"
---

本文介绍 Docker 的日志驱动（logging driver）插件。

日志驱动允许用户将容器日志转发到其他服务进行处理。Docker 提供了若干内置日志驱动，但内置驱动不可能覆盖所有用例。通过插件机制，Docker 无需在主代码库中内置各类服务的客户端库，也能支持更广泛的日志服务。更多信息参见[插件文档](legacy_plugins.md)。

## 创建日志插件

日志插件的主接口与其他插件类型相同，均使用 JSON+HTTP 的 RPC 协议。可参考这个[示例](https://github.com/cpuguy83/docker-log-driver-test)插件，作为日志插件的参考实现。该示例封装了内置的 `jsonfilelog` 日志驱动。

## LogDriver 协议

日志插件必须在插件激活时注册为 `LogDriver`。激活后，用户即可将该插件指定为日志驱动。

日志插件必须实现两个 HTTP 端点：

### `/LogDriver.StartLogging`

通知插件：某个容器即将启动，插件应开始接收该容器的日志。

日志将通过请求中指定的文件进行流式传输。在 Linux 上，该文件是一个 FIFO。当前 Windows 不支持日志插件。

请求：

```json
{
  "File": "/path/to/file/stream",
  "Info": {
          "ContainerID": "123456"
  }
}
```

`File` 是需要被消费的日志流路径。每次调用 `StartLogging` 都应提供不同的文件路径，即便该容器此前已被该插件记录过日志。该文件由 Docker 创建，名称为随机生成。

`Info` 是被记录容器的详情。其定义较为宽松，但由以下结构体定义：

```go
type Info struct {
	Config              map[string]string
	ContainerID         string
	ContainerName       string
	ContainerEntrypoint string
	ContainerArgs       []string
	ContainerImageID    string
	ContainerImageName  string
	ContainerCreated    time.Time
	ContainerEnv        []string
	ContainerLabels     map[string]string
	LogPath             string
	DaemonName          string
}
```

`ContainerID` 在该结构体中一定会提供，但其它字段可能为空或缺失。

响应：

```json
{
  "Err": ""
}
```

如果本次请求发生错误，请在响应的 `Err` 字段中填入错误消息；若没有错误，可以返回空对象（`{}`）或让 `Err` 为空字符串。

此时驱动应开始从传入的文件中消费日志消息。如果消息未被及时消费，容器在写入其标准 IO 流时可能会发生阻塞。

日志流消息以 Protocol Buffers 编码。其 protobuf 定义位于
[moby 仓库](https://github.com/moby/moby/blob/master/api/types/plugins/logdriver/entry.proto)。

由于 Protocol Buffers 本身不包含自分隔能力，你需要按如下流格式从字节流中解码：

```text
[size][message]
```

其中 `size` 为 4 字节大端（big endian）的无符号 32 位整数，表示下一条消息的大小；`message` 是实际的日志条目。

参考的 Go 语言编码/解码实现见[此处](https://github.com/docker/docker/blob/master/api/types/plugins/logdriver/io.go)。

### `/LogDriver.StopLogging`

通知插件：停止从指定文件收集日志。收到响应后，Docker 会删除该文件。你必须确保在响应本请求前已消费完该流中的所有日志，否则可能导致日志丢失。

注意：对该端点的请求并不意味着容器已被删除，仅表示容器已停止。

请求：

```json
{
  "File": "/path/to/file/stream"
}
```

响应：

```json
{
  "Err": ""
}
```

如果本次请求发生错误，请在响应的 `Err` 字段中填入错误消息；若没有错误，可以返回空对象（`{}`）或让 `Err` 为空字符串。

## 可选端点

日志插件可以实现两个额外的日志端点：

### `/LogDriver.Capabilities`

定义日志驱动的能力。若希望 Docker 利用这些能力，你必须实现该端点。

请求：

```json
{}
```

响应：

```json
{
  "ReadLogs": true
}
```

支持的能力：

- `ReadLogs`：告知 Docker 此插件支持向客户端回读日志。声明支持 `ReadLogs` 的插件必须实现 `/LogDriver.ReadLogs` 端点。

### `/LogDriver.ReadLogs`

向客户端回读日志。在用户执行 `docker logs <container>` 时会用到该端点。

要让 Docker 使用此端点，插件必须在调用 `/LogDriver.Capabilities` 时声明相应能力。

请求：

```json
{
  "ReadConfig": {},
  "Info": {
    "ContainerID": "123456"
  }
}
```

`ReadConfig` 为读取选项，其 Go 结构体定义如下：

```go
type ReadConfig struct {
	Since  time.Time
	Tail   int
	Follow bool
}
```

- `Since`：最早应返回的日志时间。
- `Tail`：读取的行数（类似命令 `tail -n 10`）。
- `Follow`：在现有日志读取完后，是否继续跟随输出新日志。

`Info` 与 `/LogDriver.StartLogging` 中定义的类型相同，用于确定要读取的日志集合。

响应：

```text
{{ log stream }}
```

响应应返回按与插件从 Docker 接收日志相同的格式编码的日志流。
