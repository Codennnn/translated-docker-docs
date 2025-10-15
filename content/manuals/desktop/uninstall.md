---
description: 如何卸载 Docker Desktop
keywords: Windows, uninstall, Mac, Linux, Docker Desktop
title: 卸载 Docker Desktop
linkTitle: 卸载
weight: 210
---

> [!WARNING]
>
> 卸载 Docker Desktop 会销毁本地机器上的 Docker 容器、镜像、卷以及其他 Docker 相关数据，并删除应用程序生成的文件。要了解如何在卸载前保存重要数据，请参阅[备份和恢复数据](/manuals/desktop/settings-and-maintenance/backup-and-restore.md)部分。

{{< tabs >}}
{{< tab name="Windows" >}}

#### 通过图形界面

1. 从 Windows **开始** 菜单中，选择 **设置** > **应用** > **应用和功能**。
2. 从 **应用和功能** 列表中选择 **Docker Desktop**，然后选择 **卸载**。
3. 选择 **卸载** 确认你的选择。

#### 通过命令行

1. 找到安装程序：
   ```console
   $ C:\Program Files\Docker\Docker\Docker Desktop Installer.exe
   ```
2. 卸载 Docker Desktop。
 - 在 PowerShell 中，运行：
    ```console
    $ Start-Process 'Docker Desktop Installer.exe' -Wait uninstall
    ```
 - 在命令提示符中，运行：
    ```console
    $ start /w "" "Docker Desktop Installer.exe" uninstall
    ```

卸载 Docker Desktop 后，可能会留下一些残留文件，你可以手动删除这些文件：

```console
C:\ProgramData\Docker
C:\ProgramData\DockerDesktop
C:\Program Files\Docker
C:\Users\<your user name>\AppData\Local\Docker
C:\Users\<your user name>\AppData\Roaming\Docker
C:\Users\<your user name>\AppData\Roaming\Docker Desktop
C:\Users\<your user name>\.docker
```
 
{{< /tab >}}
{{< tab name="Mac" >}}

#### 通过图形界面

1. 打开 Docker Desktop。
2. 在 Docker Desktop 仪表板的右上角，选择 **故障排查** 图标。
3. 选择 **卸载**。
4. 出现提示时，再次选择 **卸载** 进行确认。

然后你可以将 Docker 应用程序移至废纸篓。

#### 通过命令行

运行：

```console
$ /Applications/Docker.app/Contents/MacOS/uninstall
```

然后你可以将 Docker 应用程序移至废纸篓。

> [!NOTE]
> 使用卸载命令卸载 Docker Desktop 时，你可能会遇到以下错误。
>
> ```console
> $ /Applications/Docker.app/Contents/MacOS/uninstall
> Password:
> Uninstalling Docker Desktop...
> Error: unlinkat /Users/<USER_HOME>/Library/Containers/com.docker.docker/.com.apple.containermanagerd.metadata.plist: > operation not permitted
> ```
>
> 此"操作不被允许"错误是针对文件 `.com.apple.containermanagerd.metadata.plist` 或其父目录 `/Users/<USER_HOME>/Library/Containers/com.docker.docker/` 报告的。可以忽略此错误，因为你已成功卸载 Docker Desktop。
> 你可以稍后通过为正在使用的终端应用程序授予 **完全磁盘访问** 权限来删除目录 `/Users/<USER_HOME>/Library/Containers/com.docker.docker/`（**系统设置** > **隐私与安全性** > **完全磁盘访问**）。

卸载 Docker Desktop 后，可能会留下一些残留文件，你可以删除这些文件：

```console
$ rm -rf ~/Library/Group\ Containers/group.com.docker
$ rm -rf ~/.docker
```

对于 Docker Desktop 4.36 及更早版本，文件系统上可能还会留下以下文件。你可以使用管理员权限删除这些文件：

```console
/Library/PrivilegedHelperTools/com.docker.vmnetd
/Library/PrivilegedHelperTools/com.docker.socket
```

{{< /tab >}}
{{< tab name="Ubuntu" >}}

要卸载 Docker Desktop for Ubuntu：

1. 删除 Docker Desktop 应用程序。运行：

   ```console
   $ sudo apt remove docker-desktop
   ```

   这会删除 Docker Desktop 软件包本身，但不会删除其所有文件或设置。

2. 手动删除残留文件。

   ```console
   $ rm -r $HOME/.docker/desktop
   $ sudo rm /usr/local/bin/com.docker.cli
   $ sudo apt purge docker-desktop
   ```

   这会删除 `$HOME/.docker/desktop` 中的配置和数据文件、`/usr/local/bin/com.docker.cli` 中的符号链接，并清除剩余的 systemd 服务文件。

3. 清理 Docker 配置设置。在 `$HOME/.docker/config.json` 中，删除 `credsStore` 和 `currentContext` 属性。

   这些条目告诉 Docker 在哪里存储凭据以及哪个上下文处于活动状态。如果它们在卸载 Docker Desktop 后仍然存在，可能会与将来的 Docker 设置冲突。

{{< /tab >}}
{{< tab name="Debian" >}}

要卸载 Docker Desktop for Debian，运行：

1. 删除 Docker Desktop 应用程序：

   ```console
   $ sudo apt remove docker-desktop
   ```

   这会删除 Docker Desktop 软件包本身，但不会删除其所有文件或设置。

2. 手动删除残留文件。

   ```console
   $ rm -r $HOME/.docker/desktop
   $ sudo rm /usr/local/bin/com.docker.cli
   $ sudo apt purge docker-desktop
   ```

   这会删除 `$HOME/.docker/desktop` 中的配置和数据文件、`/usr/local/bin/com.docker.cli` 中的符号链接，并清除剩余的 systemd 服务文件。

3. 清理 Docker 配置设置。在 `$HOME/.docker/config.json` 中，删除 `credsStore` 和 `currentContext` 属性。

   这些条目告诉 Docker 在哪里存储凭据以及哪个上下文处于活动状态。如果它们在卸载 Docker Desktop 后仍然存在，可能会与将来的 Docker 设置冲突。

{{< /tab >}}
{{< tab name="Fedora" >}}

要卸载 Docker Desktop for Fedora：

1. 删除 Docker Desktop 应用程序。运行：

   ```console
   $ sudo dnf remove docker-desktop
   ```

   这会删除 Docker Desktop 软件包本身，但不会删除其所有文件或设置。

2. 手动删除残留文件。

   ```console
   $ rm -r $HOME/.docker/desktop
   $ sudo rm /usr/local/bin/com.docker.cli
   $ sudo dnf remove docker-desktop
   ```

   这会删除 `$HOME/.docker/desktop` 中的配置和数据文件、`/usr/local/bin/com.docker.cli` 中的符号链接，并清除剩余的 systemd 服务文件。

3. 清理 Docker 配置设置。在 `$HOME/.docker/config.json` 中，删除 `credsStore` 和 `currentContext` 属性。

   这些条目告诉 Docker 在哪里存储凭据以及哪个上下文处于活动状态。如果它们在卸载 Docker Desktop 后仍然存在，可能会与将来的 Docker 设置冲突。

{{< /tab >}}
{{< tab name="Arch" >}}

要卸载 Docker Desktop for Arch：

1. 删除 Docker Desktop 应用程序。运行：

   ```console
   $ sudo pacman -Rns docker-desktop
   ```

   这会删除 Docker Desktop 软件包及其配置文件，以及其他软件包不需要的依赖项。

2. 手动删除残留文件。

   ```console
   $ rm -r $HOME/.docker/desktop
   ```

   这会删除 `$HOME/.docker/desktop` 中的配置和数据文件。

3. 清理 Docker 配置设置。在 `$HOME/.docker/config.json` 中，删除 `credsStore` 和 `currentContext` 属性。

   这些条目告诉 Docker 在哪里存储凭据以及哪个上下文处于活动状态。如果它们在卸载 Docker Desktop 后仍然存在，可能会与将来的 Docker 设置冲突。

{{< /tab >}}
{{< /tabs >}}


