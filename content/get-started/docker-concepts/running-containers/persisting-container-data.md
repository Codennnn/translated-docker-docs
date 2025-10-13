---
title: 持久化容器数据
weight: 3
keywords: concepts, build, images, container, docker desktop
description: 本文将介绍在 Docker 中进行数据持久化的重要性。
aliases:
 - /guides/walkthroughs/persist-data/
 - /guides/docker-concepts/running-containers/persisting-container-data/
---

{{< youtube-embed 10_2BjqB_Ls >}}

## 概念解析

容器启动时，会使用镜像中提供的文件与配置。每个容器都可以创建、修改、删除文件，且这些操作不会影响其他容器。但当容器被删除时，这些文件更改也会随之消失。

这种“短暂性”（ephemeral）对很多场景很有用，但当你需要持久化数据时就会遇到挑战。比如，重启数据库容器时，你通常不希望从一个空数据库开始。那么，该如何持久化文件呢？

### 容器卷（Volumes）

卷是一种存储机制，能够让数据的生命周期超越单个容器。可以把它理解为：从容器内部到外部存储的一个“快捷方式/符号链接”。

例如，先创建一个名为 `log-data` 的卷：

```console
$ docker volume create log-data
```

然后使用以下命令启动容器，该卷会以挂载（或附加）的方式出现在容器的 `/logs` 目录：

```console
$ docker run -d -p 80:80 -v log-data:/logs docker/welcome-to-docker
```

如果卷 `log-data` 不存在，Docker 会自动为你创建。

当容器运行时，写入 `/logs` 目录的所有文件都会保存到这个卷中（容器外部）。即使删除容器，再启动一个新容器并挂载同一个卷，文件仍然存在。

> **通过卷共享文件**
>
> 你可以把同一个卷挂载到多个容器上，以在容器间共享文件。这在日志聚合、数据管道或事件驱动应用等场景中很有帮助。

### 管理卷

卷的生命周期独立于容器本身。根据数据与应用类型不同，卷可能会变得很大。以下命令有助于管理卷：

- `docker volume ls` —— 列出所有卷
- `docker volume rm <volume-name-or-id>` —— 删除卷（仅当该卷未被任何容器使用时生效）
- `docker volume prune` —— 删除所有未使用（未附加）的卷

## 动手试试

本指南将练习如何使用卷来持久化 Postgres 容器生成的数据。数据库运行时会把文件存储在 `/var/lib/postgresql/data` 目录。将卷挂载到这里，就能在多次重启容器时保留数据。

### 使用卷

1. [下载并安装](/get-started/get-docker/) Docker Desktop。

2. 使用 [Postgres 镜像](https://hub.docker.com/_/postgres) 启动一个容器：

    ```console
    $ docker run --name=db -e POSTGRES_PASSWORD=secret -d -v postgres_data:/var/lib/postgresql/data postgres
    ```

    该命令会在后台启动数据库、设置密码，并将卷挂载到 PostgreSQL 用于持久化数据库文件的目录。

3. 使用以下命令连接数据库：

    ```console
    $ docker exec -ti db psql -U postgres
    ```

4. 在 PostgreSQL 命令行中创建表并插入两条记录：

    ```text
    CREATE TABLE tasks (
        id SERIAL PRIMARY KEY,
        description VARCHAR(100)
    );
    INSERT INTO tasks (description) VALUES ('Finish work'), ('Have fun');
    ```

5. 在 PostgreSQL 命令行中验证数据：

    ```text
    SELECT * FROM tasks;
    ```

    你应看到类似输出：

    ```text
     id | description
    ----+-------------
      1 | Finish work
      2 | Have fun
    (2 rows)
    ```

6. 通过以下命令退出 PostgreSQL：

    ```console
    \q
    ```

7. 停止并删除数据库容器。请注意，尽管容器被删除，数据仍保留在 `postgres_data` 卷中。

    ```console
    $ docker stop db
    $ docker rm db
    ```

8. 重新启动一个新容器，并挂载同一个卷：

    ```console
    $ docker run --name=new-db -d -v postgres_data:/var/lib/postgresql/data postgres 
    ```

    你会注意到，这里没有再设置 `POSTGRES_PASSWORD` 环境变量。因为该变量仅在初始化新数据库时使用。

9. 验证数据库仍包含之前的记录：

    ```console
    $ docker exec -ti new-db psql -U postgres -c "SELECT * FROM tasks"
    ```

### 查看卷内容

Docker Desktop 控制面板支持查看任意卷的内容，并可对卷进行导出、导入与克隆。

1. 打开 Docker Desktop 控制面板，进入 **Volumes** 视图。你应能看到 **postgres_data** 卷。
2. 点击 **postgres_data** 卷的名称。
3. 在 **Data** 标签页中可以浏览卷中文件，双击文件可查看与修改其内容。
4. 右键文件可保存或删除。

### 删除卷

删除卷之前，必须先确保它没有被任何容器附加。如仍有容器在使用，可用 `-f` 选项先停止再删除：

```console
$ docker rm -f new-db
```

删除卷的几种方式：

- 在 Docker Desktop 控制面板中，选择该卷并点击 **Delete Volume**。
- 使用 `docker volume rm`：

    ```console
    $ docker volume rm postgres_data
    ```
- 使用 `docker volume prune` 删除所有未使用的卷：

    ```console
    $ docker volume prune
    ```

## 延伸阅读

以下资源可帮助你进一步了解卷：

- [在 Docker 中管理数据](/engine/storage)
- [卷](/engine/storage/volumes)
- [卷挂载](/engine/containers/run/#volume-mounts)

## 下一步

既然你已经掌握了容器数据持久化，接下来学习如何与容器共享本地文件。

{{< button text="与容器共享本地文件" url="sharing-local-files" >}}


