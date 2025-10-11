---
description: 了解如何将 Docker 容器互相连接
keywords: Examples, Usage, user guide, links, linking, docker, documentation, examples,
  names, name, container naming, port, map, network port, network
title: 旧版容器链接（links）
aliases:
- /userguide/dockerlinks/
- /engine/userguide/networking/default_network/dockerlinks/
- /network/links/
---

> [!WARNING]
>
> `--link` 标志是 Docker 的遗留特性，未来可能被移除。除非确有必要，建议使用“用户自定义网络”在容器之间通信，而非 `--link`。需要注意的是，用户自定义网络不支持 `--link` 的“跨容器共享环境变量”，你可以通过卷等机制以更可控的方式共享环境变量。
>
> 参见[用户自定义 bridge 与默认 bridge 的差异](drivers/bridge.md#differences-between-user-defined-bridges-and-the-default-bridge)以获取替代方案。

本页介绍在默认 `bridge` 网络（安装 Docker 时自动创建）中的旧版容器链接机制。

在[Docker 网络特性](index.md)出现之前，可以用 links 让容器相互发现并安全传递信息。引入网络特性后仍可创建 links，但在默认 `bridge` 与[用户自定义网络](drivers/bridge.md#differences-between-user-defined-bridges-and-the-default-bridge)中的行为不同。

本文先简述通过端口映射进行连接，然后详述默认 `bridge` 网络中的容器链接。

## 通过端口映射进行连接

例如，使用如下命令运行一个简单的 Python Flask 应用：

```console
$ docker run -d -P training/webapp python app.py
```

> [!NOTE]
>
> 容器拥有内部网络与 IP；Docker 支持多种网络配置，详见[这里](index.md)。

容器创建时使用 `-P`，会将容器内暴露的端口自动映射到主机“临时端口范围”中的随机高位端口。随后 `docker ps` 显示，容器 5000 端口被映射到主机 49155。

```console
$ docker ps nostalgic_morse

CONTAINER ID  IMAGE                   COMMAND       CREATED        STATUS        PORTS                    NAMES
bc533791f3f5  training/webapp:latest  python app.py 5 seconds ago  Up 2 seconds  0.0.0.0:49155->5000/tcp  nostalgic_morse
```

也可以用 `-p` 将容器端口绑定到主机特定端口。如下把主机 80 映射到容器 5000：

```console
$ docker run -d -p 80:5000 training/webapp python app.py
```

这种方式限制在于：该主机端口上只能运行一个容器。

也可以指定一个不同于默认“临时端口范围”的主机端口区间来绑定容器端口：

```console
$ docker run -d -p 8000-9000:5000 training/webapp python app.py
```

这会把容器 5000 端口映射到主机 8000-9000 区间的随机可用端口。

`-p` 还有其他用法。默认将端口绑定至主机所有网卡；也可以绑定到特定接口，例如仅绑定 `localhost`：

```console
$ docker run -d -p 127.0.0.1:80:5000 training/webapp python app.py
```

这会把容器 5000 端口映射到主机 `localhost`（`127.0.0.1`）的 80 端口。

或者，仅在 `localhost` 上把容器 5000 映射到动态端口：

```console
$ docker run -d -p 127.0.0.1::5000 training/webapp python app.py
```

也可以在端口后添加 `/udp` 或 `/sctp` 来映射 UDP 与 SCTP（常见于通信协议，如 SIGTRAN、Diameter、S1AP/X2AP）端口，例如：

```console
$ docker run -d -p 127.0.0.1:80:5000/udp training/webapp python app.py
```

`docker port` 可查看当前端口绑定，也可用于确认具体绑定细节（例如是否仅绑定在 `localhost`）：

```console
$ docker port nostalgic_morse 5000

127.0.0.1:49155
```

> [!NOTE]
>
> 可以多次使用 `-p` 配置多个端口。

## 通过链接机制进行连接

> [!NOTE]
>
> 本节介绍默认 `bridge` 网络中的旧版链接功能。关于用户自定义网络的链接，参见[用户自定义 bridge 与默认 bridge 的差异](drivers/bridge.md#differences-between-user-defined-bridges-and-the-default-bridge)。

端口映射并非容器互联的唯一方式。Docker 还提供链接机制，可把多个容器关联并把连接信息从一个容器传递到另一个容器。建立链接后，目标容器可以查看源容器的部分信息。

### 命名的重要性

Docker 通过容器名称建立链接。每个容器都有自动生成的名称（如 `nostalgic_morse`），你也可以自行命名。自定义命名的好处：

1. 便于记忆：比如将 Web 容器命名为 `web`。

2. 便于引用：例如可以把 `web` 链接到 `db`。

使用 `--name` 指定容器名，例如：

```console
$ docker run -d -P --name web training/webapp python app.py
```

上述命令启动新容器并命名为 `web`；可通过 `docker ps` 查看。

```console
$ docker ps -l

CONTAINER ID  IMAGE                  COMMAND        CREATED       STATUS       PORTS                    NAMES
aed84ee21bde  training/webapp:latest python app.py  12 hours ago  Up 2 seconds 0.0.0.0:49154->5000/tcp  web
```

也可使用 `docker inspect` 获取容器名。


> [!NOTE]
>
> 容器名称必须唯一。若要复用同名，需要先删除旧容器（`docker container rm`），或在运行时使用 `--rm` 让容器停止后即删除。

## 链接间的通信

links 让容器彼此发现并安全传递信息。建立链接即在源与目标间创建通道，目标可访问源的部分数据。创建链接需要 `--link`。先创建数据库容器：

```console
$ docker run -d --name db training/postgres
```

这会从 `training/postgres` 创建名为 `db` 的容器（包含 PostgreSQL）。

删除之前的 `web`，以便用带链接的新容器替换：

```console
$ docker container rm -f web
```

现在创建新的 `web` 并链接到 `db`：

```console
$ docker run -d -P --name web --link db:db training/webapp python app.py
```

以上命令将 `web` 与 `db` 链接起来。`--link` 语法：

    --link <name or id>:alias

其中 `name` 为被链接容器名，`alias` 为链接别名；也可以省略别名：

    --link <name or id>

省略时别名与名称相同，因此也可写为：

```console
$ docker run -d -P --name web --link db training/webapp python app.py
```

然后用 `docker inspect` 查看链接：


```console
$ docker inspect -f "{{ .HostConfig.Links }}" web

[/db:/web/db]
```


可见 `web` 现已链接到 `db`（`/db:/web/db`），因此可访问 `db` 信息。

链接让“源”向“目标”提供自身信息。Docker 在容器间创建安全通道，无需对外暴露端口（启动 `db` 时未用 `-P`/`-p`）。优点是无需将源容器（此处 PostgreSQL）暴露到网络。

Docker 以两种方式向目标暴露源容器连接信息：

* 环境变量
* 更新 `/etc/hosts` 文件

### 环境变量

建立链接后，Docker 会在目标容器内基于 `--link` 自动创建环境变量，并暴露源容器中由 Docker 注入的环境变量，来源包括：

* 源容器 Dockerfile 中的 `ENV`
* 通过 `-e`、`--env`、`--env-file` 提供的变量

这些变量可用于在目标容器内以编程方式发现源容器信息。

> [!WARNING]
>
> 由 Docker 注入到某容器的环境变量会对链接到它的容器可见；若包含敏感数据，可能带来严重安全风险。

对于 `--link` 中的每个目标，Docker 会设置 `<alias>_NAME` 变量。例如 `--link db:webdb` 会在 `web` 中创建 `WEBDB_NAME=/web/webdb`。

对源容器的每个暴露端口，Docker 还会定义一组以 `<name>_PORT_<port>_<protocol>` 为前缀的变量：

该前缀包含：

* `--link` 指定的别名 `<name>`（如 `webdb`）
* 暴露端口 `<port>`
* 协议 `<protocol>`（TCP/UDP）

据此前缀，Docker 定义三类变量：

* `prefix_ADDR`：URL 中的 IP（如 `WEBDB_PORT_5432_TCP_ADDR=172.17.0.82`）
* `prefix_PORT`：URL 中的端口（如 `WEBDB_PORT_5432_TCP_PORT=5432`）
* `prefix_PROTO`：URL 中的协议（如 `WEBDB_PORT_5432_TCP_PROTO=tcp`）

若容器暴露多个端口，每个端口都会有一组变量。例如暴露 4 个端口会生成 12 个变量。

此外，Docker 会创建 `<alias>_PORT`，其值为源容器“第一个”（端口号最小）暴露端口的 URL；如 `WEBDB_PORT=tcp://172.17.0.82:5432`。若同时有 TCP/UDP，则取 TCP。

最后，Docker 还会将源容器由 Docker 注入的每个环境变量，以 `<alias>_ENV_<name>` 的形式暴露到目标容器，取值为启动源容器时的值。

回到示例，可使用 `env` 列出环境变量：

```console
$ docker run --rm --name web2 --link db:db training/webapp env

<...>
DB_NAME=/web2/db
DB_PORT=tcp://172.17.0.5:5432
DB_PORT_5432_TCP=tcp://172.17.0.5:5432
DB_PORT_5432_TCP_PROTO=tcp
DB_PORT_5432_TCP_PORT=5432
DB_PORT_5432_TCP_ADDR=172.17.0.5
<...>
```

可以看到 Docker 创建了以 `DB_` 开头的一系列变量（源自 `db` 别名），可用于配置应用连接到 `db`。该连接安全且私有，只有被链接的 `web` 能与 `db` 通信。

### 关于 Docker 环境变量的重要说明

不同于[`/etc/hosts` 文件](#updating-the-etchosts-file)，环境变量中的 IP 在源容器重启后不会自动更新。建议通过 `/etc/hosts` 的主机条目解析链接容器 IP。

这些变量仅对容器的第一个进程生效；某些守护进程（如 `sshd`）在派生 shell 时会清理它们。

### 更新 `/etc/hosts` 文件

除了环境变量，Docker 还会在 `/etc/hosts` 中为源容器添加主机条目。以下是 `web` 容器中的示例：

```console
$ docker run -t -i --rm --link db:webdb training/webapp /bin/bash

root@aed84ee21bde:/opt/webapp# cat /etc/hosts
172.17.0.7  aed84ee21bde
<...>
172.17.0.5  webdb 6e5cdeb2d300 db
```

可以看到两个相关条目：第一个为 `web` 自身（以容器 ID 作为主机名）；第二个通过链接别名引用 `db` 的 IP。除你提供的别名外，若链接容器的名称与别名不同，其容器名与主机名也会被加入到 `/etc/hosts` 对应 IP 下。可通过这些条目进行 ping：

```console
root@aed84ee21bde:/opt/webapp# apt-get install -yqq inetutils-ping
root@aed84ee21bde:/opt/webapp# ping webdb

PING webdb (172.17.0.5): 48 data bytes
56 bytes from 172.17.0.5: icmp_seq=0 ttl=64 time=0.267 ms
56 bytes from 172.17.0.5: icmp_seq=1 ttl=64 time=0.250 ms
56 bytes from 172.17.0.5: icmp_seq=2 ttl=64 time=0.256 ms
```

> [!NOTE]
>
> 示例中需要先安装 `ping`，因为容器默认不包含它。

此处我们通过主机条目 ping `db`（解析到 `172.17.0.5`）。应用亦可使用该主机条目访问 `db`。

> [!NOTE]
>
> 可将多个目标容器链接至同一源容器，例如多个不同名称的 Web 容器均指向 `db`。

当重启源容器时，已链接容器中的 `/etc/hosts` 会自动更新为新 IP，链接通信可继续进行。

```console
$ docker restart db
db

$ docker run -t -i --rm --link db:db training/webapp /bin/bash

root@aed84ee21bde:/opt/webapp# cat /etc/hosts
172.17.0.7  aed84ee21bde
<...>
172.17.0.9  db
```
