---
title: 安装 Docker Engine
linkTitle: 安装
weight: 10
description: 了解如何选择最适合你的方式安装 Docker Engine。该客户端-服务器
  应用可用于 Linux、Mac、Windows，亦可作为静态二进制发布。
keywords: install engine, docker engine install, install docker engine, docker engine
  installation, engine install, docker ce installation, docker ce install, engine
  installer, installing docker engine, docker server install, docker desktop vs docker engine
aliases:
- /cs-engine/
- /cs-engine/1.12/
- /cs-engine/1.12/upgrade/
- /cs-engine/1.13/
- /cs-engine/1.13/upgrade/
- /ee/docker-ee/oracle/
- /ee/supported-platforms/
- /en/latest/installation/
- /engine/installation/
- /engine/installation/frugalware/
- /engine/installation/linux/
- /engine/installation/linux/archlinux/
- /engine/installation/linux/cruxlinux/
- /engine/installation/linux/docker-ce/
- /engine/installation/linux/docker-ee/
- /engine/installation/linux/docker-ee/oracle/
- /engine/installation/linux/frugalware/
- /engine/installation/linux/gentoolinux/
- /engine/installation/linux/oracle/
- /engine/installation/linux/other/
- /engine/installation/oracle/
- /enterprise/supported-platforms/
- /install/linux/docker-ee/oracle/
---

本节介绍如何在 Linux 上安装 Docker Engine（亦称 Docker CE）。
Docker Engine 也可通过 Docker Desktop 在 Windows、macOS 和 Linux 上使用。
关于如何安装 Docker Desktop，请参见：[Docker Desktop 概览](/manuals/desktop/_index.md)。

## 受支持平台的安装步骤

点击相应平台的链接以查看安装流程。

| 平台                                           | x86_64 / amd64 | arm64 / aarch64 | arm（32 位） | ppc64le | s390x |
| :--------------------------------------------- | :------------: | :-------------: | :----------: | :-----: | :---: |
| [CentOS](centos.md)                            |       ✅       |       ✅        |              |   ✅    |       |
| [Debian](debian.md)                            |       ✅       |       ✅        |      ✅      |   ✅    |       |
| [Fedora](fedora.md)                            |       ✅       |       ✅        |              |   ✅    |       |
| [Raspberry Pi OS (32-bit)](raspberry-pi-os.md) |                |                 |      ⚠️      |         |       |
| [RHEL](rhel.md)                                |       ✅       |       ✅        |              |         |  ✅   |
| [SLES](sles.md)                                |                |                 |              |         |  ❌   |
| [Ubuntu](ubuntu.md)                            |       ✅       |       ✅        |      ✅      |   ✅    |  ✅   |
| [Binaries](binaries.md)                        |       ✅       |       ✅        |      ✅      |         |       |

### 其他 Linux 发行版

> [!NOTE]
>
> 以下说明可能有效，但 Docker 不对衍生发行版的安装进行测试或验证。

- 如果你使用基于 Debian 的发行版（如 “BunsenLabs Linux”、“Kali Linux” 或
  “LMDE”（基于 Debian 的 Mint）），请参考 [Debian](debian.md) 的安装指南，
  并将你的发行版版本替换为对应的 Debian 版本。请查阅你的发行版文档以确定
  你的衍生版对应哪个 Debian 发行版本。
- 同样地，如果你使用基于 Ubuntu 的发行版（如 “Kubuntu”、“Lubuntu” 或 “Xubuntu”），
  请参考 [Ubuntu](ubuntu.md) 的安装指南，并将你的发行版版本替换为对应的 Ubuntu 版本。
  请查阅你的发行版文档以确定你的衍生版对应哪个 Ubuntu 发行版本。
- 某些 Linux 发行版会通过其软件仓库提供 Docker Engine 的软件包。
  这些软件包由发行版的打包维护者构建和维护，可能在配置上有所差异，
  或基于修改过的源码构建。Docker 不参与这些软件包的发布，如遇到这些软件包的
  缺陷或问题，请向你的发行版问题跟踪器反馈。

Docker 提供用于手动安装 Docker Engine 的[二进制文件](binaries.md)。
这些二进制文件为静态链接，可在任意 Linux 发行版上使用。

## 发布通道

Docker Engine 提供两类更新通道：**stable** 与 **test**：

* **stable**：提供已正式发布、可用于生产的最新版本。
* **test**：提供用于发布前测试的预发布版本。

请谨慎使用 test 通道。预发布版本包含实验性与抢先体验功能，可能发生不兼容变更。

## 支持

Docker Engine 是一个开源项目，由 Moby 项目的维护者与社区成员共同支持。
Docker 不为 Docker Engine 提供官方技术支持。
Docker 为其产品提供支持，其中包括 Docker Desktop（其组件之一为 Docker Engine）。

关于该开源项目的更多信息，请参阅 [Moby 项目网站](https://mobyproject.org/)。

### 升级路径

补丁版本始终与其所属的大版本与小版本向后兼容。

### 许可

在大型企业环境（员工人数超过 250 人，或年营收超过 1,000 万美元）中，
通过 Docker Desktop 获得的 Docker Engine 若用于商业用途，则需购买
[付费订阅](https://www.docker.com/pricing/)。
Apache License, Version 2.0。完整许可请参见
[LICENSE](https://github.com/moby/moby/blob/master/LICENSE)。

## 报告安全问题

如果你发现安全问题，请尽快告知我们。

请勿创建公开 issue。请将报告私下发送至 security@docker.com。

我们非常感谢你的安全报告，Docker 会对此公开致谢。

## 开始上手

完成 Docker 的安装与设置后，你可以通过
[Docker 入门指南](/get-started/introduction/_index.md)学习基础知识。
