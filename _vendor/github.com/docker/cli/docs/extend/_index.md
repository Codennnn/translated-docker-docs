---
title: Docker Engine 托管插件系统
linkTitle: Docker Engine 插件
description: 使用托管插件系统开发与使用插件
keywords: "API, Usage, plugins, documentation, developer"
aliases:
  - "/engine/extend/plugins_graphdriver/"
---

- [安装与使用插件](index.md#installing-and-using-a-plugin)
- [开发插件](index.md#developing-a-plugin)
- [调试插件](index.md#debugging-plugins)

Docker Engine 的插件系统允许你通过 Docker Engine 安装、启动、停止与移除插件。

关于旧版（非托管）插件，请参阅《[了解传统的 Docker Engine 插件](legacy_plugins.md)》。

> [!NOTE]
> 目前 Windows 守护进程不支持 Docker Engine 托管插件。

## 安装与使用插件 {#installing-and-using-a-plugin}

插件以 Docker 镜像的形式分发，可托管在 Docker Hub 或私有仓库上。

安装插件请使用 `docker plugin install`，该命令会从 Docker Hub 或你的私有仓库拉取插件；如需授权权限/能力，会提示你确认；安装完成后自动启用插件。

使用 `docker plugin ls` 查看已安装插件的状态。成功启动的插件会在输出中显示为启用状态（enabled）。

安装完成后，你可以在其他 Docker 操作中使用该插件，例如创建卷。

下面示例安装 [`rclone` 插件](https://rclone.org/docker/)、验证其已启用，并用它创建一个数据卷。

> [!NOTE]
> 以下示例仅用于教学演示。

1. 准备前置目录。默认情况下，它们必须存在于宿主机的以下位置：

   - `/var/lib/docker-plugins/rclone/config`：保留给 `rclone.conf` 配置文件，即便文件不存在也必须创建该目录。
   - `/var/lib/docker-plugins/rclone/cache`：保存插件状态文件与可选的 VFS 缓存。

2. 安装 `rclone` 插件。

   ```console
   $ docker plugin install rclone/docker-volume-rclone --alias rclone

   Plugin "rclone/docker-volume-rclone" is requesting the following privileges:
    - network: [host]
    - mount: [/var/lib/docker-plugins/rclone/config]
    - mount: [/var/lib/docker-plugins/rclone/cache]
    - device: [/dev/fuse]
    - capabilities: [CAP_SYS_ADMIN]
   Do you grant the above permissions? [y/N] 
   ```

   该插件请求如下 5 项权限：

   - 访问 `host` 网络
   - 访问并挂载上述前置目录以存储：
      - Rclone 配置文件
      - 临时缓存数据
   - 访问 FUSE（用户态文件系统）设备。Rclone 需要 FUSE 将远端存储挂载为本地文件系统。
   - 需要 `CAP_SYS_ADMIN` 能力，以便插件运行 `mount` 命令。

2. 在 `docker plugin ls` 的输出中检查插件是否已启用。

   ```console
   $ docker plugin ls

   ID                    NAME                      DESCRIPTION                                ENABLED
   aede66158353          rclone:latest             Rclone volume plugin for Docker            true
   ```

3. 使用该插件创建一个数据卷。
   本示例将主机 `1.2.3.4` 上的 `/remote` 目录挂载到名为 `rclonevolume` 的卷中。

   该数据卷随后可以被容器挂载使用。

   ```console
   $ docker volume create \
     -d rclone \
     --name rclonevolume \
     -o type=sftp \
     -o path=remote \
     -o sftp-host=1.2.3.4 \
     -o sftp-user=user \
     -o "sftp-password=$(cat file_containing_password_for_remote_host)"
   ```

4. 验证卷是否创建成功。

   ```console
   $ docker volume ls

   DRIVER              NAME
   rclone         rclonevolume
   ```

5. 启动一个使用 `rclonevolume` 的容器。

   ```console
   $ docker run --rm -v rclonevolume:/data busybox ls /data

   <content of /remote on machine 1.2.3.4>
   ```

6. 删除该卷 `rclonevolume`

   ```console
   $ docker volume rm rclonevolume

   sshvolume
   ```

禁用插件请使用 `docker plugin disable`。若要彻底移除，使用 `docker plugin remove`。更多命令与选项，参阅[命令行参考](https://docs.docker.com/reference/cli/docker/)。

## 开发插件 {#developing-a-plugin}

#### rootfs 目录

`rootfs` 目录代表插件的根文件系统。本示例中，它由一个 Dockerfile 构建而来：

> [!NOTE]
> 插件文件系统内必须包含 `/run/docker/plugins` 目录，Docker 才能与插件通信。

```console
$ git clone https://github.com/vieux/docker-volume-sshfs
$ cd docker-volume-sshfs
$ docker build -t rootfsimage .
$ id=$(docker create rootfsimage true) # id was cd851ce43a403 when the image was created
$ sudo mkdir -p myplugin/rootfs
$ sudo docker export "$id" | sudo tar -x -C myplugin/rootfs
$ docker rm -vf "$id"
$ docker rmi rootfsimage
```

#### config.json 文件

`config.json` 描述了插件。参见[插件配置参考](config.md)。

如下是一个示例 `config.json`：

```json
{
  "description": "sshFS plugin for Docker",
  "documentation": "https://docs.docker.com/engine/extend/plugins/",
  "entrypoint": ["/docker-volume-sshfs"],
  "network": {
    "type": "host"
  },
  "interface": {
    "types": ["docker.volumedriver/1.0"],
    "socket": "sshfs.sock"
  },
  "linux": {
    "capabilities": ["CAP_SYS_ADMIN"]
  }
}
```

该插件属于卷驱动（volume driver）。它需要 `host` 网络与 `CAP_SYS_ADMIN` 能力；依赖入口点 `/docker-volume-sshfs`，并通过 `/run/docker/plugins/sshfs.sock` 套接字与 Docker Engine 通信。该插件无运行时参数。

#### 创建插件

运行 `docker plugin create <plugin-name> ./path/to/plugin/data` 可创建新插件，其中数据目录需包含插件配置文件 `config.json` 与位于子目录 `rootfs` 下的根文件系统。

创建完成后，插件 `<plugin-name>` 会出现在 `docker plugin ls` 中。可以通过 `docker plugin push <plugin-name>` 将插件推送到远端仓库。

## 调试插件 {#debugging-plugins}

插件的标准输出会重定向到 dockerd 日志。这类日志条目带有 `plugin=<ID>` 后缀。以下展示了针对插件 ID `f52a3df433b9aceee436eaada0752f5797aab1de47e5485f1690a073b860ff62` 的一些命令，以及它们在 Docker 守护进程日志中的对应记录。

```console
$ docker plugin install tiborvass/sample-volume-plugin

INFO[0036] Starting...       Found 0 volumes on startup  plugin=f52a3df433b9aceee436eaada0752f5797aab1de47e5485f1690a073b860ff62
```

```console
$ docker volume create -d tiborvass/sample-volume-plugin samplevol

INFO[0193] Create Called...  Ensuring directory /data/samplevol exists on host...  plugin=f52a3df433b9aceee436eaada0752f5797aab1de47e5485f1690a073b860ff62
INFO[0193] open /var/lib/docker/plugin-data/local-persist.json: no such file or directory  plugin=f52a3df433b9aceee436eaada0752f5797aab1de47e5485f1690a073b860ff62
INFO[0193]                   Created volume samplevol with mountpoint /data/samplevol  plugin=f52a3df433b9aceee436eaada0752f5797aab1de47e5485f1690a073b860ff62
INFO[0193] Path Called...    Returned path /data/samplevol  plugin=f52a3df433b9aceee436eaada0752f5797aab1de47e5485f1690a073b860ff62
```

```console
$ docker run -v samplevol:/tmp busybox sh

INFO[0421] Get Called...     Found samplevol                plugin=f52a3df433b9aceee436eaada0752f5797aab1de47e5485f1690a073b860ff62
INFO[0421] Mount Called...   Mounted samplevol              plugin=f52a3df433b9aceee436eaada0752f5797aab1de47e5485f1690a073b860ff62
INFO[0421] Path Called...    Returned path /data/samplevol  plugin=f52a3df433b9aceee436eaada0752f5797aab1de47e5485f1690a073b860ff62
INFO[0421] Unmount Called... Unmounted samplevol            plugin=f52a3df433b9aceee436eaada0752f5797aab1de47e5485f1690a073b860ff62
```

#### 使用 runc 获取日志文件并进入插件环境

使用 `runc`（Docker 默认容器运行时）调试插件，可将插件日志重定向到文件并收集。

```console
$ sudo runc --root /run/docker/runtime-runc/plugins.moby list

ID                                                                 PID         STATUS      BUNDLE                                                                                                                                       CREATED                          OWNER
93f1e7dbfe11c938782c2993628c895cf28e2274072c4a346a6002446c949b25   15806       running     /run/docker/containerd/daemon/io.containerd.runtime.v1.linux/moby-plugins/93f1e7dbfe11c938782c2993628c895cf28e2274072c4a346a6002446c949b25   2018-02-08T21:40:08.621358213Z   root
9b4606d84e06b56df84fadf054a21374b247941c94ce405b0a261499d689d9c9   14992       running     /run/docker/containerd/daemon/io.containerd.runtime.v1.linux/moby-plugins/9b4606d84e06b56df84fadf054a21374b247941c94ce405b0a261499d689d9c9   2018-02-08T21:35:12.321325872Z   root
c5bb4b90941efcaccca999439ed06d6a6affdde7081bb34dc84126b57b3e793d   14984       running     /run/docker/containerd/daemon/io.containerd.runtime.v1.linux/moby-plugins/c5bb4b90941efcaccca999439ed06d6a6affdde7081bb34dc84126b57b3e793d   2018-02-08T21:35:12.321288966Z   root
```

```console
$ sudo runc --root /run/docker/runtime-runc/plugins.moby exec 93f1e7dbfe11c938782c2993628c895cf28e2274072c4a346a6002446c949b25 cat /var/log/plugin.log
```

如果插件内置 Shell，可按如下方式进入：

```console
$ sudo runc --root /run/docker/runtime-runc/plugins.moby exec -t 93f1e7dbfe11c938782c2993628c895cf28e2274072c4a346a6002446c949b25 sh
```

#### 使用 curl 调试插件套接字问题

要验证 Docker 守护进程与插件通信所用的 API 套接字是否响应，可使用 curl。下面示例在宿主机上对卷与网络插件发起 API 调用（以 curl 7.47.0 为例），以确保插件正在监听该套接字。对于正常工作的插件，这些基础请求应能成功。注意：主机上插件的套接字位于 `/var/run/docker/plugins/<pluginID>`。

```console
$ curl -H "Content-Type: application/json" -XPOST -d '{}' --unix-socket /var/run/docker/plugins/e8a37ba56fc879c991f7d7921901723c64df6b42b87e6a0b055771ecf8477a6d/plugin.sock http:/VolumeDriver.List

{"Mountpoint":"","Err":"","Volumes":[{"Name":"myvol1","Mountpoint":"/data/myvol1"},{"Name":"myvol2","Mountpoint":"/data/myvol2"}],"Volume":null}
```

```console
$ curl -H "Content-Type: application/json" -XPOST -d '{}' --unix-socket /var/run/docker/plugins/45e00a7ce6185d6e365904c8bcf62eb724b1fe307e0d4e7ecc9f6c1eb7bcdb70/plugin.sock http:/NetworkDriver.GetCapabilities

{"Scope":"local"}
```

当使用 curl 7.5 及以上版本时，URL 形式应为 `http://hostname/APICall`，其中 `hostname` 是安装插件的主机名，`APICall` 为插件 API 的调用路径。

例如：`http://localhost/VolumeDriver.List`
