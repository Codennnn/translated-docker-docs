---
title: 使用 IPv6 网络
weight: 20
description: 如何在 Docker 守护进程中启用 IPv6 支持
keywords: daemon, network, networking, ipv6
aliases:
- /engine/userguide/networking/default_network/ipv6/
- /config/daemon/ipv6/
---

IPv6 仅在运行于 Linux 宿主机上的 Docker 守护进程中受支持。

## 创建 IPv6 网络

- 使用 `docker network create`：

  ```console
  $ docker network create --ipv6 ip6net
  ```

- 使用 `docker network create` 并指定 IPv6 子网：

  ```console
  $ docker network create --ipv6 --subnet 2001:db8::/64 ip6net
  ```

- 使用 Docker Compose 文件：

  ```yaml
   networks:
     ip6net:
       enable_ipv6: true
       ipam:
         config:
           - subnet: 2001:db8::/64
  ```

现在可以运行并加入 `ip6net` 网络的容器：

```console
$ docker run --rm --network ip6net -p 80:80 traefik/whoami
```

这会在 IPv6 和 IPv4 上同时发布 80 端口。
可以通过在 IPv6 回环地址上的 80 端口发起 curl 请求来验证 IPv6 连接：

```console
$ curl http://[::1]:80
Hostname: ea1cfde18196
IP: 127.0.0.1
IP: ::1
IP: 172.17.0.2
IP: 2001:db8::2
IP: fe80::42:acff:fe11:2
RemoteAddr: [2001:db8::1]:37574
GET / HTTP/1.1
Host: [::1]
User-Agent: curl/8.1.2
Accept: */*
```

## 在默认 bridge 网络上启用 IPv6

以下步骤演示在默认 bridge 网络上启用 IPv6：

1. 编辑位于 `/etc/docker/daemon.json` 的 Docker 守护进程配置文件，设置以下参数：

   ```json
   {
     "ipv6": true,
     "fixed-cidr-v6": "2001:db8:1::/64"
   }
   ```

   - `ipv6`：在默认网络上启用 IPv6。
   - `fixed-cidr-v6`：为默认 bridge 网络分配一个子网，从而启用动态 IPv6 地址分配。
   - `ip6tables`：启用额外的 IPv6 包过滤规则，提供网络隔离和端口映射。默认开启，可手动关闭。

2. 保存配置文件。
3. 重启 Docker 守护进程以使配置生效。

   ```console
   $ sudo systemctl restart docker
   ```

现在可以在默认 bridge 网络上运行容器：

```console
$ docker run --rm -p 80:80 traefik/whoami
```

这会在 IPv6 和 IPv4 上同时发布 80 端口。
你可以向 IPv6 回环地址的 80 端口发起请求来验证 IPv6 连接：

```console
$ curl http://[::1]:80
Hostname: ea1cfde18196
IP: 127.0.0.1
IP: ::1
IP: 172.17.0.2
IP: 2001:db8:1::242:ac12:2
IP: fe80::42:acff:fe12:2
RemoteAddr: [2001:db8:1::1]:35558
GET / HTTP/1.1
Host: [::1]
User-Agent: curl/8.1.2
Accept: */*
```

## 动态 IPv6 子网分配

如果你未通过 `docker network create --subnet=<your-subnet>` 为自定义网络显式配置子网，
这些网络会回退使用守护进程的默认地址池。对于在 Docker Compose 文件中创建的网络，
当 `enable_ipv6` 设为 `true` 时同样适用。

如果 Docker Engine 的 `default-address-pools` 中没有包含 IPv6 地址池，
且未指定 `--subnet`，在启用 IPv6 时将使用[唯一本地地址（ULA）][wikipedia-ipv6-ula]。
这些 `/64` 子网包含基于 Docker Engine 随机生成 ID 的 40 位全局 ID，
以保证较高的唯一性概率。

如需为动态分配使用其他 IPv6 子网池，你需要在守护进程中手动配置地址池，包含：

- 默认 IPv4 地址池
- 一个或多个你自定义的 IPv6 地址池

默认地址池配置如下：

```json
{
  "default-address-pools": [
    { "base": "172.17.0.0/16", "size": 16 },
    { "base": "172.18.0.0/16", "size": 16 },
    { "base": "172.19.0.0/16", "size": 16 },
    { "base": "172.20.0.0/14", "size": 16 },
    { "base": "172.24.0.0/14", "size": 16 },
    { "base": "172.28.0.0/14", "size": 16 },
    { "base": "192.168.0.0/16", "size": 20 }
  ]
}
```

下面示例展示了在默认值基础上新增一个 IPv6 地址池的合法配置。
该 IPv6 地址池的前缀长度为 `/56`，可提供最多 256 个 `/64` 子网。

```json
{
  "default-address-pools": [
    { "base": "172.17.0.0/16", "size": 16 },
    { "base": "172.18.0.0/16", "size": 16 },
    { "base": "172.19.0.0/16", "size": 16 },
    { "base": "172.20.0.0/14", "size": 16 },
    { "base": "172.24.0.0/14", "size": 16 },
    { "base": "172.28.0.0/14", "size": 16 },
    { "base": "192.168.0.0/16", "size": 20 },
    { "base": "2001:db8::/56", "size": 64 }
  ]
}
```

> [!NOTE]
>
> 示例中的 `2001:db8::` 为[文档保留地址][wikipedia-ipv6-reserved]，请替换为有效的 IPv6 网络。
>
> 默认 IPv4 地址池来自私有地址范围，与默认 IPv6 的 [ULA][wikipedia-ipv6-ula] 类似。

[wikipedia-ipv6-reserved]: https://en.wikipedia.org/wiki/Reserved_IP_addresses#IPv6
[wikipedia-ipv6-ula]: https://en.wikipedia.org/wiki/Unique_local_address

## Docker in Docker

在使用 `xtables`（旧版 `iptables`）而非 `nftables` 的宿主机上，
在创建 IPv6 Docker 网络之前必须加载内核模块 `ip6_tables`。该模块通常会在 Docker 启动时自动加载。

但如果你运行的是非官方最新镜像的 Docker in Docker，可能需要在宿主机上执行 `modprobe ip6_tables`。
或者，使用守护进程选项 `--ip6tables=false` 来禁用容器化 Docker Engine 的 `ip6tables`。

## 进一步阅读

- [网络概览](/manuals/engine/network/_index.md)
