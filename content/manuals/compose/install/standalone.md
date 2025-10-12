---
title: 安装 Docker Compose 独立版
linkTitle: 独立版
description: 在 Linux 与 Windows Server 上安装旧版 Docker Compose 独立工具的说明
keywords: install docker-compose, standalone docker compose, docker-compose windows server, install docker compose linux, legacy compose install
toc_max: 3
weight: 20
---

本页介绍如何在 Linux 或 Windows Server 上通过命令行安装 Docker Compose 独立版。

> [!WARNING]
>
> 独立版使用 `-compose` 语法，而非当前标准语法 `compose`。  
> 例如，使用独立版时需要输入 `docker-compose up`，而不是 `docker compose up`。
> 此方式仅用于向后兼容。

## 在 Linux 上 {#on-linux}

1. 下载并安装 Docker Compose 独立版：

   ```console
   $ curl -SL https://github.com/docker/compose/releases/download/{{% param "compose_version" %}}/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose
   ```

2. 为目标路径中的独立版二进制文件添加可执行权限：

   ```console
   $ chmod +x /usr/local/bin/docker-compose
   ```

3. 使用 `docker-compose` 测试并执行 Docker Compose 命令。

> [!TIP]
>
> 安装后若 `docker-compose` 命令无法执行，请检查环境变量 `PATH`。
> 也可以创建指向 `/usr/bin`（或 `PATH` 中任意目录）的符号链接，例如：
> ```console
> $ sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
> ```

## 在 Windows Server 上 {#on-windows-server}

如果你在 [Microsoft Windows Server 上直接运行 Docker 守护进程](/manuals/engine/install/binaries.md#install-server-and-client-binaries-on-windows)，并希望安装 Docker Compose，请按以下步骤进行。

1.  以管理员身份运行 PowerShell。
    当系统询问是否允许此应用对设备进行更改时，选择 **Yes** 以继续安装。

2.  可选：确保已启用 TLS1.2。
    GitHub 要求使用 TLS1.2 建立安全连接。若你使用较旧的 Windows Server（例如 2016），或怀疑未启用 TLS1.2，请在 PowerShell 中执行：

    ```powershell
    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
    ```

3. 下载 Docker Compose 最新版本（{{% param "compose_version" %}}）：

    ```powershell
     Start-BitsTransfer -Source "https://github.com/docker/compose/releases/download/{{% param "compose_version" %}}/docker-compose-windows-x86_64.exe" -Destination $Env:ProgramFiles\Docker\docker-compose.exe
    ```

    若需安装其他版本，请将 `{{% param "compose_version" %}}` 替换为目标版本号。

    > [!NOTE]
    >
    > 在 Windows Server 2019 上，你可以将 Compose 可执行文件添加到 `$Env:ProgramFiles\Docker`。
    > 由于该目录已注册到系统 `PATH`，在后续步骤中无需额外配置即可运行 `docker-compose --version` 命令。

4.  验证安装：

    ```console
    $ docker-compose.exe version
    Docker Compose version {{% param "compose_version" %}}
    ```

## 下一步

- [了解 Compose 的工作原理](/manuals/compose/intro/compose-application-model.md)
- [试试快速开始](/manuals/compose/gettingstarted.md)
