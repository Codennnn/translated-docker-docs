---
description: 了解如何诊断和排查 Docker Desktop 问题，以及如何查看日志。
keywords: Linux, Mac, Windows, troubleshooting, logs, issues, Docker Desktop
toc_max: 2
title: Docker Desktop 故障排查
linkTitle: 故障排查与诊断
aliases:
 - /desktop/linux/troubleshoot/
 - /desktop/mac/troubleshoot/
 - /desktop/windows/troubleshoot/
 - /docker-for-mac/troubleshoot/
 - /mackit/troubleshoot/
 - /windows/troubleshoot/
 - /docker-for-win/troubleshoot/
 - /docker-for-windows/troubleshoot/
 - /desktop/troubleshoot/overview/
 - /desktop/troubleshoot/
tags: [ Troubleshooting ]
weight: 10
---

本页面介绍如何诊断和排查 Docker Desktop 问题，以及如何查看日志。

## 故障排查菜单

访问**故障排查**功能有以下两种方式：

- 点击 Docker 菜单 {{< inline-image src="../../images/whale-x.svg" alt="whale menu" >}}，然后选择**故障排查**。
- 点击 Docker Dashboard 右上角附近的**故障排查**图标。

**故障排查**菜单包含以下选项：

- **重启 Docker Desktop**。

- **重置 Kubernetes 集群**。删除所有堆栈和 Kubernetes 资源。更多信息请参阅 [Kubernetes](/manuals/desktop/settings-and-maintenance/settings.md#kubernetes)。

- **清理/清除数据**。此选项会重置所有 Docker 数据，但不会恢复出厂设置。选择此选项会导致现有设置丢失。

- **恢复出厂设置**。选择此选项可将 Docker Desktop 的所有设置重置为初始状态，就像刚安装时一样。

如果您使用的是 Mac 或 Linux 系统，还可以选择从系统中**卸载** Docker Desktop。

## 诊断

> [!TIP]
>
> 如果在故障排查中没有找到解决方案，可以浏览 GitHub 代码仓库或创建新的 issue：
>
> - [docker/for-mac](https://github.com/docker/for-mac/issues)
> - [docker/for-win](https://github.com/docker/for-win/issues)
> - [docker/desktop-linux](https://github.com/docker/desktop-linux/issues)

### 从应用内诊断

1. 在**故障排查**中，选择**获取支持**。这将打开应用内支持页面并开始收集诊断信息。
2. 当诊断信息收集完成后，选择**上传以获取诊断 ID**。
3. 诊断信息上传后，Docker Desktop 会生成一个诊断 ID。复制此 ID。
4. 使用诊断 ID 获取帮助：
    - 如果您有付费的 Docker 订阅，选择**联系支持**。这将打开 Docker Desktop 支持表单。填写所需信息，并将第三步复制的 ID 添加到**诊断 ID 字段**中，然后选择**提交工单**以请求 Docker Desktop 支持。
        > [!NOTE]
        >
        > 您必须登录 Docker Desktop 才能访问支持表单。有关 Docker Desktop 支持涵盖哪些内容的信息，请参阅[支持](/manuals/desktop/troubleshoot-and-support/support.md)。
    - 如果您没有付费的 Docker 订阅，选择**报告错误**以在 GitHub 上创建新的 Docker Desktop issue。填写所需信息，并确保添加第三步中复制的诊断 ID。

### 从错误消息诊断

1. 当出现错误消息时，选择**收集诊断信息**。
2. 诊断信息上传后，Docker Desktop 会生成一个诊断 ID。复制此 ID。
3. 使用诊断 ID 获取帮助：
    - 如果您有付费的 Docker 订阅，选择**联系支持**。这将打开 Docker Desktop 支持表单。填写所需信息，并将第三步复制的 ID 添加到**诊断 ID 字段**中，然后选择**提交工单**以请求 Docker Desktop 支持。
        > [!NOTE]
        >
        > 您必须登录 Docker Desktop 才能访问支持表单。有关 Docker Desktop 支持涵盖哪些内容的信息,请参阅[支持](/manuals/desktop/troubleshoot-and-support/support.md)。
    - 如果您没有付费的 Docker 订阅，可以在 GitHub 上为 [Mac](https://github.com/docker/for-mac/issues)、[Windows](https://github.com/docker/for-win/issues) 或 [Linux](https://github.com/docker/for-linux/issues) 创建新的 Docker Desktop issue。填写所需信息，并确保添加第二步中生成的诊断 ID。 

### 从终端诊断

在某些情况下，自行运行诊断工具会很有用，例如当 Docker Desktop 无法启动时。

{{< tabs group="os" >}}
{{< tab name="Windows" >}}

1. 找到 `com.docker.diagnose` 工具：

   ```console
   $ C:\Program Files\Docker\Docker\resources\com.docker.diagnose.exe
   ```

2. 创建并上传诊断 ID。在 PowerShell 中运行：

   ```console
   $ & "C:\Program Files\Docker\Docker\resources\com.docker.diagnose.exe" gather -upload
   ```

诊断完成后，终端会显示您的诊断 ID 和诊断文件的路径。诊断 ID 由您的用户 ID 和时间戳组成。例如 `BE9AFAAF-F68B-41D0-9D12-84760E6B8740/20190905152051`。

{{< /tab >}}
{{< tab name="Mac" >}}

1. 找到 `com.docker.diagnose` 工具：

   ```console
   $ /Applications/Docker.app/Contents/MacOS/com.docker.diagnose
   ```

2. 创建并上传诊断 ID。运行：

   ```console
   $ /Applications/Docker.app/Contents/MacOS/com.docker.diagnose gather -upload
   ```

诊断完成后，终端会显示您的诊断 ID 和诊断文件的路径。诊断 ID 由您的用户 ID 和时间戳组成。例如 `BE9AFAAF-F68B-41D0-9D12-84760E6B8740/20190905152051`。

{{< /tab >}}
{{< tab name="Linux" >}}

1. 找到 `com.docker.diagnose` 工具：

   ```console
   $ /opt/docker-desktop/bin/com.docker.diagnose
   ```

2. 创建并上传诊断 ID。运行：

   ```console
   $ /opt/docker-desktop/bin/com.docker.diagnose gather -upload
   ```

诊断完成后，终端会显示您的诊断 ID 和诊断文件的路径。诊断 ID 由您的用户 ID 和时间戳组成。例如 `BE9AFAAF-F68B-41D0-9D12-84760E6B8740/20190905152051`。

{{< /tab >}}
{{< /tabs >}}

要查看诊断文件的内容：

{{< tabs group="os" >}}
{{< tab name="Windows" >}}

1. 解压文件。在 PowerShell 中，将诊断文件的路径复制并粘贴到以下命令中，然后运行。命令类似于以下示例：

   ```powershell
   $ Expand-Archive -LiteralPath "C:\Users\testUser\AppData\Local\Temp\5DE9978A-3848-429E-8776-950FC869186F\20230607101602.zip" -DestinationPath "C:\Users\testuser\AppData\Local\Temp\5DE9978A-3848-429E-8776-950FC869186F\20230607101602"
   ```  

2. 在您偏好的文本编辑器中打开文件。运行：

   ```powershell
   $ code <path-to-file>
   ```

{{< /tab >}}
{{< tab name="Mac" >}}

运行：

```console
$ open /tmp/<your-diagnostics-ID>.zip
```

{{< /tab >}}
{{< tab name="Linux" >}}

运行：

```console
$ unzip –l /tmp/<your-diagnostics-ID>.zip
```

{{< /tab >}}
{{< /tabs >}}

#### 使用诊断 ID 获取帮助

如果您有付费的 Docker 订阅，选择**联系支持**。这将打开 Docker Desktop 支持表单。填写所需信息，并将第三步中复制的 ID 添加到**诊断 ID 字段**中，然后选择**提交工单**以请求 Docker Desktop 支持。
    
如果您没有付费的 Docker 订阅，可以在 GitHub 上创建 issue：

- [Linux](https://github.com/docker/desktop-linux/issues)
- [Mac](https://github.com/docker/for-mac/issues)
- [Windows](https://github.com/docker/for-win/issues)

### 自诊断工具

> [!IMPORTANT]
>
> 此工具已弃用。

## 查看日志

除了使用诊断选项提交日志外，您还可以自行浏览日志。

{{< tabs group="os" >}}
{{< tab name="Windows" >}}

在 PowerShell 中运行：

```powershell
$ code $Env:LOCALAPPDATA\Docker\log
```

这将在您偏好的文本编辑器中打开所有日志供您查看。

{{< /tab >}}
{{< tab name="Mac" >}}

### 从终端查看

要在命令行中实时查看 Docker Desktop 日志流，请在您偏好的 shell 中运行以下脚本：

```console
$ pred='process matches ".*(ocker|vpnkit).*" || (process in {"taskgated-helper", "launchservicesd", "kernel"} && eventMessage contains[c] "docker")'
$ /usr/bin/log stream --style syslog --level=debug --color=always --predicate "$pred"
```

或者，要将最近一天（`1d`）的日志收集到文件中，请运行：

```console
$ /usr/bin/log show --debug --info --style syslog --last 1d --predicate "$pred" >/tmp/logs.txt
```

### 从控制台应用查看

Mac 提供了一个名为 **Console** 的内置日志查看器，您可以使用它来查看 Docker 日志。

控制台位于 `/Applications/Utilities` 目录。您可以使用 Spotlight 搜索来找到它。

要读取 Docker 应用的日志消息，请在控制台窗口的搜索栏中输入 `docker` 并按回车键。然后选择 `ANY` 以展开 `docker` 搜索条目旁边的下拉列表，并选择 `Process`。

![Mac Console search for Docker app](../../images/console.png)

您可以使用控制台日志查询来搜索日志、以各种方式筛选结果并创建报告。

{{< /tab >}}
{{< tab name="Linux" >}}

您可以通过运行以下命令访问 Docker Desktop 日志：

```console
$ journalctl --user --unit=docker-desktop
```

您还可以在 `$HOME/.docker/desktop/log/` 目录下找到 Docker Desktop 包含的内部组件的日志。

{{< /tab >}}
{{< /tabs >}}

## 查看 Docker 守护进程日志

请参阅[读取守护进程日志](/manuals/engine/daemon/logs.md)部分，了解如何查看 Docker 守护进程日志。

## 更多资源

- 查看特定的[故障排查主题](topics.md)。
- 查看[已知问题](known-issues.md)相关信息
- [修复 macOS 上的 "Docker.app is damaged" 问题](mac-damaged-dialog.md) - 解决 macOS 安装问题
