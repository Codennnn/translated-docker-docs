---
title: 在 Compose 中使用生命周期钩子
linkTitle: 使用生命周期钩子
weight: 20
description: 了解如何使用 Docker Compose 的生命周期钩子（post_start、pre_stop）定制容器行为。
keywords: docker compose lifecycle hooks, post_start, pre_stop, docker compose entrypoint, docker container stop hooks, compose hook commands
---

{{< summary-bar feature_name="Compose lifecycle hooks" >}}

## 服务的生命周期钩子

在 Docker Compose 运行容器时，会通过两个要素——[ENTRYPOINT 与 COMMAND](/manuals/engine/containers/run.md#default-command-and-options)——来控制容器启动与停止时的行为。

不过，在一些场景下，更适合将这些任务拆分给生命周期钩子处理——也就是在容器启动后立即执行，或在容器停止前执行的命令。

生命周期钩子尤其有用的一点是，即使容器出于安全原因以较低权限运行，这些钩子也可以拥有特殊权限（例如以 root 身份运行）。这意味着需要更高权限的任务可以在不降低容器整体安全性的前提下完成。

### Post-start 钩子

Post-start 钩子是在容器启动之后运行的命令，但其确切执行时机没有固定保证，尤其是在容器 `entrypoint` 执行期间无法保证钩子的触发时序。

例如：

- 使用该钩子将卷的属主改为非 root 用户（因为卷默认以 root 身份创建）。
- 在容器启动之后，使用 `chown` 将 `/data` 目录的所有权更改为用户 `1001`。

```yaml
services:
  app:
    image: backend
    user: 1001
    volumes:
      - data:/data    
    post_start:
      - command: chown -R /data 1001:1001
        user: root

volumes:
  data: {} # Docker 卷默认以 root 所有权创建
```

### Pre-stop 钩子

Pre-stop 钩子是在容器被特定命令停止之前执行的命令（例如 `docker compose down` 或通过 `Ctrl+C` 手动停止）。如果容器自身退出或被强制终止，这些钩子不会被触发。

在下面的示例中，容器停止之前会运行 `./data_flush.sh` 脚本以执行必要的清理。

```yaml
services:
  app:
    image: backend
    pre_stop:
      - command: ./data_flush.sh
```

## 参考信息

- [`post_start`](/reference/compose-file/services.md#post_start)
- [`pre_stop`](/reference/compose-file/services.md#pre_stop)
