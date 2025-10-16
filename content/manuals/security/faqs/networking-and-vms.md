---
title: 网络与虚拟机常见问题
linkTitle: 网络与虚拟机
description: 关于 Docker Desktop 网络与虚拟化安全的常见问题
keywords: Docker Desktop 网络, 虚拟化, Hyper-V, WSL 2, 网络安全, 防火墙
weight: 30
tags: [FAQ]
aliases:
- /faq/security/networking-and-vms/
---

## 如何限制容器访问互联网？

Docker Desktop 没有内置此功能，但你可以在宿主机使用进程级防火墙。针对用户态进程 `com.docker.vpnkit` 设置规则，以控制其可连接的目的地（例如 DNS 允许列表、包过滤）以及允许使用的端口和协议。

在企业环境中，可考虑使用 [离线容器（Air-gapped containers）](/manuals/enterprise/security/hardened-desktop/air-gapped-containers.md)，它为容器提供网络访问控制能力。

## 能否对容器网络流量应用防火墙规则？

可以。Docker Desktop 通过用户态进程（`com.docker.vpnkit`）提供网络连接，该进程会继承启动它的用户所配置的限制，例如防火墙规则、VPN 设置和 HTTP 代理属性。

## 启用 Hyper-V 的 Windows 版 Docker Desktop 是否允许创建其他虚拟机？

不会。服务中将 `DockerDesktopVM` 名称进行了硬编码，无法使用 Docker Desktop 创建或操作其他虚拟机。

## Docker Desktop 如何在 Hyper-V 与 WSL 2 下实现网络隔离？

Docker Desktop 在 WSL 2（发行版为 `docker-desktop`）与 Hyper-V（`DockerDesktopVM`）下使用相同的虚拟机进程。宿主机与虚拟机之间通过 `AF_VSOCK` 管理程序套接字（共享内存）通信，而不是通过网络交换机或网络接口。所有宿主机侧的网络操作均由 `com.docker.vpnkit.exe` 与 `com.docker.backend.exe` 进程使用标准 TCP/IP 套接字完成。

了解更多，请参阅 [Docker Desktop 网络在底层如何工作](https://www.docker.com/blog/how-docker-desktop-networking-works-under-the-hood/)。
