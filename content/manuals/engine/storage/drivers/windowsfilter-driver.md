---
description: 了解 windowsfilter 存储驱动
keywords: 容器, 存储, 驱动, Windows, windowsfilter
title: windowsfilter 存储驱动
---

windowsfilter 存储驱动是 Windows 上 Docker Engine 的默认存储驱动。windowsfilter 使用 Windows 原生的文件系统层机制在磁盘上保存 Docker 的层与卷数据。该驱动仅适用于使用 NTFS 格式化的文件系统。

## 配置 windowsfilter 存储驱动

在大多数场景下，无需对 windowsfilter 存储驱动进行额外配置。

Windows 上的 Docker Engine 默认存储上限为 127GB。若需使用不同的存储大小，请为 windowsfilter 存储驱动设置 `size` 选项。参见[windowsfilter 选项](/reference/cli/dockerd.md#windowsfilter-options)。

默认情况下，数据存放在 Docker 宿主机 `C:\ProgramData\docker` 目录下的 `image` 与 `windowsfilter` 子目录中。你可以通过在[守护进程配置文件](/reference/cli/dockerd.md#on-windows)中设置 `data-root` 来更改存储位置：

```json
{
  "data-root": "d:\\docker"
}
```

你需要重启守护进程，配置变更方可生效。

## 更多信息

关于 Windows 上容器存储工作方式的更多信息，请参阅 Microsoft 的[Windows 容器文档](https://learn.microsoft.com/en-us/virtualization/windowscontainers/manage-containers/container-storage)。
