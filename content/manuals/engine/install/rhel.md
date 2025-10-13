---
description: 了解如何在 RHEL 上安装 Docker Engine。本文涵盖多种安装方式、卸载方法与后续步骤。
keywords: requirements, dnf, installation, rhel, rpm, install, install docker engine, uninstall, upgrade,
  update
title: 在 RHEL 上安装 Docker Engine
linkTitle: RHEL
weight: 30
toc_max: 4
aliases:
- /ee/docker-ee/rhel/
- /engine/installation/linux/docker-ce/rhel/
- /engine/installation/linux/docker-ee/rhel/
- /engine/installation/linux/rhel/
- /engine/installation/rhel/
- /install/linux/docker-ee/rhel/
- /installation/rhel/
download-url-base: https://download.docker.com/linux/rhel
---

开始在 RHEL 上使用 Docker Engine 之前，请先确认你已[满足先决条件](#prerequisites)，然后按照[安装步骤](#installation-methods)进行安装。

## 先决条件

### 操作系统要求

要安装 Docker Engine，需要使用仍在维护期内的以下 RHEL 版本之一：

- RHEL 8
- RHEL 9
- RHEL 10

### 卸载旧版本

在安装 Docker Engine 之前，需要卸载任何可能冲突的软件包。

你的发行版可能提供了非官方的 Docker 软件包，可能与 Docker 官方软件包发生冲突。请在安装官方版本之前卸载这些软件包。

```console
$ sudo dnf remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine \
                  podman \
                  runc
```

`dnf` 可能会提示这些软件包均未安装。

位于 `/var/lib/docker/` 的镜像、容器、卷与网络在卸载 Docker 时不会自动删除。

## 安装方式

根据你的需求，可以使用不同方式安装 Docker Engine：

- 你可以[配置 Docker 仓库](#install-using-the-repository)并从中安装，便于安装与升级。
  这是推荐方式。

- 你也可以下载 RPM 包并[手动安装](#install-from-a-package)，完全手动管理升级。
  适用于无法联网（air-gapped）等场景。

- 在测试与开发环境中，你可以使用自动化的[便捷脚本](#install-using-the-convenience-script)进行安装。

{{% include "engine-license.md" %}}

### 通过 rpm 仓库安装 {#install-using-the-repository}

在新主机上首次安装 Docker Engine 前，需要先配置 Docker 仓库。之后即可从该仓库安装与更新 Docker。

#### 配置仓库

安装 `dnf-plugins-core`（提供管理 DNF 仓库的相关命令），并完成仓库配置。

```console
$ sudo dnf -y install dnf-plugins-core
$ sudo dnf config-manager --add-repo {{% param "download-url-base" %}}/docker-ce.repo
```

#### 安装 Docker Engine

1. 安装 Docker 软件包。

   {{< tabs >}}
   {{< tab name="Latest" >}}

   安装最新版本：

   ```console
   $ sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
   ```

   如果提示接受 GPG 密钥，请核对其指纹是否为
   `060A 61C5 1B55 8A7F 742B 77AA C52F EB6B 621E 9F35`，如匹配则接受。

   以上命令仅安装 Docker，不会启动。它也会创建 `docker` 组，但默认不会添加任何用户。

   {{< /tab >}}
   {{< tab name="Specific version" >}}

   若需安装特定版本，先列出仓库中可用版本：

   ```console
   $ dnf list docker-ce --showduplicates | sort -r

   docker-ce.x86_64    3:{{% param "docker_ce_version" %}}-1.el9    docker-ce-stable
   docker-ce.x86_64    3:{{% param "docker_ce_version_prev" %}}-1.el9    docker-ce-stable
   <...>
   ```

   返回的列表取决于启用的仓库，并与你的 RHEL 版本有关（例如此处的 `.el9` 后缀）。

   通过完整的软件包名安装特定版本：即软件包名（`docker-ce`）加上版本号（第二列），
   用连字符（`-`）连接。例如 `docker-ce-3:{{% param "docker_ce_version" %}}-1.el9`。

   将 `<VERSION_STRING>` 替换为目标版本并运行：

   ```console
   $ sudo dnf install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io docker-buildx-plugin docker-compose-plugin
   ```

   以上命令仅安装 Docker，不会启动。它也会创建 `docker` 组，但默认不会添加任何用户。

   {{< /tab >}}
   {{< /tabs >}}

2. 启动 Docker Engine。

   ```console
   $ sudo systemctl enable --now docker
   ```

   上述命令会将 Docker 的 systemd 服务配置为开机自启动。
   如果你不希望 Docker 随系统启动，可改用 `sudo systemctl start docker`。

3. 通过运行 `hello-world` 镜像验证安装是否成功：

   ```console
   $ sudo docker run hello-world
   ```

   该命令会下载测试镜像并在容器中运行。容器运行后会打印确认信息，然后退出。

至此，你已经成功安装并启动了 Docker Engine。

{{% include "root-errors.md" %}}

#### 升级 Docker Engine

要升级 Docker Engine，请按照[安装说明](#install-using-the-repository)执行，并选择你希望安装的新版本。

### 通过 RPM 包安装

如果无法使用 Docker 的 `rpm` 仓库安装 Docker Engine，可以下载与你发行版对应的 `.rpm` 文件并手动安装。
每次升级 Docker Engine 时，都需要下载新的安装包。

<!-- markdownlint-disable-next-line -->
1. 访问 [{{% param "download-url-base" %}}/]({{% param "download-url-base" %}}/)。

2. 在列表中选择你的 RHEL 版本。

3. 选择适用的架构（`x86_64`、`aarch64` 或 `s390x`），然后进入 `stable/Packages/`。

4. 下载以下 Docker Engine、CLI、containerd 与 Docker Compose 的 `rpm` 安装包：

   - `containerd.io-<version>.<arch>.rpm`
   - `docker-ce-<version>.<arch>.rpm`
   - `docker-ce-cli-<version>.<arch>.rpm`
   - `docker-buildx-plugin-<version>.<arch>.rpm`
   - `docker-compose-plugin-<version>.<arch>.rpm`

5. 安装 Docker Engine，并将以下路径替换为你下载这些安装包的位置。

   ```console
   $ sudo dnf install ./containerd.io-<version>.<arch>.rpm \
     ./docker-ce-<version>.<arch>.rpm \
     ./docker-ce-cli-<version>.<arch>.rpm \
     ./docker-buildx-plugin-<version>.<arch>.rpm \
     ./docker-compose-plugin-<version>.<arch>.rpm
   ```

   此时 Docker 已安装但未启动；系统已创建 `docker` 组，但未添加任何用户。

6. 启动 Docker Engine。

   ```console
   $ sudo systemctl enable --now docker
   ```

   上述命令会将 Docker 的 systemd 服务配置为开机自启动。
   如果你不希望 Docker 随系统启动，可改用 `sudo systemctl start docker`。

7. 通过运行 `hello-world` 镜像验证安装是否成功：

   ```console
   $ sudo docker run hello-world
   ```

   该命令会下载测试镜像并在容器中运行。容器运行后会打印确认信息，然后退出。

至此，你已经成功安装并启动了 Docker Engine。

{{% include "root-errors.md" %}}

#### 升级 Docker Engine

要升级 Docker Engine，请下载新版安装包并重复[安装流程](#install-from-a-package)，
使用 `dnf upgrade`（而非 `dnf install`），并指向新文件。

{{% include "install-script.md" %}}

## 卸载 Docker Engine

1. 卸载 Docker Engine、CLI、containerd 与 Docker Compose 软件包：

   ```console
   $ sudo dnf remove docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin docker-ce-rootless-extras
   ```

2. 主机上的镜像、容器、卷或自定义配置文件不会自动删除。若要删除所有镜像、容器与卷：

   ```console
   $ sudo rm -rf /var/lib/docker
   $ sudo rm -rf /var/lib/containerd
   ```

任何你修改过的配置文件需要手动删除。

## 进一步阅读

- 继续阅读 [Linux 安装后的步骤](linux-postinstall.md)。
