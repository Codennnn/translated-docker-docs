---
description: 在单个容器中运行多个进程的实践
keywords: docker, supervisor, process management
title: 在容器中运行多个进程
weight: 20
aliases:
  - /articles/using_supervisord/
  - /engine/admin/multi-service_container/
  - /engine/admin/using_supervisord/
  - /engine/articles/using_supervisord/
  - /config/containers/multi-service_container/
---

一个容器的主进程由 `Dockerfile` 末尾的 `ENTRYPOINT` 和/或 `CMD` 指定。最佳实践是“单容器单服务”，即用多个容器拆分关注点。某些服务会派生出多个子进程（例如 Apache 会启动多个工作进程）。在容器中存在多个进程并非不可以，但为了充分发挥 Docker 的价值，应避免让单个容器承担应用的多个职责。可以通过自定义网络与共享卷将多个容器连接起来。

容器的主进程需要对其启动的所有子进程负责。在某些情况下，主进程设计不佳，无法在容器退出时正确“收割”（终止）子进程。如果存在这种情况，可以在运行容器时使用 `--init` 选项。该选项会将一个极小的 init 进程作为容器主进程，用于在容器退出时收割所有子进程。相较于在容器内运行完整的 `sysvinit` 或 `systemd`，这种方式更为轻量且合适。

如果确有需要在一个容器内运行多项服务，可考虑以下方式：

## Use a wrapper script

将要执行的命令写入一个包装脚本（可包含必要的测试与调试信息），并将该脚本作为 `CMD` 运行。下面是一个简单示例。首先是包装脚本：

```bash
#!/bin/bash

# 启动第一个进程
./my_first_process &

# 启动第二个进程
./my_second_process &

# 等待任一子进程退出
wait -n

# 以最先退出的子进程状态码作为脚本退出码
exit $?
```

接下来是 Dockerfile：

```dockerfile
# syntax=docker/dockerfile:1
FROM ubuntu:latest
COPY my_first_process my_first_process
COPY my_second_process my_second_process
COPY my_wrapper_script.sh my_wrapper_script.sh
CMD ./my_wrapper_script.sh
```

## Use Bash job controls

如果你有一个需要先启动并保持运行的主进程，同时还需要临时运行其他进程（例如与主进程交互），可以利用 Bash 的作业控制。首先是包装脚本：

```bash
#!/bin/bash

# 打开 bash 的作业控制
set -m

# 启动主进程并置于后台
./my_main_process &

# 启动辅助进程
./my_helper_process

# my_helper_process 可能需要等待主进程准备就绪
# 再执行其工作并返回


# 将主进程重新带回前台并保持运行
fg %1
```

```dockerfile
# syntax=docker/dockerfile:1
FROM ubuntu:latest
COPY my_main_process my_main_process
COPY my_helper_process my_helper_process
COPY my_wrapper_script.sh my_wrapper_script.sh
CMD ./my_wrapper_script.sh
```

## 使用进程管理器

也可以使用如 `supervisord` 之类的进程管理器。与前述方案相比，这需要在镜像中打包 `supervisord` 及其配置（或使用已包含 `supervisord` 的基础镜像），并一并纳入其所管理的应用。随后启动 `supervisord` 来统一管理这些进程。

下面的 Dockerfile 展示了这种做法。示例假设以下文件位于构建上下文根目录：

- `supervisord.conf`
- `my_first_process`
- `my_second_process`

```dockerfile
# syntax=docker/dockerfile:1
FROM ubuntu:latest
RUN apt-get update && apt-get install -y supervisor
RUN mkdir -p /var/log/supervisor
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf
COPY my_first_process my_first_process
COPY my_second_process my_second_process
CMD ["/usr/bin/supervisord"]
```

如果希望两个进程的 `stdout` 和 `stderr` 都输出到容器日志中，可以在 `supervisord.conf` 中添加：

```ini
[supervisord]
nodaemon=true
logfile=/dev/null
logfile_maxbytes=0

[program:app]
stdout_logfile=/dev/fd/1
stdout_logfile_maxbytes=0
redirect_stderr=true
```
