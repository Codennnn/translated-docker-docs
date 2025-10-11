---
description: 配置 Docker 守护进程
keywords: docker, daemon, configuration
title: Docker 守护进程配置概览
linkTitle: 守护进程
weight: 60
aliases:
  - /articles/chef/
  - /articles/configuring/
  - /articles/dsc/
  - /articles/puppet/
  - /config/thirdparty/
  - /config/thirdparty/ansible/
  - /config/thirdparty/chef/
  - /config/thirdparty/dsc/
  - /config/thirdparty/puppet/
  - /engine/admin/
  - /engine/admin/ansible/
  - /engine/admin/chef/
  - /engine/admin/configuring/
  - /engine/admin/dsc/
  - /engine/admin/puppet/
  - /engine/articles/chef/
  - /engine/articles/configuring/
  - /engine/articles/dsc/
  - /engine/articles/puppet/
  - /engine/userguide/
  - /config/daemon/
---

本页介绍如何自定义 Docker 守护进程 `dockerd`。

> [!NOTE]
>
> 本页面面向手动安装 Docker Engine 的用户。若你使用 Docker Desktop，请参阅
> [设置页面](/manuals/desktop/settings-and-maintenance/settings.md#docker-engine)。

## 配置 Docker 守护进程

配置 Docker 守护进程有两种方式：

- 使用 JSON 配置文件（推荐，集中管理所有配置）。
- 在启动 `dockerd` 时通过命令行标志传参。

两者可以同时使用，但不能对同一选项既在命令行标志中声明，又在 JSON 文件中配置。
否则 Docker 守护进程将无法启动并输出错误信息。

### 配置文件

下表给出了 Docker 守护进程默认查找配置文件的位置，具体取决于系统与运行方式：

| 操作系统与方式         | 文件位置                                     |
| -------------------- | ------------------------------------------ |
| Linux（常规）         | `/etc/docker/daemon.json`                  |
| Linux（rootless）     | `~/.config/docker/daemon.json`             |
| Windows              | `C:\\ProgramData\\docker\\config\\daemon.json` |

在 rootless 模式下，守护进程会遵循 `XDG_CONFIG_HOME` 变量；若设置，该文件位于 `$XDG_CONFIG_HOME/docker/daemon.json`。

你也可以在启动时通过 `dockerd --config-file` 显式指定配置文件位置。

有关可用配置项，参阅
[dockerd 参考文档](/reference/cli/dockerd.md#daemon-configuration-file)

### 使用标志进行配置

你也可以手动启动 Docker 守护进程，并通过标志进行配置。
这在排查问题时很有用。

下面示例展示如何手动启动守护进程，使用与前文 JSON 配置等效的参数：

```console
$ dockerd --debug \
  --tls=true \
  --tlscert=/var/docker/server.pem \
  --tlskey=/var/docker/serverkey.pem \
  --host tcp://192.168.59.3:2376
```

更多可用配置项请参阅
[dockerd 参考文档](/reference/cli/dockerd.md)，或运行：

```console
$ dockerd --help
```

## 守护进程数据目录

Docker 守护进程会将数据持久化到单一目录中，其中包含与 Docker 相关的一切，
如容器、镜像、卷、服务定义与机密等。

默认数据目录：

- Linux：`/var/lib/docker`
- Windows：`C:\\ProgramData\\docker`

可以通过 `data-root` 配置项更改该目录，例如：

```json
{
  "data-root": "/mnt/docker-data"
}
```

由于守护进程的状态保存在该目录中，请确保每个守护进程使用独立的目录。
如果多个守护进程共享同一目录（例如通过 NFS 共享），将导致难以排查的错误。

## 进一步阅读

关于更多具体配置项，你可以继续阅读：

- [容器开机自启](/manuals/engine/containers/start-containers-automatically.md)
- [限制容器资源](/manuals/engine/containers/resource_constraints.md)
- [配置存储驱动](/manuals/engine/storage/drivers/select-storage-driver.md)
- [容器安全](/manuals/engine/security/_index.md)
- [为 Docker 守护进程配置代理](./proxy.md)
