---
description: 了解如何解决影响 macOS 上 Docker Desktop 用户的问题，包括启动问题和错误的恶意软件警告，提供升级、补丁和临时解决方案。
keywords: Docker desktop, 修复, mac, 故障排除, macos, 错误恶意软件警告, 补丁, 升级解决方案
title: 解决 macOS 上 Docker Desktop 的最新问题
linkTitle: 修复 Mac 启动问题
weight: 220
sitemap: false
---

本指南提供了解决影响部分 macOS 上 Docker Desktop 用户的最新问题的步骤。此问题可能导致 Docker Desktop 无法启动，在某些情况下，还可能触发不准确的恶意软件警告。有关此事件的更多详细信息，请参阅[博客文章](https://www.docker.com/blog/incident-update-docker-desktop-for-mac/)。

> [!NOTE]
>
> Docker Desktop 4.28 及更早版本不受此问题影响。

## 可用解决方案

根据您的情况，有以下几种选项可供选择：

### 升级到 Docker Desktop 4.37.2 版本（推荐）

推荐的方式是升级到最新的 Docker Desktop 4.37.2 版本。

如果可能，请直接通过应用程序更新。如果不行，并且您仍然看到恶意软件弹出窗口，请按照以下步骤操作：

1. 终止无法正常启动的 Docker 进程：
   ```console
   $ sudo launchctl bootout system/com.docker.vmnetd 2>/dev/null || true
   $ sudo launchctl bootout system/com.docker.socket 2>/dev/null || true
    
   $ sudo rm /Library/PrivilegedHelperTools/com.docker.vmnetd || true
   $ sudo rm /Library/PrivilegedHelperTools/com.docker.socket || true
 
   $ ps aux | grep -i docker | awk '{print $2}' | sudo xargs kill -9 2>/dev/null
   ```
    
2. 确保恶意软件弹出窗口已永久关闭。

3. [下载并安装 4.37.2 版本](/manuals/desktop/release-notes.md#4372)。

4. 启动 Docker Desktop。5 到 10 秒后会显示权限弹出消息。

5. 输入您的密码。

现在您应该能看到 Docker Desktop 仪表板。

> [!TIP]
>
> 如果完成这些步骤后恶意软件弹出窗口仍然存在，且 Docker 已在回收站中，请尝试清空回收站并重新运行这些步骤。

### 如果您使用的是 4.32 - 4.36 版本，请安装补丁

如果您无法升级到最新版本，并且看到恶意软件弹出窗口，请按照以下步骤操作：

1. 终止无法正常启动的 Docker 进程：
   ```console
   $ sudo launchctl bootout system/com.docker.vmnetd 2>/dev/null || true
   $ sudo launchctl bootout system/com.docker.socket 2>/dev/null || true
    
   $ sudo rm /Library/PrivilegedHelperTools/com.docker.vmnetd || true
   $ sudo rm /Library/PrivilegedHelperTools/com.docker.socket || true
 
   $ ps aux | grep docker | awk '{print $2}' | sudo xargs kill -9 2>/dev/null
   ```

2. 确保恶意软件弹出窗口已永久关闭。

3. [下载并安装与您当前基础版本匹配的补丁安装程序](/manuals/desktop/release-notes.md)。例如，如果您使用的是 4.36.0 版本，请安装 4.36.1。

4. 启动 Docker Desktop。5 到 10 秒后会显示权限弹出消息。

5. 输入您的密码。

现在您应该能看到 Docker Desktop 仪表板。

> [!TIP]
>
> 如果完成这些步骤后恶意软件弹出窗口仍然存在，且 Docker 已在回收站中，请尝试清空回收站并重新运行这些步骤。

## MDM 脚本

如果您是 IT 管理员，并且您的开发人员看到恶意软件弹出窗口：

1. 确保您的开发人员拥有重新签名的 Docker Desktop 4.32 或更高版本。
2. 运行以下脚本：

   ```console
   #!/bin/bash

   # 停止 docker 服务
   echo "正在停止 Docker..."
   sudo pkill -i docker

   # 停止 vmnetd 服务
   echo "正在停止 com.docker.vmnetd 服务..."
   sudo launchctl bootout system /Library/LaunchDaemons/com.docker.vmnetd.plist

   # 停止 socket 服务
   echo "正在停止 com.docker.socket 服务..."
   sudo launchctl bootout system /Library/LaunchDaemons/com.docker.socket.plist

   # 移除 vmnetd 二进制文件
   echo "正在移除 com.docker.vmnetd 二进制文件..."
   sudo rm -f /Library/PrivilegedHelperTools/com.docker.vmnetd

   # 移除 socket 二进制文件
   echo "正在移除 com.docker.socket 二进制文件..."
   sudo rm -f /Library/PrivilegedHelperTools/com.docker.socket

   # 安装新的二进制文件
   echo "正在安装新的二进制文件..."
   sudo cp /Applications/Docker.app/Contents/Library/LaunchServices/com.docker.vmnetd /Library/PrivilegedHelperTools/
   sudo cp /Applications/Docker.app/Contents/MacOS/com.docker.socket /Library/PrivilegedHelperTools/
   ```

## Homebrew casks

如果您使用 Homebrew casks 安装了 Docker Desktop，推荐的解决方案是执行完全重新安装以解决此问题。

要重新安装 Docker Desktop，请在终端中运行以下命令：

```console
$ brew update
$ brew reinstall --cask docker
```

这些命令将更新 Homebrew 并完全重新安装 Docker Desktop，确保您拥有已应用修复的最新版本。
