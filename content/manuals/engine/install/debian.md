---
description: 了解如何在 Debian 上安装 Docker Engine。本文涵盖多种安装方式、卸载方法与后续步骤。
keywords: requirements, apt, installation, debian, install, uninstall, install debian, docker engine, install docker engine, upgrade, update
title: 在 Debian 上安装 Docker Engine
linkTitle: Debian
weight: 20
toc_max: 4
aliases:
- /engine/installation/debian/
- /engine/installation/linux/debian/
- /engine/installation/linux/docker-ce/debian/
- /install/linux/docker-ce/debian/
download-url-base: https://download.docker.com/linux/debian
---

开始在 Debian 上使用 Docker Engine 之前，请先确认你已[满足先决条件](#prerequisites)，然后按照[安装步骤](#installation-methods)进行安装。

## 先决条件

### 防火墙限制

> [!WARNING]
>
> 在安装 Docker 之前，请务必了解以下安全注意事项与防火墙兼容性限制。

- 如果你使用 ufw 或 firewalld 管理防火墙设置，请注意：当你通过 Docker 暴露容器端口时，这些端口会绕过你的防火墙规则。更多信息参见
  [Docker 与 ufw](/manuals/engine/network/packet-filtering-firewalls.md#docker-and-ufw)。
- Docker 仅兼容 `iptables-nft` 与 `iptables-legacy`。在安装了 Docker 的系统上，使用 `nft` 创建的防火墙规则不受支持。
  请确保你的防火墙规则集使用 `iptables` 或 `ip6tables` 创建，并将其添加到 `DOCKER-USER` 链。参见
  [数据包过滤与防火墙](/manuals/engine/network/packet-filtering-firewalls.md)。

### 操作系统要求

要安装 Docker Engine，需要以下任一 Debian 版本：

- Debian Trixie 13（stable）
- Debian Bookworm 12（oldstable）
- Debian Bullseye 11（oldoldstable）

Debian 版 Docker Engine 兼容 x86_64（或 amd64）、armhf（arm/v7）、arm64 与 ppc64le（ppc64el）架构。

### 卸载旧版本

安装 Docker Engine 之前，需要卸载任何可能冲突的软件包。

你的发行版可能提供了非官方的 Docker 软件包，可能与 Docker 官方软件包发生冲突。请在安装官方版本之前卸载这些软件包。

需要卸载的非官方软件包包括：

- `docker.io`
- `docker-compose`
- `docker-doc`
- `podman-docker`

此外，Docker Engine 依赖 `containerd` 与 `runc`。Docker Engine 通过 `containerd.io` 一并打包这些依赖。
如果你之前安装过 `containerd` 或 `runc`，请卸载它们，以避免与 Docker Engine 打包版本产生冲突。

运行以下命令卸载所有冲突软件包：

```console
$ for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done
```

`apt-get` 可能会提示这些软件包均未安装。

位于 `/var/lib/docker/` 的镜像、容器、卷与网络在卸载 Docker 时不会自动删除。
如果你希望进行“干净安装”，并清理已有数据，请参见[卸载 Docker Engine](#uninstall-docker-engine)。

## 安装方式

根据你的需求，你可以使用不同方式安装 Docker Engine：

- Docker Engine 随 [Docker Desktop for Linux](/manuals/desktop/setup/install/linux/_index.md) 一同提供。
  这是最简单、最快速的上手方式。

- 通过 [Docker 的 `apt` 仓库](#install-using-the-repository) 进行配置与安装。

- [手动安装](#install-from-a-package)，并由你手动管理升级。

- 使用[便捷脚本](#install-using-the-convenience-script)。仅推荐用于测试与开发环境。

{{% include "engine-license.md" %}}

### 通过 `apt` 仓库安装 {#install-using-the-repository}

在新主机上首次安装 Docker Engine 之前，需要先配置 Docker 的 `apt` 仓库。之后即可从该仓库安装与更新 Docker。

1. 配置 Docker 的 `apt` 仓库。

   ```bash
   # Add Docker's official GPG key:
   sudo apt-get update
   sudo apt-get install ca-certificates curl
   sudo install -m 0755 -d /etc/apt/keyrings
   sudo curl -fsSL {{% param "download-url-base" %}}/gpg -o /etc/apt/keyrings/docker.asc
   sudo chmod a+r /etc/apt/keyrings/docker.asc

   # Add the repository to Apt sources:
   echo \
     "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] {{% param "download-url-base" %}} \
     $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
     sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
   sudo apt-get update
   ```

   > [!NOTE]
   >
   > 如果你使用衍生发行版（例如 Kali Linux），可能需要替换命令中用于打印发行版代号的部分：
   >
   > ```console
   > $(. /etc/os-release && echo "$VERSION_CODENAME")
   > ```
   >
   > 将其替换为对应 Debian 发行版的代号，例如 `bookworm`。

2. 安装 Docker 软件包。

   {{< tabs >}}
   {{< tab name="Latest" >}}

   安装最新版本：

   ```console
   $ sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
   ```

   {{< /tab >}}
   {{< tab name="Specific version" >}}

   若需安装特定版本，先列出仓库中可用的版本：

   ```console
   # List the available versions:
   $ apt-cache madison docker-ce | awk '{ print $3 }'

   5:{{% param "docker_ce_version" %}}-1~debian.12~bookworm
   5:{{% param "docker_ce_version_prev" %}}-1~debian.12~bookworm
   ...
   ```

   选择目标版本并安装：

   ```console
   $ VERSION_STRING=5:{{% param "docker_ce_version" %}}-1~debian.12~bookworm
   $ sudo apt-get install docker-ce=$VERSION_STRING docker-ce-cli=$VERSION_STRING containerd.io docker-buildx-plugin docker-compose-plugin
   ```

   {{< /tab >}}
   {{< /tabs >}}

    > [!NOTE]
    >
    > 安装完成后，Docker 服务会自动启动。你可以通过以下命令验证 Docker 是否正在运行：
    > 
    > ```console
    > $ sudo systemctl status docker
    > ```
    >
    > 部分系统可能禁用了自动启动，此时需要手动启动：
    >
    > ```console
    > $ sudo systemctl start docker
    > ```

3. 通过运行 `hello-world` 镜像验证安装是否成功：

   ```console
   $ sudo docker run hello-world
   ```

   该命令会下载一个测试镜像并在容器中运行。容器运行后会打印确认信息，然后退出。

至此，你已经成功安装并启动了 Docker Engine。

{{% include "root-errors.md" %}}

#### 升级 Docker Engine

要升级 Docker Engine，请按照[安装说明](#install-using-the-repository)中的第 2 步执行，选择你希望安装的新版本。

### 通过安装包进行安装

如果无法使用 Docker 的 `apt` 仓库安装 Docker Engine，你可以下载与你发行版对应的 `deb` 安装包并手动安装。每次升级 Docker Engine 时，都需要下载新的安装包。

<!-- markdownlint-disable-next-line -->
1. 访问 [`{{% param "download-url-base" %}}/dists/`]({{% param "download-url-base" %}}/dists/)。

2. 在列表中选择你的 Debian 版本。

3. 进入 `pool/stable/`，选择适用的架构（`amd64`、`armhf`、`arm64` 或 `s390x`）。

4. 下载以下 Docker Engine、CLI、containerd 与 Docker Compose 的 `deb` 安装包：

   - `containerd.io_<version>_<arch>.deb`
   - `docker-ce_<version>_<arch>.deb`
   - `docker-ce-cli_<version>_<arch>.deb`
   - `docker-buildx-plugin_<version>_<arch>.deb`
   - `docker-compose-plugin_<version>_<arch>.deb`

5. 安装这些 `.deb` 包。请将以下示例中的路径替换为你下载 Docker 包的位置。

   ```console
   $ sudo dpkg -i ./containerd.io_<version>_<arch>.deb \
     ./docker-ce_<version>_<arch>.deb \
     ./docker-ce-cli_<version>_<arch>.deb \
     ./docker-buildx-plugin_<version>_<arch>.deb \
     ./docker-compose-plugin_<version>_<arch>.deb
   ```

    > [!NOTE]
    >
    > 安装完成后，Docker 服务会自动启动。使用以下命令验证 Docker 是否在运行：
    > 
    > ```console
    > $ sudo systemctl status docker
    > ```
    >
    > 如果你的系统禁用了该行为，请手动启动：
    >
    > ```console
    > $ sudo systemctl start docker
    > ```

6. 通过运行 `hello-world` 镜像验证安装是否成功：

   ```console
   $ sudo docker run hello-world
   ```

   该命令会下载一个测试镜像并在容器中运行。容器运行后会打印确认信息，然后退出。

至此，你已经成功安装并启动了 Docker Engine。

{{% include "root-errors.md" %}}

#### 升级 Docker Engine

要升级 Docker Engine，请下载新版安装包并重复[安装流程](#install-from-a-package)，将命令指向新文件。

{{% include "install-script.md" %}}

## 卸载 Docker Engine

1. 卸载 Docker Engine、CLI、containerd 与 Docker Compose 软件包：

   ```console
   $ sudo apt-get purge docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin docker-ce-rootless-extras
   ```

2. 主机上的镜像、容器、卷或自定义配置文件不会自动删除。若要删除所有镜像、容器与卷：

   ```console
   $ sudo rm -rf /var/lib/docker
   $ sudo rm -rf /var/lib/containerd
   ```

3. 移除源列表与密钥环

   ```console
   $ sudo rm /etc/apt/sources.list.d/docker.list
   $ sudo rm /etc/apt/keyrings/docker.asc
   ```

任何你修改过的配置文件需要手动删除。

## 进一步阅读

- 继续阅读 [Linux 安装后的步骤](linux-postinstall.md)。
