---
description: 了解暂停 Docker Desktop 仪表板意味着什么
keywords: Docker Desktop 仪表板, 管理, 容器, 图形界面, 仪表板, 暂停, 用户手册
title: 暂停 Docker Desktop
weight: 60
---

暂停 Docker Desktop 会暂时挂起运行 Docker 引擎的 Linux 虚拟机。此操作会将所有容器的当前状态保存在内存中并冻结所有正在运行的进程，从而显著降低 CPU 与内存占用，有助于笔记本电脑节省电量。

要暂停 Docker Desktop，请在 Docker 仪表板页脚左侧选择 **Pause** 图标。要手动恢复，请在 Docker 菜单中选择 **Resume**，或执行任意 Docker CLI 命令。

当你手动暂停 Docker Desktop 时，Docker 菜单和 Docker Desktop 仪表板上会显示“已暂停”状态。你仍然可以访问 **Settings** 与 **Troubleshoot** 菜单。

> [!TIP]
>
> Resource Saver 功能默认启用，且相较于手动 Pause 能提供更好的 CPU 与内存节省效果。参见[Resource Saver 模式](resource-saver.md)了解更多。
