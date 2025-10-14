---
title: 如何备份与恢复 Docker Desktop 数据
linkTitle: 备份与恢复数据
keywords: Docker Desktop, 备份, 恢复, 迁移, 重装, 容器, 镜像,
  卷
weight: 20
aliases:
 - /desktop/backup-and-restore/
---

使用以下步骤备份与恢复你的镜像和容器数据。以下场景尤其有用：重置 VM 磁盘、将 Docker 环境迁移到新电脑、或从失败的 Docker Desktop 更新/安装中恢复。

> [!IMPORTANT]
>
> 如果你使用卷或绑定挂载存储容器数据，可能无需备份容器本身。但请务必记录创建容器时使用的选项；或者使用 [Docker Compose 文件](/reference/compose-file/_index.md)，以便在重新安装后按相同配置重新创建容器。

## 如果 Docker Desktop 正常运行

### 备份你的数据

1. 使用 [`docker container commit`](/reference/cli/docker/container/commit.md) 将容器提交为镜像。

   提交操作会将文件系统变更与部分容器配置（如标签、环境变量）保存为本地镜像。注意：环境变量可能包含敏感信息（如密码、代理认证），推送至仓库时请谨慎处理。

   同时，挂载到容器的卷中的文件系统变更不会包含在镜像中，需要单独备份。

   如果你使用了[命名卷](/manuals/engine/storage/_index.md#more-details-about-mount-types)存储容器数据（如数据库），请参考存储章节的[备份、恢复或迁移数据卷](/manuals/engine/storage/volumes.md#back-up-restore-or-migrate-data-volumes)。

2. 使用 [`docker push`](/reference/cli/docker/image/push.md) 将需要保留的本地构建镜像推送到 [Docker Hub 仓库](/manuals/docker-hub/_index.md)。
   
   > [!TIP]
   >
   > 若镜像包含敏感内容，请[将仓库可见性设置为私有](/manuals/docker-hub/repos/_index.md)。

   或使用 [`docker image save -o images.tar image1 [image2 ...]`](/reference/cli/docker/image/save.md) 将镜像保存为本地 `.tar` 文件。 

备份完成后，你可以卸载当前版本的 Docker Desktop，并[安装其他版本](/manuals/desktop/release-notes.md)，或将 Docker Desktop 重置为出厂设置。

### 恢复你的数据

1. 加载你的镜像。

   - 如果你推送到了 Docker Hub：
   
      ```console
      $ docker pull <my-backup-image>
      ```
   
   - 如果你保存为 `.tar` 文件：
   
      ```console
      $ docker image load -i images.tar
      ```

2. 视需要重新创建容器，可使用 [`docker run`](/reference/cli/docker/container/run.md) 或 [Docker Compose](/manuals/compose/_index.md)。

要恢复卷中的数据，请参考[备份、恢复或迁移数据卷](/manuals/engine/storage/volumes.md#back-up-restore-or-migrate-data-volumes)。 

## 如果 Docker Desktop 无法启动 

若 Docker Desktop 无法启动且需要重新安装，你可以直接从磁盘备份其 VM 磁盘与镜像数据。备份这些文件前必须完全停止 Docker Desktop。

{{< tabs >}}
{{< tab name="Windows" >}}

1. 备份 Docker 容器/镜像。

   备份以下文件：

   ```console
   %LOCALAPPDATA%\Docker\wsl\data\docker_data.vhdx
   ```

   将其复制到安全位置。 

1. 备份 WSL 发行版。

   如果你运行了任意 WSL Linux 发行版（Ubuntu、Alpine 等），请按照 [Microsoft 的指南](https://learn.microsoft.com/en-us/windows/wsl/faq#how-can-i-back-up-my-wsl-distributions-) 进行备份。

1. 恢复。 

   重新安装 Docker Desktop 后，将 `docker_data.vhdx` 还原到相同位置；如需，重新导入你的 WSL 发行版。

{{< /tab >}}
{{< tab name="Mac" >}}

1. 备份 Docker 容器/镜像。

   备份以下文件：

   ```console
   ~/Library/Containers/com.docker.docker/Data/vms/0/data/Docker.raw
   ```

   将其复制到安全位置。 

1. 恢复。 

   重新安装 Docker Desktop 后，将 `Docker.raw` 还原到相同位置。

{{< /tab >}}
{{< tab name="Linux" >}}

1. 备份 Docker 容器/镜像：

   备份以下文件：

   ```console
   ~/.docker/desktop/vms/0/data/Docker.raw
   ```

   将其复制到安全位置。

1. 恢复。 

   重新安装 Docker Desktop 后，将 `Docker.raw` 还原到相同位置。

{{< /tab >}}
{{< /tabs >}}