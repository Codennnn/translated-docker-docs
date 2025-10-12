---
description: 通过软件源或手动方式在 Linux 上安装 Docker Compose 插件的分步指南。
keywords: install docker compose linux, docker compose plugin, docker-compose-plugin linux, docker compose v2, docker compose manual install, linux docker compose
toc_max: 3
title: 安装 Docker Compose 插件
linkTitle: 插件
aliases:
- /compose/compose-plugin/
- /compose/compose-linux/
- /compose/install/compose-plugin/
weight: 10
---

本页介绍如何在 Linux 上通过命令行安装 Docker Compose 插件。

在 Linux 上安装 Docker Compose 插件，你可以选择：
- [使用 Docker 官方软件源](#install-using-the-repository)
- [手动安装](#install-the-plugin-manually)

> [!NOTE]
>
> 以下步骤假设你已安装 Docker Engine 与 Docker CLI，现仅需安装 Docker Compose 插件。

## 使用软件源安装 {#install-using-the-repository}

1. 设置软件源。针对不同发行版的说明：

    [Ubuntu](/manuals/engine/install/ubuntu.md#install-using-the-repository) |
    [CentOS](/manuals/engine/install/centos.md#set-up-the-repository) |
    [Debian](/manuals/engine/install/debian.md#install-using-the-repository) |
    [Raspberry Pi OS](/manuals/engine/install/raspberry-pi-os.md#install-using-the-repository) |
    [Fedora](/manuals/engine/install/fedora.md#set-up-the-repository) |
    [RHEL](/manuals/engine/install/rhel.md#set-up-the-repository)。

2. 更新软件索引并安装最新版本的 Docker Compose：

    * Ubuntu 与 Debian：

        ```console
        $ sudo apt-get update
        $ sudo apt-get install docker-compose-plugin
        ```
    * 基于 RPM 的发行版：

        ```console
        $ sudo yum update
        $ sudo yum install docker-compose-plugin
        ```

3.  通过查看版本验证是否安装成功。

    ```console
    $ docker compose version
    ```

### 更新 Docker Compose

如需更新 Docker Compose 插件，执行：

* Ubuntu 与 Debian：

    ```console
    $ sudo apt-get update
    $ sudo apt-get install docker-compose-plugin
    ```
* 基于 RPM 的发行版：

    ```console
    $ sudo yum update
    $ sudo yum install docker-compose-plugin
    ```

## 手动安装插件 {#install-the-plugin-manually}

> [!WARNING]
>
> 手动安装不会自动更新。为便于后续维护，建议优先使用 Docker 软件源方式。

1.  下载并安装 Docker Compose CLI 插件：

    ```console
    $ DOCKER_CONFIG=${DOCKER_CONFIG:-$HOME/.docker}
    $ mkdir -p $DOCKER_CONFIG/cli-plugins
    $ curl -SL https://github.com/docker/compose/releases/download/{{% param "compose_version" %}}/docker-compose-linux-x86_64 -o $DOCKER_CONFIG/cli-plugins/docker-compose
    ```

    上述命令会为当前用户在 `$HOME` 目录下下载并安装最新发布版的 Docker Compose。

    如需：
    - 在系统中为“所有用户”安装，将 `~/.docker/cli-plugins` 替换为 `/usr/local/lib/docker/cli-plugins`。
    - 安装指定版本，将 `{{% param "compose_version" %}}` 替换为目标版本号。
    - 针对不同 CPU 架构， 将 `x86_64` 替换为[所需架构](https://github.com/docker/compose/releases)。   


2. 为二进制文件添加可执行权限：

    ```console
    $ chmod +x $DOCKER_CONFIG/cli-plugins/docker-compose
    ```
    若选择为所有用户安装：

    ```console
    $ sudo chmod +x /usr/local/lib/docker/cli-plugins/docker-compose
    ```

3. 验证安装：

    ```console
    $ docker compose version
    ```

## 下一步

- [了解 Compose 的工作原理](/manuals/compose/intro/compose-application-model.md)
- [试试快速开始](/manuals/compose/gettingstarted.md)
