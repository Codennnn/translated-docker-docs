---
description: 如何卸载 Docker Compose
keywords: compose, orchestration, uninstall, uninstallation, docker, documentation
title: 卸载 Docker Compose
linkTitle: 卸载 
---

卸载 Docker Compose 的方式取决于你最初的安装方法。本文涵盖以下卸载路径：

- 通过 Docker Desktop 安装的 Docker Compose
- 作为 CLI 插件安装的 Docker Compose

## 通过 Docker Desktop 卸载 Docker Compose

如果你使用 Docker Desktop 安装了 Docker Compose，请参见《[卸载 Docker Desktop](/manuals/desktop/uninstall.md)》。

> [!WARNING]
>
> 除非该环境中另有其他 Docker 实例，否则卸载 Docker Desktop 会移除所有 Docker 组件，包括 Docker Engine、Docker CLI 与 Docker Compose。

## 卸载 Docker Compose CLI 插件

如果你通过软件包管理器安装了 Docker Compose，请执行：

在 Ubuntu 或 Debian 上：

   ```console
   $ sudo apt-get remove docker-compose-plugin
   ```
在基于 RPM 的发行版上：

   ```console
   $ sudo yum remove docker-compose-plugin
   ```

### 手动安装的情况

如果你使用 curl 手动安装了 Docker Compose，可通过删除二进制文件来移除：

   ```console
   $ rm $DOCKER_CONFIG/cli-plugins/docker-compose
   ```

### 为所有用户安装的情况

若为系统“所有用户”安装，请从系统目录中移除：

   ```console
   $ rm /usr/local/lib/docker/cli-plugins/docker-compose
   ```

> [!NOTE]
>
> 如果在执行上述任一命令时出现 **Permission denied** 错误，说明当前用户权限不足。
> 可在命令前加上 `sudo` 再次执行以强制移除。

### 查看 Compose CLI 插件的安装位置

要检查 Compose 的安装位置，使用：

```console
$ docker info --format '{{range .ClientInfo.Plugins}}{{if eq .Name "compose"}}{{.Path}}{{end}}{{end}}'
```
