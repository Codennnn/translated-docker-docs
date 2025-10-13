---
title: 使用 Docker Compose
weight: 80
linkTitle: "第 7 部分：使用 Docker Compose"
keywords: 入门, 安装, 导览, 快速开始, 介绍, 概念, 容器,
  Docker Desktop
description: 使用 Docker Compose 管理多容器应用
aliases:
 - /get-started/08_using_compose/
 - /guides/workshop/08_using_compose/
---

[Docker Compose](/manuals/compose/_index.md) 是一个帮助你定义并共享多容器应用的工具。借助 Compose，你可以通过一个 YAML 文件定义各个服务，并用一条命令启动整个应用栈，或将其完全清理。

使用 Compose 的最大优势在于：你可以把应用栈定义在一个文件中，放在项目仓库的根目录（享受版本控制），并且让他人更容易参与贡献。别人只需克隆你的仓库，用 Compose 就能启动应用。事实上，如今在 GitHub/GitLab 上已有大量项目采用这种方式。

## 创建 Compose 文件

在 `getting-started-app` 目录下，新建一个名为 `compose.yaml` 的文件。

```text
├── getting-started-app/
│ ├── Dockerfile
│ ├── compose.yaml
│ ├── node_modules/
│ ├── package.json
│ ├── spec/
│ ├── src/
│ └── yarn.lock
```

## 定义应用服务

在[第 6 部分](./07_multi_container.md)中，你使用了下面这条命令来启动应用服务。

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

接下来，你将在 `compose.yaml` 文件中定义该服务。

1. 在编辑器中打开 `compose.yaml`，首先为你要运行的第一个服务（或容器）定义名称和镜像。该名称会自动成为一个网络别名，这在稍后定义 MySQL 服务时会用到。

   ```yaml
   services:
     app:
       image: node:lts-alpine
   ```

2. 通常会把 `command` 放在 `image` 定义附近（顺序并无强制要求）。在文件中加入 `command`：

   ```yaml
   services:
     app:
       image: node:lts-alpine
       command: sh -c "yarn install && yarn run dev"
   ```

3. 将命令中的 `-p 127.0.0.1:3000:3000` 迁移为服务的 `ports` 配置：

   ```yaml
   services:
     app:
       image: node:lts-alpine
       command: sh -c "yarn install && yarn run dev"
       ports:
         - 127.0.0.1:3000:3000
   ```

4. 接着，把工作目录（`-w /app`）和卷映射（`-v "$(pwd):/app"`）迁移为 `working_dir` 与 `volumes` 配置。

   使用 Docker Compose 定义卷的一个好处是：可以基于当前目录使用相对路径。

   ```yaml
   services:
     app:
       image: node:lts-alpine
       command: sh -c "yarn install && yarn run dev"
       ports:
         - 127.0.0.1:3000:3000
       working_dir: /app
       volumes:
         - ./:/app
   ```

5. 最后，使用 `environment` 定义环境变量。

   ```yaml
   services:
     app:
       image: node:lts-alpine
       command: sh -c "yarn install && yarn run dev"
       ports:
         - 127.0.0.1:3000:3000
       working_dir: /app
       volumes:
         - ./:/app
       environment:
         MYSQL_HOST: mysql
         MYSQL_USER: root
         MYSQL_PASSWORD: secret
         MYSQL_DB: todos
   ```

### 定义 MySQL 服务

现在来定义 MySQL 服务。你之前为该容器运行过如下命令：

```console
$ docker run -d \
  --network todo-app --network-alias mysql \
  -v todo-mysql-data:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=secret \
  -e MYSQL_DATABASE=todos \
  mysql:8.0
```

1. 先定义一个名为 `mysql` 的新服务，使其自动获得同名的网络别名，并指定要使用的镜像。

   ```yaml

   services:
     app:
       # The app service definition
     mysql:
       image: mysql:8.0
   ```

2. 然后定义卷映射。使用 `docker run` 时，Docker 会自动创建命名卷；而使用 Compose 时，你需要在顶层 `volumes:` 段定义该卷，并在服务配置中声明挂载点。仅提供卷名即可使用默认选项。

   ```yaml
   services:
     app:
       # The app service definition
     mysql:
       image: mysql:8.0
       volumes:
         - todo-mysql-data:/var/lib/mysql

   volumes:
     todo-mysql-data:
   ```

3. 最后，补充环境变量配置。

   ```yaml
   services:
     app:
       # The app service definition
     mysql:
       image: mysql:8.0
       volumes:
         - todo-mysql-data:/var/lib/mysql
       environment:
         MYSQL_ROOT_PASSWORD: secret
         MYSQL_DATABASE: todos

   volumes:
     todo-mysql-data:
   ```

此时，完整的 `compose.yaml` 应如下所示：


```yaml
services:
  app:
    image: node:lts-alpine
    command: sh -c "yarn install && yarn run dev"
    ports:
      - 127.0.0.1:3000:3000
    working_dir: /app
    volumes:
      - ./:/app
    environment:
      MYSQL_HOST: mysql
      MYSQL_USER: root
      MYSQL_PASSWORD: secret
      MYSQL_DB: todos

  mysql:
    image: mysql:8.0
    volumes:
      - todo-mysql-data:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: todos

volumes:
  todo-mysql-data:
```

## 运行应用栈

现在有了 `compose.yaml` 文件，就可以启动应用了。

1. 先确保没有已有容器实例在运行。使用 `docker ps` 列出容器，并用 `docker rm -f <ids>` 移除它们。

2. 使用 `docker compose up` 启动应用栈。加上 `-d` 参数可在后台运行。

   ```console
   $ docker compose up -d
   ```

    运行上述命令后，你会看到类似如下输出：

   ```plaintext
   Creating network "app_default" with the default driver
   Creating volume "app_todo-mysql-data" with default driver
   Creating app_app_1   ... done
   Creating app_mysql_1 ... done
   ```

    你会注意到 Docker Compose 同时创建了卷与网络。默认情况下，Compose 会为应用栈自动创建一个专用网络（这也是为什么我们没有在 Compose 文件中显式定义网络）。

3. 使用 `docker compose logs -f` 查看日志。你会看到所有服务的日志交错输出到同一条流里，这在排查时序相关问题时非常有用。`-f` 参数会持续跟随日志，实时显示新输出。

    如果你已经运行了该命令，会看到类似如下的输出：

    ```plaintext
    mysql_1  | 2019-10-03T03:07:16.083639Z 0 [Note] mysqld: ready for connections.
    mysql_1  | Version: '8.0.31'  socket: '/var/run/mysqld/mysqld.sock'  port: 3306  MySQL Community Server (GPL)
    app_1    | Connected to mysql db at host mysql
    app_1    | Listening on port 3000
    ```

    每行日志开头会显示服务名（通常带颜色），便于区分来源。若只想看某个服务的日志，可在命令末尾追加服务名（例如 `docker compose logs -f app`）。

4. 现在，打开浏览器访问 [http://localhost:3000](http://localhost:3000)，即可看到应用运行起来了。

## 在 Docker Desktop Dashboard 中查看应用栈

在 Docker Desktop Dashboard 中，你会看到一个名为 **getting-started-app** 的分组。这是 Compose 的项目名，用于把相关容器归为一组。默认情况下，项目名就是 `compose.yaml` 所在目录的名称。

展开该堆栈后，你会看到在 Compose 文件中定义的两个容器。容器名称更具可读性，遵循 `<service-name>-<replica-number>` 的格式，因此很容易区分哪个是应用容器，哪个是 MySQL 数据库容器。

## 清理

当你需要清理时，直接运行 `docker compose down`，或者在 Docker Desktop Dashboard 中点击该应用堆栈的垃圾桶图标。容器会被停止，网络也会被移除。

> [!WARNING]
>
> 默认情况下，运行 `docker compose down` 不会删除 Compose 文件中定义的命名卷。若要同时删除卷，请添加 `--volumes` 参数。
>
> 在 Docker Desktop Dashboard 中删除应用堆栈时，也不会移除卷。

## 小结

本节介绍了 Docker Compose，以及它如何帮助你更简单地定义与共享多服务应用。

相关信息：
 - [Compose overview](/manuals/compose/_index.md)
 - [Compose file reference](/reference/compose-file/_index.md)
 - [Compose CLI reference](/reference/cli/docker/compose/_index.md)

## 下一步

接下来，你将学习一些用于改进 Dockerfile 的最佳实践。

{{< button text="镜像构建最佳实践" url="09_image_best.md" >}}
