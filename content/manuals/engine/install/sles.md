---
description: 关于 Docker Engine 在 SLES 上的可用性说明。针对 SLES 的 s390x 架构已不再提供 Docker 软件包。
keywords: sles, install, uninstall, upgrade, update, s390x, ibm-z, not supported, unavailable
title: SLES（s390x）上的 Docker Engine
linkTitle: SLES (s390x)
weight: 70
toc_max: 4
aliases:
- /ee/docker-ee/sles/
- /ee/docker-ee/suse/
- /engine/installation/linux/docker-ce/sles/
- /engine/installation/linux/docker-ee/sles/
- /engine/installation/linux/docker-ee/suse/
- /engine/installation/linux/sles/
- /engine/installation/linux/SUSE/
- /engine/installation/linux/suse/
- /engine/installation/sles/
- /engine/installation/SUSE/
- /install/linux/docker-ce/sles/
- /install/linux/docker-ee/sles/
- /install/linux/docker-ee/suse/
- /install/linux/sles/
- /installation/sles/
---

## Docker Engine 不再适用于 SLES

> [!IMPORTANT]
>
> 面向 **s390x** 架构（IBM Z）的 SUSE Linux Enterprise Server（SLES）已**不再提供** Docker Engine 软件包。

IBM 已决定停止为 SLES s390x 系统构建与提供 Docker Engine 软件包。
Docker 公司从未直接构建过这些软件包，仅参与过其分发。

## 这意味着什么

- SLES s390x 上不再提供新的 Docker Engine 安装
- 现有安装可继续运行，但不再获得更新
- 不再提供新版本或安全更新
- SLES s390x 的 Docker 软件包仓库不再维护

## 如果你当前已安装 Docker

如果你的 SLES s390x 系统上已安装 Docker Engine：

- 现有安装将继续可用
- 将不再获得自动更新
- 请结合你的容器化需求做好相应规划
- 请充分考虑在缺乏更新情况下运行软件的安全影响

## 进一步阅读

如对该决策或替代方案有疑问，请联系 IBM 支持。

