---
title: 增强容器隔离常见问题
linkTitle: 常见问题
description: 关于增强容器隔离的常见问题解答
keywords: 增强容器隔离, 常见问题, 故障排除, docker desktop
toc_max: 2
aliases:
 - /desktop/hardened-desktop/enhanced-container-isolation/faq/
 - /security/for-admins/hardened-desktop/enhanced-container-isolation/faq/
weight: 40
---

{{< summary-bar feature_name="Hardened Docker Desktop" >}}

本页回答了关于增强容器隔离（ECI）的常见问题，这些问题在主要文档中未涵盖。

## 启用 ECI 后，我需要改变使用 Docker 的方式吗？

不需要。ECI 通过创建更安全的容器在后台自动工作。您可以继续使用所有现有的 Docker 命令、工作流程和开发工具，无需任何更改。

## 所有容器工作负载都能与 ECI 良好协作吗？

大多数容器工作负载在启用 ECI 后都能正常运行。但是，一些需要特定内核级别访问的高级工作负载可能无法正常工作。有关哪些工作负载受影响的详细信息，请参阅 [ECI 限制](/manuals/enterprise/security/hardened-desktop/enhanced-container-isolation/limitations.md)。

## 为什么不直接限制 `--privileged` 标志的使用？

特权容器有合法用途，如 Docker-in-Docker、Kubernetes-in-Docker 和访问硬件设备。ECI 提供了更好的解决方案，允许这些高级工作负载安全运行，同时防止它们危及 Docker Desktop VM。

## ECI 会影响容器性能吗？

ECI 对容器性能的影响很小。唯一的例外是执行大量 `mount` 和 `umount` 系统调用的容器，因为这些调用会被 Sysbox 运行时检查以确保安全。大多数开发工作负载不会注意到明显的性能差异。

## 启用 ECI 后，我可以覆盖容器运行时吗？


不可以。启用 ECI 后，所有容器都使用 Sysbox 运行时，无论是否有任何 `--runtime` 标志：

```console
$ docker run --runtime=runc alpine echo "test"
# 这仍然使用 sysbox-runc，而不是 runc
```

`--runtime` 标志被忽略，以防止用户通过在 Docker Desktop VM 中以真正的 root 用户身份运行容器来绕过 ECI 安全性。

## ECI 是否保护在启用它之前创建的容器？

不会。ECI 只保护在启用它之后创建的容器。在启用 ECI 之前，请删除现有容器：

```console
$ docker stop $(docker ps -q)
$ docker rm $(docker ps -aq)
```

有关更多详细信息，请参阅 [启用增强容器隔离](/manuals/enterprise/security/hardened-desktop/enhanced-container-isolation/enable-eci.md)。

## ECI 保护哪些容器？

ECI 保护因容器类型和 Docker Desktop 版本而异：

### 始终受保护

- 使用 `docker run` 和 `docker create` 创建的容器
- 使用 `docker-container` 构建驱动程序的容器

### 版本相关

- Docker Build：在 Docker Desktop 4.30+ 中受保护（WSL 2 除外）
- Kubernetes：在使用 kind 供应器的 Docker Desktop 4.38+ 中受保护

### 不受保护

- Docker 扩展
- Docker 调试容器
- 使用 Kubeadm 供应器的 Kubernetes

有关完整详细信息，请参阅 [ECI 限制](/manuals/enterprise/security/hardened-desktop/enhanced-container-isolation/limitations.md)。

## 启用 ECI 后，我可以挂载 Docker 套接字吗？

默认情况下，不可以。出于安全考虑，ECI 阻止 Docker 套接字绑定挂载。但是，您可以为受信任的镜像（如 Testcontainers）配置例外。

有关配置详细信息，请参阅 [配置 Docker 套接字例外](/manuals/enterprise/security/hardened-desktop/enhanced-container-isolation/config.md)。

## ECI 限制哪些绑定挂载？

ECI 限制 Docker Desktop VM 目录的绑定挂载，但允许在 Docker Desktop 设置中配置的主机目录挂载。
