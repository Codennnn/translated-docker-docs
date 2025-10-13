---
title: 更新应用
weight: 30
linkTitle: "第 2 部分：更新应用"
keywords: 入门, 安装, 上手, 快速开始, 概念, 容器, Docker Desktop
description: 修改你的应用
aliases:
 - /get-started/03_updating_app/
 - /guides/workshop/03_updating_app/
---

在[第 1 部分](./02_our_app.md)中，你已经将一个待办事项应用容器化。本节你将更新应用及其镜像，并学习如何停止与删除容器。

## 更新源代码

接下来，你将把在没有任何待办事项时显示的“空列表提示”从“No items yet! Add one above!”修改为“You have no todo items yet! Add one above!”。


1. 在 `src/static/js/app.js` 文件中，将第 56 行更新为新的空列表提示文本。

   ```diff
   - <p className="text-center">No items yet! Add one above!</p>
   + <p className="text-center">You have no todo items yet! Add one above!</p>
   ```

2. 使用 `docker build` 命令构建更新后的镜像：

   ```console
   $ docker build -t getting-started .
   ```

3. 使用更新后的代码启动一个新容器：

   ```console
   $ docker run -dp 127.0.0.1:3000:3000 getting-started
   ```

你可能会看到如下错误：

```console
docker: Error response from daemon: driver failed programming external connectivity on endpoint laughing_burnell 
(bb242b2ca4d67eba76e79474fb36bb5125708ebdabd7f45c8eaf16caaabde9dd): Bind for 127.0.0.1:3000 failed: port is already allocated.
```

出现该错误的原因是旧容器仍在运行，导致无法启动新的容器。旧容器已经占用了宿主机的 3000 端口，而同一台机器上（包括容器在内）同一个端口只能被一个进程监听。要解决这个问题，你需要先移除旧容器。

## 移除旧容器

要移除容器，首先需要停止它。容器停止后，你就可以将其删除。你可以使用 CLI 或 Docker Desktop 图形界面来移除旧容器，选择你更熟悉的方式即可。

{{< tabs >}}
{{< tab name="CLI" >}}

### 使用 CLI 移除容器

1. 通过 `docker ps` 命令获取容器 ID。

   ```console
   $ docker ps
   ```

2. 使用 `docker stop` 命令停止容器。将 `<the-container-id>` 替换为 `docker ps` 输出中的容器 ID。

   ```console
   $ docker stop <the-container-id>
   ```

3. 容器停止后，使用 `docker rm` 命令将其删除。

   ```console
   $ docker rm <the-container-id>
   ```

> [!NOTE]
>
> 你也可以通过给 `docker rm` 命令添加 `-f`（`--force`）参数，在一条命令中同时停止并移除容器。例如：`docker rm -f <the-container-id>`

{{< /tab >}}
{{< tab name="Docker Desktop" >}}

### 使用 Docker Desktop 移除容器

1. 打开 Docker Desktop，进入 **Containers** 视图。
2. 在要删除的容器行的 **Actions** 列中，选择垃圾桶图标。
3. 在确认对话框中，选择 **Delete forever**。

{{< /tab >}}
{{< /tabs >}}

### 启动已更新的应用容器

1. 现在使用 `docker run` 命令启动更新后的应用：

   ```console
   $ docker run -dp 127.0.0.1:3000:3000 getting-started
   ```

2. 刷新浏览器访问 [http://localhost:3000](http://localhost:3000)，你应能看到已更新的提示文本。

## 小结

本节你学习了如何更新并重建镜像，以及如何停止并移除容器。

相关信息：
 - [docker CLI reference](/reference/cli/docker/)

## 下一步

接下来，你将学习如何与他人共享镜像。

{{< button text="分享应用" url="04_sharing_app.md" >}}
