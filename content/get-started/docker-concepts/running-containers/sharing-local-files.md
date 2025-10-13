---
title: 与容器共享本地文件
weight: 4
keywords: concepts, images, container, docker desktop
description: 本文将介绍 Docker 中可用的多种存储选项及其常见用法。
aliases: 
 - /guides/docker-concepts/running-containers/sharing-local-files/
---

{{< youtube-embed 2dAzsVg3Dek >}}


## 概念解析

每个容器都包含其运行所需的一切，而不依赖宿主机预先安装的依赖。容器在隔离环境中运行，尽量减少对宿主机与其他容器的影响。这种隔离带来一个重要好处：最大程度降低与宿主系统或其他容器之间的冲突。然而，也正因为隔离，容器默认无法直接访问宿主机上的数据。

设想一个场景：你的 Web 应用容器需要读取宿主机上的某个配置文件。该文件可能包含敏感信息，如数据库凭据或 API Key。如果把这些敏感信息直接打包进镜像，在镜像分发时就会存在安全风险。为解决隔离与数据访问之间的矛盾，Docker 提供了在宿主机与容器之间共享文件的存储选项。

Docker 提供两种主要方式来进行数据持久化与主机-容器文件共享：卷（volumes）与绑定挂载（bind mounts）。

### 卷 vs 绑定挂载

- 如果你希望确保容器内部生成或修改的数据在容器停止后仍能保留，应该使用“卷”。参见[持久化容器数据](/get-started/docker-concepts/running-containers/persisting-container-data/)了解更多场景与用法。
- 如果你希望把宿主机上的某些特定文件或目录直接共享给容器（如配置文件或开发代码），应使用“绑定挂载”。它就像在主机与容器之间开了一扇“直通门”。绑定挂载非常适合需要实时文件访问与共享的开发场景。

### 在主机与容器之间共享文件

在 `docker run` 命令中，`-v`（或 `--volume`）与 `--mount` 都可用于在宿主机与容器间共享文件或目录，但两者在行为与用法上存在差异。

- `-v` 语法更简单，适合基础的卷或绑定挂载操作。如果使用 `-v`/`--volume` 指定的宿主路径不存在，会被自动创建为目录。

设想你在开发机上有一个源码目录，构建生成的产物（可执行文件、图片等）放在该目录的子目录中，下文示例把这个子目录称为 `/HOST/PATH`。你希望这些产物在容器中可用，并且每次重新构建时容器能读取到最新的产物。

使用绑定挂载启动容器并把它映射到容器内的指定路径：

```console
$ docker run -v /HOST/PATH:/CONTAINER/PATH -it nginx
```

- `--mount` 提供更高级、更细粒度的控制，更适合复杂挂载或生产部署。如果用 `--mount` 绑定一个宿主机上尚不存在的路径，`docker run` 不会自动创建该路径，而是报错。

```console
$ docker run --mount type=bind,source=/HOST/PATH,target=/CONTAINER/PATH,readonly nginx
```

> [!NOTE]
>
> 官方建议优先使用 `--mount` 语法，它对挂载过程的控制更精细，也可以避免因目录缺失导致的隐性问题。

### 宿主文件的访问权限

使用绑定挂载时，务必确保 Docker 对宿主目录具有相应权限。你可以在创建容器时为挂载路径添加 `:ro`（只读）或 `:rw`（读写）后缀来控制权限。

例如，以下命令为容器授予读写权限：

```console
$ docker run -v HOST-DIRECTORY:/CONTAINER-DIRECTORY:rw nginx
```

只读挂载允许容器读取宿主文件，但不能修改或删除；读写挂载允许容器修改/删除文件，并会同步反映到宿主机上。只读挂载有助于避免容器意外改动宿主文件。

> **同步文件共享（Synchronized File Share）**
>
> 随着代码库规模增长，传统绑定挂载在频繁文件访问的开发环境中可能变慢。[同步文件共享](/manuals/desktop/features/synchronized-file-sharing.md) 通过同步文件系统缓存加速绑定挂载，在宿主机与虚拟机之间提供更快的访问体验。

## 动手试试

本实践将演示如何创建并使用绑定挂载在主机与容器之间共享文件。

### 运行容器

1. [下载并安装](/get-started/get-docker/) Docker Desktop。

2. 使用 [httpd](https://hub.docker.com/_/httpd) 镜像启动容器：

   ```console
   $ docker run -d -p 8080:80 --name my_site httpd:2.4
   ```

   该命令会在后台启动 `httpd` 服务，并将网页发布到宿主机的 `8080` 端口。

3. 打开浏览器访问 [http://localhost:8080](http://localhost:8080)，或使用 curl 验证服务是否正常：

    ```console
    $ curl localhost:8080
    ```

### 使用绑定挂载

通过绑定挂载，你可以把宿主机上的配置或站点文件映射到容器内指定位置。下面通过替换网页的方式演示其效果：

1. 在 Docker Desktop 控制面板删除现有容器：

   ![Docker Desktop Dashboard 截图，展示如何删除 httpd 容器](images/delete-httpd-container.webp?border=true)

2. 在宿主机上创建目录 `public_html`：

    ```console
    $ mkdir public_html
    ```

3. 进入 `public_html` 目录，并创建 `index.html` 文件，内容如下（一个包含友好鲸鱼问候的小网页）：

    ```html
    <!DOCTYPE html>
    <html lang="en">
    <head>
    <meta charset="UTF-8">
    <title> My Website with a Whale & Docker!</title>
    </head>
    <body>
    <h1>Whalecome!!</h1>
    <p>Look! There's a friendly whale greeting you!</p>
    <pre id="docker-art">
       ##         .
      ## ## ##        ==
     ## ## ## ## ##    ===
     /""""""""""""""\___/ ===
   {                       /  ===-
   \______ O           __/
    \    \         __/
     \____\_______/

    Hello from Docker!
    </pre>
    </body>
    </html>
    ```

4. 重新运行容器。`--mount` 与 `-v` 两种方式效果等同，不要同时执行（若已运行其一，需要先删除 `my_site` 再执行另一种）。

   {{< tabs >}}
   {{< tab name="`-v`" >}}

   ```console
   $ docker run -d --name my_site -p 8080:80 -v .:/usr/local/apache2/htdocs/ httpd:2.4
   ```

   {{< /tab >}}
   {{< tab name="`--mount`" >}}

   ```console
   $ docker run -d --name my_site -p 8080:80 --mount type=bind,source=./,target=/usr/local/apache2/htdocs/ httpd:2.4
   ```

   {{< /tab >}}
   {{< /tabs >}}

   > [!TIP]
   > 在 Windows PowerShell 中使用 `-v` 或 `--mount` 时，请提供目录的绝对路径，而不是 `./`。这是因为 PowerShell 处理相对路径的方式与 macOS/Linux 上常见的 bash 不同。

   一切就绪后，访问 [http://localhost:8080](http://localhost:8080)，你会看到带有友好鲸鱼问候的新网页。

### 在 Docker Desktop 中查看文件

1. 在容器的 **Files** 标签页中，浏览 `/usr/local/apache2/htdocs/` 下的已挂载文件，点击 **Open file editor** 可在线查看与编辑。

   ![Docker Desktop Dashboard 截图，展示容器内的已挂载文件](images/mounted-files.webp?border=true)

2. 在宿主机上删除该文件，回到 **Files** 视图可见该文件也已消失。

   ![Docker Desktop Dashboard 截图，展示容器内已被删除的文件](images/deleted-files.webp?border=true)

3. 在宿主机上重新创建该 HTML 文件，容器 **Files** 视图会重新出现该文件，同时网页也可再次访问。

### 停止容器

容器会持续运行直到你将其停止。

1. 打开 Docker Desktop 控制面板的 **Containers** 视图。
2. 找到需要停止的容器。
3. 在操作列中选择 **Delete**。

![Docker Desktop Dashboard 截图，展示如何删除容器](images/delete-the-container.webp?border=true)

## 延伸阅读

以下资源将帮助你进一步了解绑定挂载：

* [在 Docker 中管理数据](/storage/)
* [卷](/storage/volumes/)
* [绑定挂载](/storage/bind-mounts/)
* [运行容器](/reference/run/)
* [存储问题排查](/storage/troubleshooting_volume_errors/)
* [持久化容器数据](/get-started/docker-concepts/running-containers/persisting-container-data/)

## 下一步

既然你已经学会与容器共享本地文件，接下来可以学习多容器应用。

{{< button text="多容器应用" url="Multi-container applications" >}}
