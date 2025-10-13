---
description: 了解如何写入、查看并配置容器日志
keywords: docker, 日志
title: 查看容器日志
linkTitle: 日志与指标
weight: 70
aliases:
  - /engine/admin/logging/
  - /engine/admin/logging/view_container_logs/
  - /config/containers/logging/
---

`docker logs` 命令用于查看正在运行的容器输出的日志信息。`docker service logs` 则显示参与某个服务的所有容器的日志信息。日志记录的内容与格式几乎完全取决于容器的启动命令。

默认情况下，`docker logs` 与 `docker service logs` 显示的输出与在终端交互式运行该命令时的输出一致。Unix/Linux 命令在运行时通常会打开三个 I/O 流：`STDIN`、`STDOUT` 与 `STDERR`。`STDIN` 是输入流（来自键盘或其他命令的输入），`STDOUT` 通常是命令的正常输出，`STDERR` 通常用于输出错误信息。默认情况下，`docker logs` 会显示命令的 `STDOUT` 与 `STDERR`。关于 Linux I/O 的更多信息，参见：[I/O 重定向（Linux Documentation Project）](https://tldp.org/LDP/abs/html/io-redirection.html)。

在某些情况下，如果不进行额外设置，`docker logs` 可能不会显示有用的信息：

- 如果你使用的[日志驱动](configure.md)会将日志发送到文件、外部主机、数据库或其他后端，并且禁用了[“双重日志”](dual-logging.md)，则 `docker logs` 可能不会有用。
- 如果你的镜像运行的是非交互式进程（如 Web 服务器或数据库），该应用可能会把输出写入自身的日志文件，而不是写到 `STDOUT`/`STDERR`。

对于第一种情况，日志会通过其他方式处理，你可以不使用 `docker logs`。对于第二种情况，官方 `nginx` 镜像与官方 Apache `httpd` 镜像分别提供了不同的解决方案。

官方 `nginx` 镜像通过创建符号链接，将 `/var/log/nginx/access.log` 指向 `/dev/stdout`，将 `/var/log/nginx/error.log` 指向 `/dev/stderr`，从而覆盖原始日志文件并把日志发送到对应的特殊设备。参见其
[Dockerfile](https://github.com/nginxinc/docker-nginx/blob/8921999083def7ba43a06fabd5f80e4406651353/mainline/jessie/Dockerfile#L21-L23)。

官方 `httpd` 镜像通过修改应用配置，将正常输出直接写到 `/proc/self/fd/1`（即 `STDOUT`），将错误输出写到 `/proc/self/fd/2`（即 `STDERR`）。参见其
[Dockerfile](https://github.com/docker-library/httpd/blob/b13054c7de5c74bbaa6d595dbe38969e6d4f860c/2.2/Dockerfile#L72-L75)。

## 进一步阅读

- 配置[日志驱动](configure.md)。
- 编写 [Dockerfile](/reference/dockerfile.md)。
