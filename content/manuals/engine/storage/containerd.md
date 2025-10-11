---
title: 在 Docker Engine 中使用 containerd 镜像存储
linkTitle: containerd 镜像存储
weight: 50
keywords: containerd, snapshotters, image store, docker engine
description: 了解如何在 Docker Engine 上启用 containerd 镜像存储
aliases:
  - /storage/containerd/
---

{{< summary-bar feature_name="containerd" >}}

industry 标准的容器运行时 containerd 使用 snapshotter（快照器）来存储镜像与容器数据，替代经典的存储驱动。
虽然 `overlay2` 仍是 Docker Engine 的默认存储驱动，但你可以选择启用 containerd 的 snapshotter 功能（当前为实验特性）。

要了解 containerd 镜像存储及其优势，请参阅
[Docker Desktop 上的 containerd 镜像存储](/manuals/desktop/features/containerd.md)。

## 在 Docker Engine 上启用 containerd 镜像存储

切换到 containerd snapshotter 后，你将暂时看不到此前使用经典存储驱动创建的镜像与容器。
这些资源仍保留在文件系统中；关闭 snapshotter 功能后即可再次访问它们。

以下步骤演示如何启用 containerd snapshotter 功能：

1. 在 `/etc/docker/daemon.json` 配置文件中添加如下内容：

   ```json
   {
     "features": {
       "containerd-snapshotter": true
     }
   }
   ```

2. 保存文件。
3. 重启 Docker 守护进程使配置生效：

   ```console
   $ sudo systemctl restart docker
   ```

守护进程重启后，运行 `docker info` 会显示已在使用 containerd snapshotter 存储驱动：

```console
$ docker info -f '{{ .DriverStatus }}'
[[driver-type io.containerd.snapshotter.v1]]
```

Docker Engine 默认使用 `overlayfs` 类型的 containerd snapshotter。
