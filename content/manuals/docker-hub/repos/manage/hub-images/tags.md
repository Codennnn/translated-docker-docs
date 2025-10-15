---
description: 了解如何在 Docker Hub 上管理存储库标签。
keywords: Docker Hub, Hub, 存储库内容, 标签
title: Docker Hub 上的标签
linkTitle: 标签
weight: 10
---

在同一个 Docker Hub 存储库中，标签（tag）可用于管理镜像的多个版本。
为每个镜像添加特定的 `:<tag>`（例如 `docs/base:testing`），
即可按不同使用场景对镜像版本进行组织与区分。
如果未指定标签，默认使用 `latest`。

## 为本地镜像打标签

为本地镜像打标签，你可以使用以下任一方式：

- 在构建镜像时使用 `docker build -t <org-or-user-namespace>/<repo-name>[:<tag>]`。
- 使用 `docker tag <existing-image> <org-or-user-namespace>/<repo-name>[:<tag>]` 重新打标签。
- 提交变更时使用 `docker commit <existing-container> <org-or-user-namespace>/<repo-name>[:<tag>]`。

随后，你可以将该镜像推送到其名称或标签指定的存储库：

```console
$ docker push <org-or-user-namespace>/<repo-name>:<tag>
```

镜像上传成功后，即可在 Docker Hub 中使用。

## 查看存储库标签

你可以查看可用的标签以及其对应镜像的大小。

1. 登录 [Docker Hub](https://hub.docker.com)。
2. 选择 **My Hub** > **Repositories**。

   随即显示你的存储库列表。

3. 选择一个存储库。

   将进入该存储库的 **General** 页面。

4. 切换到 **Tags** 选项卡。

你可以点击某个标签的 digest 查看更多详情。

## 删除存储库标签

仅存储库所有者或被授予权限的团队成员可以删除标签。

1. 登录 [Docker Hub](https://hub.docker.com)。
2. 选择 **My Hub** > **Repositories**。

   随即显示你的存储库列表。

3. 选择一个存储库。

   将进入该存储库的 **General** 页面。

4. 切换到 **Tags** 选项卡。

5. 勾选要删除的标签旁的复选框。

6. 点击 **Delete**。

   随后会弹出确认对话框。

7. 再次点击 **Delete**。
