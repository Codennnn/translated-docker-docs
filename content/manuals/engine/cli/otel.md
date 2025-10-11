---
title: Docker CLI 的 OpenTelemetry 支持
description: 了解如何为 Docker 命令行采集 OpenTelemetry 指标
keywords: otel, opentelemetry, telemetry, traces, tracing, metrics, logs
aliases:
  - /config/otel/
---

{{< summary-bar feature_name="Docker CLI OpenTelemetry" >}}

Docker CLI 支持 [OpenTelemetry](https://opentelemetry.io/docs/) 插桩，用于在命令调用时上报指标。该功能默认关闭。你可以配置一个上报端点，让 CLI 将指标发送至该端点，以便采集 `docker` 命令调用信息，获取更深入的使用洞察。

指标导出为自愿启用（opt-in），你可通过指定采集器的目标地址来控制数据的发送位置。

## 什么是 OpenTelemetry？

OpenTelemetry（简称 OTel）是一个开放的可观测性框架，用于生成与管理遥测数据，包括链路、指标与日志。OpenTelemetry 与具体厂商或工具无关，可配合多种可观测性后端使用。

Docker CLI 对 OpenTelemetry 的支持意味着 CLI 能按照 OTel 规范定义的协议与约定，发出所发生事件的相关信息。

## 工作原理

Docker CLI 默认不会输出任何遥测数据。只有当你在系统中设置了相应环境变量时，CLI 才会尝试将 OpenTelemetry 指标发送到你指定的端点。

```bash
DOCKER_CLI_OTEL_EXPORTER_OTLP_ENDPOINT=<endpoint>
```

该变量用于指定 OpenTelemetry 采集器的端点，`docker` CLI 的调用指标会发送至该端点。要接收这些数据，你需要在该地址上运行一个 OTel 采集器。

采集器的职责是接收遥测数据、进行处理，并导出至后端进行存储。后端可以是 Prometheus、InfluxDB 等多种实现。

部分后端自带可视化功能；或者你也可以使用独立的前端工具（如 Grafana）生成更直观的图表。

## 设置

要开始为 Docker CLI 采集遥测数据，你需要：

- 设置 `DOCKER_CLI_OTEL_EXPORTER_OTLP_ENDPOINT` 环境变量，指向某个 OTel 采集器端点
- 运行一个 OTel 采集器，接收来自 CLI 命令调用的信号
- 部署一个后端用于存储采集器收到的数据

下面的 Docker Compose 文件可快速启动一组服务，包含一个 CLI 可发送指标的 OTel 采集器，以及一个从采集器抓取指标的 Prometheus 后端。

```yaml {collapse=true,title=compose.yaml}
name: cli-otel
services:
  prometheus:
    image: prom/prometheus
    command:
      - "--config.file=/etc/prometheus/prom.yml"
    ports:
      # 在 localhost:9091 暴露 Prometheus 前端
      - 9091:9090
    restart: always
    volumes:
      # 将 Prometheus 数据存储到卷中：
      - prom_data:/prometheus
      # 挂载 prom.yml 配置文件
      - ./prom.yml:/etc/prometheus/prom.yml
  otelcol:
    image: otel/opentelemetry-collector
    restart: always
    depends_on:
      - prometheus
    ports:
      - 4317:4317
    volumes:
      # 挂载 otelcol.yml 配置文件
      - ./otelcol.yml:/etc/otelcol/config.yaml

volumes:
  prom_data:
```

该栈假设 `compose.yaml` 同目录下存在如下两个配置文件：

- ```yaml {collapse=true,title=otelcol.yml}
  # 通过 gRPC 与 HTTP 接收信号
  receivers:
    otlp:
      protocols:
        grpc:
        http:

  # 暴露供 Prometheus 抓取的端点
  exporters:
    prometheus:
      endpoint: "0.0.0.0:8889"

  service:
    pipelines:
      metrics:
        receivers: [otlp]
        exporters: [prometheus]
  ```

- ```yaml {collapse=true,title=prom.yml}
  # 配置 Prometheus 抓取 OpenTelemetry 采集器端点
  scrape_configs:
    - job_name: "otel-collector"
      scrape_interval: 1s
      static_configs:
        - targets: ["otelcol:8889"]
  ```

准备好上述文件后：

1. 启动 Docker Compose 服务：

   ```console
   $ docker compose up
   ```

2. 配置 Docker CLI 将遥测导出至 OpenTelemetry 采集器：

   ```console
   $ export DOCKER_CLI_OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4317
   ```

3. 运行任意 `docker` 命令，触发 CLI 向 OpenTelemetry 采集器发送指标信号：

   ```console
   $ docker version
   ```

4. 查看 CLI 产生的指标：打开 Prometheus 表达式浏览器，访问 <http://localhost:9091/graph>。

5. 在“**Query**”输入框中输入 `command_time_milliseconds_total`，执行查询以查看遥测数据。

## 可用指标

Docker CLI 当前导出一个指标 `command.time`，表示命令执行耗时（毫秒）。该指标包含以下属性：

- `command.name`：命令名称
- `command.status.code`：命令的退出码
- `command.stderr.isatty`：若标准错误（stderr）附加到 TTY 则为 true
- `command.stdin.isatty`：若标准输入（stdin）附加到 TTY 则为 true
- `command.stdout.isatty`：若标准输出（stdout）附加到 TTY 则为 true
