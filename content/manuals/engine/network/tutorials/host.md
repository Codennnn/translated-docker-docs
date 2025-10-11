---
title: 使用 host 网络进行联网
description: 使用 host 网络进行联网、禁用网络隔离的教程
keywords: networking, host, standalone
aliases:
  - /network/network-tutorial-host/
---

本系列教程介绍将独立容器直接绑定到 Docker 主机网络、且不进行网络隔离的场景。更多网络主题参见[概览](/manuals/engine/network/_index.md)。

## 目标

本教程的目标是启动一个直接绑定到 Docker 主机 80 端口的 `nginx` 容器。从网络角度看，其隔离级别与直接在主机上运行 `nginx` 进程相同。但在其他方面（如存储、进程命名空间、用户命名空间），`nginx` 进程仍与主机隔离。

## 前提条件

- 需要确保 Docker 主机上的 80 端口可用。若希望 Nginx 监听其他端口，请参考[`nginx` 镜像文档](https://hub.docker.com/_/nginx/)。

- `host` 网络驱动仅适用于 Linux 主机；在 Docker Desktop 4.34 及以上版本中作为可选功能提供。要在 Docker Desktop 启用此功能，请到 **Settings** 的 **Resources** → **Network**，勾选 **Enable host networking**。

## 操作步骤

1.  以后台方式创建并启动容器。`--rm` 表示容器退出/停止后即删除；`-d` 表示以后台（detached）方式启动。

    ```console
    $ docker run --rm -d --network host --name my_nginx nginx
    ```

2.  在浏览器访问 [http://localhost:80/](http://localhost:80/)。

3.  使用以下命令检查你的网络栈：

    - 查看所有网络接口，确认未创建新的接口：

      ```console
      $ ip addr show
      ```

    - 使用 `netstat` 验证哪个进程绑定了 80 端口。需要使用 `sudo`，因为该进程归属于 Docker 守护进程用户，否则无法查看其名称或 PID。

      ```console
      $ sudo netstat -tulpn | grep :80
      ```

4.  停止容器。由于启动时使用了 `--rm`，容器会自动删除。

    ```console
    docker container stop my_nginx
    ```

## 其他网络教程

- [独立网络教程](/manuals/engine/network/tutorials/standalone.md)
- [Overlay 网络教程](/manuals/engine/network/tutorials/overlay.md)
- [Macvlan 网络教程](/manuals/engine/network/tutorials/macvlan.md)
