---
description: 在 Swarm 中部署应用栈（Stack）
keywords: guide, swarm mode, composefile, stack, compose, deploy
title: 在 Swarm 中部署应用栈
---

当 Docker Engine 以 Swarm 模式运行时，你可以使用 `docker stack deploy` 将完整的应用栈部署到集群。`deploy` 命令接受以 [Compose 文件](/reference/compose-file/legacy-versions.md) 描述的栈定义。

{{% include "swarm-compose-compat.md" %}}

开始本教程前，你需要：

1.  一台已开启 [Swarm 模式](swarm-mode.md) 的 Docker Engine。
    如果你不熟悉 Swarm 模式，建议先阅读《[Swarm 模式关键概念](key-concepts.md)》与《[服务的工作原理](how-swarm-mode-works/services.md)》。

    > [!NOTE]
    >
    > 如果你在本地开发环境试用，可通过 `docker swarm init` 将引擎切换为 Swarm 模式。
    >
    > 若你已有多节点集群在运行，请注意所有 `docker stack` 与 `docker service` 命令都必须在管理节点上执行。

2.  安装当前版本的 [Docker Compose](/manuals/compose/install/_index.md)。

## 准备 Docker 仓库（Registry）

由于 Swarm 由多台 Docker Engine 组成，需要一个仓库来向所有节点分发镜像。你可以使用 [Docker Hub](https://hub.docker.com) 或自建仓库。以下示例演示如何创建一个临时测试用的本地仓库，完成后即可删除。

1.  在集群中以服务的方式启动仓库：

    ```console
    $ docker service create --name registry --publish published=5000,target=5000 registry:2
    ```

2.  使用 `docker service ls` 查看状态：

    ```console
    $ docker service ls

    ID            NAME      REPLICAS  IMAGE                                                                               COMMAND
    l7791tpuwkco  registry  1/1       registry:2@sha256:1152291c7f93a4ea2ddc95e46d142c31e743b6dd70e194af9e6ebe530f782c17
    ```

    当 `REPLICAS` 一栏为 `1/1` 时表示已运行；若为 `0/1`，通常是镜像仍在拉取中。

3.  使用 `curl` 验证服务是否可用：

    ```console
    $ curl http://127.0.0.1:5000/v2/

    {}
    ```

## 创建示例应用

本指南的示例应用改自《[Docker Compose 入门](/manuals/compose/gettingstarted.md)》中的计数器应用。它包含一个 Python 程序，通过 Redis 维护访问计数，每访问一次即自增。

1.  为项目创建目录：

    ```console
    $ mkdir stackdemo
    $ cd stackdemo
    ```

2.  在项目目录中创建 `app.py`，并粘贴如下内容：

    ```python
    from flask import Flask
    from redis import Redis

    app = Flask(__name__)
    redis = Redis(host='redis', port=6379)

    @app.route('/')
    def hello():
        count = redis.incr('hits')
        return 'Hello World! I have been seen {} times.\n'.format(count)

    if __name__ == "__main__":
        app.run(host="0.0.0.0", port=8000, debug=True)
    ```

3.  创建 `requirements.txt`，写入以下两行：

    ```text
    flask
    redis
    ```

4.  创建 `Dockerfile`，写入以下内容：

    ```dockerfile
    # syntax=docker/dockerfile:1
    FROM python:3.4-alpine
    ADD . /code
    WORKDIR /code
    RUN pip install -r requirements.txt
    CMD ["python", "app.py"]
    ```

5.  创建 `compose.yaml`，写入以下内容：

    ```yaml
      services:
        web:
          image: 127.0.0.1:5000/stackdemo
          build: .
          ports:
            - "8000:8000"
        redis:
          image: redis:alpine
    ```

    Web 应用镜像由上面的 Dockerfile 构建，并打上 `127.0.0.1:5000` 的仓库标签（即前面创建的本地仓库地址）。这一步对将应用分发到 Swarm 至关重要。


## 使用 Compose 进行本地验证

1.  通过 `docker compose up` 启动应用。该命令会构建 Web 镜像、拉取（如需）Redis 镜像，并创建两个容器。

    你可能会看到关于引擎处于 Swarm 模式的警告。原因是 Compose 不会利用 Swarm，多数情况下仅在单个节点部署。这里可以忽略该警告。

    ```console
    $ docker compose up -d

    WARNING: The Docker Engine you're using is running in swarm mode.

    Compose does not use swarm mode to deploy services to multiple nodes in
    a swarm. All containers are scheduled on the current node.

    To deploy your application across the swarm, use `docker stack deploy`.

    Creating network "stackdemo_default" with the default driver
    Building web
    ...(build output)...
    Creating stackdemo_redis_1
    Creating stackdemo_web_1
    ```

2.  使用 `docker compose ps` 查看应用是否运行：

    ```console
    $ docker compose ps

          Name                     Command               State           Ports
    -----------------------------------------------------------------------------------
    stackdemo_redis_1   docker-entrypoint.sh redis ...   Up      6379/tcp
    stackdemo_web_1     python app.py                    Up      0.0.0.0:8000->8000/tcp
    ```

    使用 `curl` 测试应用：

    ```console
    $ curl http://localhost:8000
    Hello World! I have been seen 1 times.

    $ curl http://localhost:8000
    Hello World! I have been seen 2 times.

    $ curl http://localhost:8000
    Hello World! I have been seen 3 times.
    ```

3.  停止并移除应用：

    ```console
    $ docker compose down --volumes

    Stopping stackdemo_web_1 ... done
    Stopping stackdemo_redis_1 ... done
    Removing stackdemo_web_1 ... done
    Removing stackdemo_redis_1 ... done
    Removing network stackdemo_default
    ```


## 将构建出的镜像推送到仓库

要在 Swarm 中分发 Web 应用镜像，需要先推送到前面设置的仓库。使用 Compose 推送非常简单：

```console
$ docker compose push

Pushing web (127.0.0.1:5000/stackdemo:latest)...
The push refers to a repository [127.0.0.1:5000/stackdemo]
5b5a49501a76: Pushed
be44185ce609: Pushed
bd7330a79bcf: Pushed
c9fc143a069a: Pushed
011b303988d2: Pushed
latest: digest: sha256:a81840ebf5ac24b42c1c676cbda3b2cb144580ee347c07e1bc80e35e5ca76507 size: 1372
```

现在，应用栈已经可以部署了。


## 将应用栈部署到 Swarm

1.  使用 `docker stack deploy` 创建栈：

    ```console
    $ docker stack deploy --compose-file compose.yaml stackdemo

    Ignoring unsupported options: build

    Creating network stackdemo_default
    Creating service stackdemo_web
    Creating service stackdemo_redis
    ```

    最后的参数是栈名。创建后，网络、卷与服务的名称都会加上该栈名前缀。

2.  使用 `docker stack services stackdemo` 查看运行状态：

    ```console
    $ docker stack services stackdemo

    ID            NAME             MODE        REPLICAS  IMAGE
    orvjk2263y1p  stackdemo_redis  replicated  1/1       redis:3.2-alpine@sha256:f1ed3708f538b537eb9c2a7dd50dc90a706f7debd7e1196c9264edeea521a86d
    s1nf0xy8t1un  stackdemo_web    replicated  1/1       127.0.0.1:5000/stackdemo@sha256:adb070e0805d04ba2f92c724298370b7a4eb19860222120d43e0f6351ddbc26f
    ```

    当两项服务的 `REPLICAS` 都显示为 `1/1` 时表示已就绪。若为多节点集群，因涉及镜像拉取，可能需要等待片刻。

    同前，你可以用 `curl` 测试应用：

    ```console
    $ curl http://localhost:8000
    Hello World! I have been seen 1 times.

    $ curl http://localhost:8000
    Hello World! I have been seen 2 times.

    $ curl http://localhost:8000
    Hello World! I have been seen 3 times.
    ```

    借助 Docker 内置的路由网格，你可以访问 Swarm 中任一节点的 `8000` 端口，都会被路由到应用：

    ```console
    $ curl http://address-of-other-node:8000
    Hello World! I have been seen 4 times.
    ```

3.  使用 `docker stack rm` 删除应用栈：

    ```console
    $ docker stack rm stackdemo

    Removing service stackdemo_web
    Removing service stackdemo_redis
    Removing network stackdemo_default
    ```

4.  使用 `docker service rm` 删除仓库服务：

    ```console
    $ docker service rm registry
    ```

5.  如果你只是在本机测试，完成后想退出 Swarm 模式，可使用：

    ```console
    $ docker swarm leave --force

    Node left the swarm.
    ```
