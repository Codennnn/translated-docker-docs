---
title: 多容器应用
weight: 70
linkTitle: "第 6 部分：多容器应用"
keywords: 入门, 安装, 导览, 快速开始, 介绍, 概念, 容器,
  Docker Desktop
description: 在你的应用中使用多个容器
aliases:
 - /get-started/07_multi_container/
 - /guides/workshop/07_multi_container/
---

到目前为止，你一直在使用单容器应用。接下来，我们要在应用栈中加入 MySQL。常见的问题是：“MySQL 应该在哪里运行？和应用在同一个容器里，还是独立运行？”
通常的原则是：每个容器只做一件事，并且把这件事做好。以下是将数据库独立为单独容器的几个原因：

- API/前端与数据库的扩缩容策略通常不同。
- 分离的容器可以独立进行版本管理与升级。
- 本地可以使用容器运行数据库，但生产环境往往会使用托管数据库服务；此时你不会希望把数据库引擎和应用打包在一起。
- 在一个容器中运行多个进程需要引入进程管理器（容器默认只启动一个进程），这会增加启动/停止的复杂度。

当然还有其他原因。总之，如下图所示，最好将应用拆分为多个容器运行。

![Todo 应用连接到 MySQL 容器](images/multi-container.webp?w=350h=250)


## 容器网络

请记住，容器默认彼此隔离，同一台机器上的其他进程或容器对它来说是“不可见”的。那么，如何让一个容器与另一个容器通信？答案是“网络”。只要把两个容器放到同一个网络中，它们就可以互相访问。

## 启动 MySQL

将容器加入某个网络有两种方式：
 - 在启动容器时指定网络；
 - 将已在运行的容器连接到某个网络。

下面的步骤中，我们先创建网络，然后在启动时将 MySQL 容器加入该网络。

1. 创建网络。

   ```console
   $ docker network create todo-app
   ```

2. 启动一个 MySQL 容器并将其加入该网络。同时，定义一些用于初始化数据库的环境变量。关于 MySQL 支持的环境变量，请参考 [MySQL Docker Hub 页面](https://hub.docker.com/_/mysql/)中的“Environment Variables”部分。

   {{< tabs >}}
   {{< tab name="Mac / Linux / Git Bash" >}}
   
   ```console
   $ docker run -d \
       --network todo-app --network-alias mysql \
       -v todo-mysql-data:/var/lib/mysql \
       -e MYSQL_ROOT_PASSWORD=secret \
       -e MYSQL_DATABASE=todos \
       mysql:8.0
   ```

   {{< /tab >}}
   {{< tab name="PowerShell" >}}

   ```powershell
   $ docker run -d `
       --network todo-app --network-alias mysql `
       -v todo-mysql-data:/var/lib/mysql `
       -e MYSQL_ROOT_PASSWORD=secret `
       -e MYSQL_DATABASE=todos `
       mysql:8.0
   ```
   
   {{< /tab >}}
   {{< tab name="Command Prompt" >}}

   ```console
   $ docker run -d ^
       --network todo-app --network-alias mysql ^
       -v todo-mysql-data:/var/lib/mysql ^
       -e MYSQL_ROOT_PASSWORD=secret ^
       -e MYSQL_DATABASE=todos ^
       mysql:8.0
   ```
   
   {{< /tab >}}
   {{< /tabs >}}
   
   在上面的命令中，你可以看到 `--network-alias` 选项。稍后我们会进一步介绍它的作用。

   > [!TIP]
   >
   > 你会注意到，上述命令中挂载了名为 `todo-mysql-data` 的卷到 `/var/lib/mysql`（MySQL 的数据目录）。虽然我们没有运行 `docker volume create`，但 Docker 看到你使用了“命名卷”，会自动为你创建。

3. 为确认数据库已正常运行，连接到数据库并验证连接是否成功。

   ```console
   $ docker exec -it <mysql-container-id> mysql -u root -p
   ```

   出现密码提示后输入 `secret`。进入 MySQL shell 后，列出数据库并确认能看到 `todos`：

   ```console
   mysql> SHOW DATABASES;
   ```

   你将看到类似如下的输出：

   ```plaintext
   +--------------------+
   | Database           |
   +--------------------+
   | information_schema |
   | mysql              |
   | performance_schema |
   | sys                |
   | todos              |
   +--------------------+
   5 rows in set (0.00 sec)
   ```

4. 退出 MySQL shell，返回到宿主机的终端。

   ```console
   mysql> exit
   ```

   至此，`todos` 数据库已创建并可供使用。

## 连接到 MySQL

既然 MySQL 已启动并在运行，就可以开始使用它了。但具体怎么用？如果你在同一网络中再运行一个容器，该如何找到 MySQL 容器？请记住，每个容器都有自己的 IP 地址。

为了解答上述问题并加深对容器网络的理解，我们将使用 [nicolaka/netshoot](https://github.com/nicolaka/netshoot) 容器，它内置了大量用于排查与调试网络问题的工具。

1. 使用 nicolaka/netshoot 镜像启动一个新容器，并确保将其连接到同一个网络。

   ```console
   $ docker run -it --network todo-app nicolaka/netshoot
   ```

2. 在该容器内，使用 `dig` 命令（常用的 DNS 工具）查询主机名 `mysql` 对应的 IP 地址。

   ```console
   $ dig mysql
   ```

   你将看到类似如下的输出：

   ```text
   ; <<>> DiG 9.18.8 <<>> mysql
   ;; global options: +cmd
   ;; Got answer:
   ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 32162
   ;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

   ;; QUESTION SECTION:
   ;mysql.				IN	A

   ;; ANSWER SECTION:
   mysql.			600	IN	A	172.23.0.2

   ;; Query time: 0 msec
   ;; SERVER: 127.0.0.11#53(127.0.0.11)
   ;; WHEN: Tue Oct 01 23:47:24 UTC 2019
   ;; MSG SIZE  rcvd: 44
   ```

   在 “ANSWER SECTION” 中，可以看到 `mysql` 的 `A` 记录解析为 `172.23.0.2`（你的 IP 很可能不同）。虽然 `mysql` 不是常规的主机名，但 Docker 能将其解析到具有该网络别名的容器 IP。还记得之前使用的 `--network-alias` 吗？

   这意味着你的应用只需要连接主机名 `mysql`，就能与数据库通信。

## 使用 MySQL 运行应用

该 Todo 应用支持通过一组环境变量配置 MySQL 连接，分别是：

- `MYSQL_HOST` - 正在运行的 MySQL 的主机名
- `MYSQL_USER` - 连接所用的用户名
- `MYSQL_PASSWORD` - 连接所用的密码
- `MYSQL_DB` - 连接成功后使用的数据库名

> [!NOTE]
>
> 在开发环境中使用环境变量配置连接信息是常见做法，但在生产环境中并不推荐。Docker 前安全负责人 Diogo Monica 有一篇很好的文章
> [说明了原因](https://blog.diogomonica.com/2017/03/27/why-you-shouldnt-use-env-variables-for-secret-data/)。
>
> 更安全的方式是使用容器编排框架提供的“机密（secret）”支持。在多数情况下，这些机密会以文件形式挂载到容器中。很多应用（包括 MySQL 镜像与本 Todo 应用）也支持带有 `_FILE` 后缀的环境变量，用于指向包含机密的文件。
>
> 例如，设置 `MYSQL_PASSWORD_FILE` 后，应用会读取该文件内容作为连接密码。需要注意，Docker 并不会对这些变量做特殊处理，你的应用必须自行读取变量并获取文件内容。

现在，可以启动开发环境的容器了。

1. 指定上述环境变量，并将容器连接到应用网络。执行命令前，请确保当前目录为 `getting-started-app`。

   {{< tabs >}}
   {{< tab name="Mac / Linux" >}}

   ```console
   $ docker run -dp 127.0.0.1:3000:3000 \
     -w /app -v "$(pwd):/app" \
     --network todo-app \
     -e MYSQL_HOST=mysql \
     -e MYSQL_USER=root \
     -e MYSQL_PASSWORD=secret \
     -e MYSQL_DB=todos \
     node:lts-alpine \
     sh -c "yarn install && yarn run dev"
   ```
   
   {{< /tab >}}
   {{< tab name="PowerShell" >}}
   在 Windows 下，请在 PowerShell 中运行以下命令。

   ```powershell
   $ docker run -dp 127.0.0.1:3000:3000 `
     -w /app -v "$(pwd):/app" `
     --network todo-app `
     -e MYSQL_HOST=mysql `
     -e MYSQL_USER=root `
     -e MYSQL_PASSWORD=secret `
     -e MYSQL_DB=todos `
     node:lts-alpine `
     sh -c "yarn install && yarn run dev"
   ```

   {{< /tab >}}
   {{< tab name="Command Prompt" >}}
   在 Windows 下，请在命令提示符（Command Prompt）中运行以下命令。

   ```console
   $ docker run -dp 127.0.0.1:3000:3000 ^
     -w /app -v "%cd%:/app" ^
     --network todo-app ^
     -e MYSQL_HOST=mysql ^
     -e MYSQL_USER=root ^
     -e MYSQL_PASSWORD=secret ^
     -e MYSQL_DB=todos ^
     node:lts-alpine ^
     sh -c "yarn install && yarn run dev"
   ```

   {{< /tab >}}
   {{< tab name="Git Bash" >}}

   ```console
   $ docker run -dp 127.0.0.1:3000:3000 \
     -w //app -v "/$(pwd):/app" \
     --network todo-app \
     -e MYSQL_HOST=mysql \
     -e MYSQL_USER=root \
     -e MYSQL_PASSWORD=secret \
     -e MYSQL_DB=todos \
     node:lts-alpine \
     sh -c "yarn install && yarn run dev"
   ```
   
   {{< /tab >}}
   {{< /tabs >}}

2. 查看该容器的日志（`docker logs -f <container-id>`），你应能看到类似如下的信息，表示应用已连接到 MySQL 数据库：

   ```console
   $ nodemon src/index.js
   [nodemon] 2.0.20
   [nodemon] to restart at any time, enter `rs`
   [nodemon] watching dir(s): *.*
   [nodemon] starting `node src/index.js`
   Connected to mysql db at host mysql
   Listening on port 3000
   ```

3. 在浏览器中打开应用，向待办列表添加几条数据。

4. 连接到 MySQL 数据库，验证这些数据确实写入了数据库。记得密码是 `secret`。

   ```console
   $ docker exec -it <mysql-container-id> mysql -p todos
   ```

   在 mysql shell 中执行：

   ```console
   mysql> select * from todo_items;
   +--------------------------------------+--------------------+-----------+
   | id                                   | name               | completed |
   +--------------------------------------+--------------------+-----------+
   | c906ff08-60e6-44e6-8f49-ed56a0853e85 | Do amazing things! |         0 |
   | 2912a79e-8486-4bc3-a4c5-460793a575ab | Be awesome!        |         0 |
   +--------------------------------------+--------------------+-----------+
   ```

   你的表内容会与此不同，因为包含你刚添加的记录。但可以确认它们已被存储。

## 小结

至此，你已经让应用把数据存储到一个独立容器中的外部数据库里。你还了解了容器网络的基础，以及通过 DNS 实现的服务发现。

相关信息：
 - [docker CLI reference](/reference/cli/docker/)
 - [Networking overview](/manuals/engine/network/_index.md)

## 下一步

你可能开始觉得启动这个应用要做的事情有些繁琐：创建网络、启动容器、设置一堆环境变量、暴露端口等。这既难以记忆，也不方便分享给他人。

下一节将介绍 Docker Compose。借助 Compose，你可以更简单地共享应用栈，其他人也能用一条命令快速启动它们。

{{< button text="使用 Docker Compose" url="08_using_compose.md" >}}
