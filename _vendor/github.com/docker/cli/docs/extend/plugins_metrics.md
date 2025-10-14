---
title: Docker 指标采集插件
description: "指标插件。"
keywords: "Examples, Usage, plugins, docker, documentation, user guide, metrics"
---

Docker 以 Prometheus 格式暴露内部指标。指标插件（metrics plugins）通过在预定义路径提供一个 Unix 套接字，使插件可以一致地抓取这些指标。

> [!NOTE]
> 虽然指标的插件接口已非实验性质，但具体的指标名称与指标标签仍被视为实验性质，未来版本可能变更。

## 创建指标插件

当前你必须在插件的 `config.json` 中将 `PropagatedMount` 设置为 `/run/docker`。这使插件在完成配置之后，仍能从 Docker 接收更新的挂载（即绑定挂载的套接字）。

## MetricsCollector 协议

指标插件必须在 `config.json` 中注册声明实现了 `MetricsCollector` 接口。

在 Unix 平台上，套接字位于插件 rootfs 内的 `/run/docker/metrics.sock`。

`MetricsCollector` 需要实现两个端点：

### `MetricsCollector.StartMetrics`

通知插件：指标套接字现已可用于抓取。

请求：

```json
{}
```

该请求无负载。

响应：

```json
{
  "Err": ""
}
```

如果本次请求发生错误，请在响应的 `Err` 字段中填入错误消息；若没有错误，可以返回空对象（`{}`）或让 `Err` 为空字符串。错误只会被记录到日志中。

### `MetricsCollector.StopMetrics`

通知插件：指标套接字不再可用（例如守护进程正在关闭）。

请求：

```json
{}
```

该请求无负载。

响应：

```json
{
  "Err": ""
}
```

如果本次请求发生错误，请在响应的 `Err` 字段中填入错误消息；若没有错误，可以返回空对象（`{}`）或让 `Err` 为空字符串。错误只会被记录到日志中。
