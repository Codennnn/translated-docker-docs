---
keywords: "API, Usage, plugins, documentation, developer"
title: 插件与服务
---

<!-- 本文件在 docker/cli GitHub 仓库维护：
     https://github.com/docker/cli/ 。请将所有 Pull Request 提交到该仓库。
     如果你在其他仓库看到本文件，请将其视为只读文件，因为它会被
     权威版本定期覆盖。在其他仓库中提交对此文件的修改将被拒绝。 -->

# 在 Docker 服务中使用卷与网络插件

在 Swarm 模式下，你可以创建允许附加由插件提供的网络或挂载由插件提供的数据卷的服务。Swarm 会根据节点上插件的可用性来调度服务。

### 卷插件

以下示例展示：在一个工作节点上安装卷插件并用该插件创建数据卷；随后在管理节点上创建服务并配置相应的挂载选项。你会看到该服务被调度到安装了该卷插件与数据卷的工作节点上运行。注意：node1 为管理节点，node2 为工作节点。

1. 准备管理节点（node1）：

    ```console
    $ docker swarm init
    Swarm initialized: current node (dxn1zf6l61qsb1josjja83ngz) is now a manager.
    ```

2. 在工作节点（node2）加入 Swarm、安装插件并创建数据卷：

    ```console
    $ docker swarm join \
      --token SWMTKN-1-49nj1cmql0jkz5s954yi3oex3nedyz0fb0xx14ie39trti4wxv-8vxv8rssmk743ojnwacrr2e7c \
      192.168.99.100:2377
    ```

    ```console
    $ docker plugin install tiborvass/sample-volume-plugin
    latest: Pulling from tiborvass/sample-volume-plugin
    eb9c16fbdc53: Download complete
    Digest: sha256:00b42de88f3a3e0342e7b35fa62394b0a9ceb54d37f4c50be5d3167899994639
    Status: Downloaded newer image for tiborvass/sample-volume-plugin:latest
    Installed plugin tiborvass/sample-volume-plugin
    ```

    ```console
    $ docker volume create -d tiborvass/sample-volume-plugin --name pluginVol
    ```

3. 在管理节点（node1）使用该插件与数据卷创建服务：

    ```console
    $ docker service create --name my-service --mount type=volume,volume-driver=tiborvass/sample-volume-plugin,source=pluginVol,destination=/tmp busybox top

    $ docker service ls
    z1sj8bb8jnfn  my-service   replicated  1/1       busybox:latest
    ```

    `docker service ls` 显示该服务有 1 个副本正在运行。

4. 在节点 2 上查看任务被调度的情况：

    ```console
    $ docker ps --format '{{.ID}}\t {{.Status}} {{.Names}} {{.Command}}'
    83fc1e842599     Up 2 days my-service.1.9jn59qzn7nbc3m0zt1hij12xs "top"
    ```

### 网络插件

本示例在管理节点与工作节点上都安装了一个全局作用域（global scope）的网络插件，并创建了一个具有多个副本的服务来使用该插件。我们将观察插件的可用性如何影响网络创建与容器调度。

注意：node1 为管理节点，node2 为工作节点。

1. 在管理节点与工作节点（node1 与 node2）上都安装一个全局作用域的网络插件：

    ```console
    $ docker plugin install bboreham/weave2
    Plugin "bboreham/weave2" is requesting the following privileges:
    - network: [host]
    - capabilities: [CAP_SYS_ADMIN CAP_NET_ADMIN]
    Do you grant the above permissions? [y/N] y
    latest: Pulling from bboreham/weave2
    7718f575adf7: Download complete
    Digest: sha256:2780330cc15644b60809637ee8bd68b4c85c893d973cb17f2981aabfadfb6d72
    Status: Downloaded newer image for bboreham/weave2:latest
    Installed plugin bboreham/weave2
    ```

2. 在管理节点（node1）上使用该插件创建网络：

    ```console
    $ docker network create --driver=bboreham/weave2:latest globalnet

    $ docker network ls
    NETWORK ID          NAME                DRIVER                   SCOPE
    qlj7ueteg6ly        globalnet           bboreham/weave2:latest   swarm
    ```

3. 在管理节点上创建一个具有 8 个副本的服务。可以观察到容器被调度到管理节点与工作节点上：

    在节点 1：

    ```console
    $ docker service create --network globalnet --name myservice --replicas=8 mrjana/simpleweb simpleweb
w90drnfzw85nygbie9kb89vpa
    ```

    ```console
    $ docker ps
    CONTAINER ID        IMAGE                                                                                      COMMAND             CREATED             STATUS              PORTS               NAMES
    87520965206a        mrjana/simpleweb@sha256:317d7f221d68c86d503119b0ea12c29de42af0a22ca087d522646ad1069a47a4   "simpleweb"         5 seconds ago       Up 4 seconds                            myservice.4.ytdzpktmwor82zjxkh118uf1v
    15e24de0f7aa        mrjana/simpleweb@sha256:317d7f221d68c86d503119b0ea12c29de42af0a22ca087d522646ad1069a47a4   "simpleweb"         5 seconds ago       Up 4 seconds                            myservice.2.kh7a9n3iauq759q9mtxyfs9hp
    c8c8f0144cdc        mrjana/simpleweb@sha256:317d7f221d68c86d503119b0ea12c29de42af0a22ca087d522646ad1069a47a4   "simpleweb"         5 seconds ago       Up 4 seconds                            myservice.6.sjhpj5gr3xt33e3u2jycoj195
    2e8e4b2c5c08        mrjana/simpleweb@sha256:317d7f221d68c86d503119b0ea12c29de42af0a22ca087d522646ad1069a47a4   "simpleweb"         5 seconds ago       Up 4 seconds                            myservice.8.2z29zowsghx66u2velublwmrh
    ```

    在节点 2：

    ```console
    $ docker ps
    CONTAINER ID        IMAGE                                                                                      COMMAND             CREATED             STATUS                  PORTS               NAMES
    53c0ae7c1dae        mrjana/simpleweb@sha256:317d7f221d68c86d503119b0ea12c29de42af0a22ca087d522646ad1069a47a4   "simpleweb"         2 seconds ago       Up Less than a second                       myservice.7.x44tvvdm3iwkt9kif35f7ykz1
    9b56c627fee0        mrjana/simpleweb@sha256:317d7f221d68c86d503119b0ea12c29de42af0a22ca087d522646ad1069a47a4   "simpleweb"         2 seconds ago       Up Less than a second                       myservice.1.x7n1rm6lltw5gja3ueikze57q
    d4f5927ba52c        mrjana/simpleweb@sha256:317d7f221d68c86d503119b0ea12c29de42af0a22ca087d522646ad1069a47a4   "simpleweb"         2 seconds ago       Up 1 second                                 myservice.5.i97bfo9uc6oe42lymafs9rz6k
    478c0d395bd7        mrjana/simpleweb@sha256:317d7f221d68c86d503119b0ea12c29de42af0a22ca087d522646ad1069a47a4   "simpleweb"         2 seconds ago       Up Less than a second                       myservice.3.yr7nkffa48lff1vrl2r1m1ucs
    ```

4. 缩容：在节点 1 上将副本数缩为 0：

    ```console
    $ docker service scale myservice=0
    myservice scaled to 0
    ```

5. 在工作节点（node2）上禁用并卸载插件：

    ```console
    $ docker plugin rm -f bboreham/weave2
    bboreham/weave2
    ```

6. 再次扩容。可以观察到，所有容器都被调度到管理节点上，而不会调度到工作节点，因为该插件在工作节点上已不可用。

    在节点 1：

    ```console
    $ docker service scale myservice=8
    myservice scaled to 8
    ```

    ```console
    $ docker ps
    CONTAINER ID        IMAGE                                                                                      COMMAND             CREATED             STATUS              PORTS               NAMES
    cf4b0ec2415e        mrjana/simpleweb@sha256:317d7f221d68c86d503119b0ea12c29de42af0a22ca087d522646ad1069a47a4   "simpleweb"         39 seconds ago      Up 36 seconds                           myservice.3.r7p5o208jmlzpcbm2ytl3q6n1
    57c64a6a2b88        mrjana/simpleweb@sha256:317d7f221d68c86d503119b0ea12c29de42af0a22ca087d522646ad1069a47a4   "simpleweb"         39 seconds ago      Up 36 seconds                           myservice.4.dwoezsbb02ccstkhlqjy2xe7h
    3ac68cc4e7b8        mrjana/simpleweb@sha256:317d7f221d68c86d503119b0ea12c29de42af0a22ca087d522646ad1069a47a4   "simpleweb"         39 seconds ago      Up 35 seconds                           myservice.5.zx4ezdrm2nwxzkrwnxthv0284
    006c3cb318fc        mrjana/simpleweb@sha256:317d7f221d68c86d503119b0ea12c29de42af0a22ca087d522646ad1069a47a4   "simpleweb"         39 seconds ago      Up 36 seconds                           myservice.8.q0e3umt19y3h3gzo1ty336k5r
    dd2ffebde435        mrjana/simpleweb@sha256:317d7f221d68c86d503119b0ea12c29de42af0a22ca087d522646ad1069a47a4   "simpleweb"         39 seconds ago      Up 36 seconds                           myservice.7.a77y3u22prjipnrjg7vzpv3ba
    a86c74d8b84b        mrjana/simpleweb@sha256:317d7f221d68c86d503119b0ea12c29de42af0a22ca087d522646ad1069a47a4   "simpleweb"         39 seconds ago      Up 36 seconds                           myservice.6.z9nbn14bagitwol1biveeygl7
    2846a7850ba0        mrjana/simpleweb@sha256:317d7f221d68c86d503119b0ea12c29de42af0a22ca087d522646ad1069a47a4   "simpleweb"         39 seconds ago      Up 37 seconds                           myservice.2.ypufz2eh9fyhppgb89g8wtj76
    e2ec01efcd8a        mrjana/simpleweb@sha256:317d7f221d68c86d503119b0ea12c29de42af0a22ca087d522646ad1069a47a4   "simpleweb"         39 seconds ago      Up 38 seconds                           myservice.1.8w7c4ttzr6zcb9sjsqyhwp3yl
    ```

    在节点 2：

    ```console
    $ docker ps
    CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
    ```
