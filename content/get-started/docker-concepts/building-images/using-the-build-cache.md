---
title: 使用构建缓存
keywords: concepts, build, images, container, docker desktop
description: 本文介绍构建缓存、哪些更改会使缓存失效，以及如何高效使用构建缓存。
summary: |
  合理使用构建缓存可以复用以往构建的结果，从而跳过不必要的步骤，
  大幅缩短构建时间。要想最大化缓存命中并避免高开销、耗时的全量重建，
  需要理解缓存失效机制。本指南将教你如何高效利用 Docker 构建缓存，
  优化镜像开发与持续集成工作流。
weight: 4
aliases: 
 - /guides/docker-concepts/building-images/using-the-build-cache/
---

{{< youtube-embed Ri6jMknjprY >}}

## 概念解析

回顾你在[编写 Dockerfile](./writing-a-dockerfile/) 为 getting-started 应用创建的示例：

```dockerfile
FROM node:22-alpine
WORKDIR /app
COPY . .
RUN yarn install --production
CMD ["node", "./src/index.js"]
```

当你运行 `docker build` 创建新镜像时，Docker 会按顺序执行 Dockerfile 中的每条指令，并为每条指令创建一层。在执行每条指令时，Docker 都会检查是否可以复用之前构建的结果。如果发现某条指令此前已经执行过且上下文一致，Docker 将直接复用缓存的结果，而无需重复执行。这样就能显著加速构建、节省时间与资源。

高效利用构建缓存的关键在于理解“缓存何时会失效”。以下情况会导致缓存被判定为失效：

- 对 `RUN` 指令中命令的任何更改，都会使对应层的缓存失效。Docker 会比对命令是否发生变化，一旦变更就会丢弃该层缓存。
- 对通过 `COPY` 或 `ADD` 指令复制到镜像中的任何文件发生变化。无论是文件内容，还是如权限等属性变化，都会被视为缓存失效的触发条件。
- 一旦某一层缓存失效，其后的所有层也会一并失效。包括基础镜像或中间层在内的任意前置层发生变化，依赖它们的后续层都会被重新构建，以确保一致性。

在编写或修改 Dockerfile 时，应尽量避免不必要的缓存未命中，以保证构建尽可能快速、稳定。

## 动手试试

本实践将演示如何在一个 Node.js 应用中有效利用 Docker 构建缓存。

### 构建应用

1. [下载并安装](https://www.docker.com/products/docker-desktop/) Docker Desktop。

2. 打开终端，[克隆示例应用](https://github.com/dockersamples/todo-list-app)。

    ```console
    $ git clone https://github.com/dockersamples/todo-list-app
    ```

3. 进入 `todo-list-app` 目录：

    ```console
    $ cd todo-list-app
    ```

    在该目录下，你会看到一个 `Dockerfile`，内容如下：

    ```dockerfile
    FROM node:22-alpine
    WORKDIR /app
    COPY . .
    RUN yarn install --production
    EXPOSE 3000
    CMD ["node", "./src/index.js"]
    ```

4. 执行以下命令构建镜像：

    ```console
    $ docker build .
    ```

    构建结果类似：

    ```console
    [+] Building 20.0s (10/10) FINISHED
    ```

    第一行表示整个构建耗时约为 20.0 秒。首次构建通常较慢，因为需要安装依赖。

5. 在不做任何修改的情况下再次构建。

   如下所示，再次执行 `docker build`，且不修改源码或 Dockerfile：

    ```console
    $ docker build .
    ```

   除首次构建外，后续构建若命令与上下文均未变化，将显著提速。这是因为 Docker 会缓存中间层；当 Dockerfile 与源码均未变更时，构建可以直接复用缓存层。

    ```console
    [+] Building 1.0s (9/9) FINISHED                                                                            docker:desktop-linux
     => [internal] load build definition from Dockerfile                                                                        0.0s
     => => transferring dockerfile: 187B                                                                                        0.0s
     ...
     => [internal] load build context                                                                                           0.0s
     => => transferring context: 8.16kB                                                                                         0.0s
     => CACHED [2/4] WORKDIR /app                                                                                               0.0s
     => CACHED [3/4] COPY . .                                                                                                   0.0s
     => CACHED [4/4] RUN yarn install --production                                                                              0.0s
     => exporting to image                                                                                                      0.0s
     => => exporting layers                                                                                                     0.0s
     => => exporting manifest
   ```

   可以看到，通过复用缓存层，二次构建仅耗时约 1.0 秒，无需重复执行诸如依赖安装等耗时步骤。

   <table>
     <thead>
       <tr>
         <th>步骤</th>
         <th>说明</th>
         <th>首次用时</th>
         <th>二次用时</th>
       </tr>
     </thead>
     <tbody>
       <tr>
         <td>1</td>
         <td><code>从 Dockerfile 读取构建定义</code></td>
         <td>0.0s</td>
         <td>0.0s</td>
       </tr>
       <tr>
         <td>2</td>
         <td><code>加载 docker.io/library/node:22-alpine 元数据</code></td>
         <td>2.7s</td>
         <td>0.9s</td>
       </tr>
       <tr>
         <td>3</td>
         <td><code>加载 .dockerignore</code></td>
         <td>0.0s</td>
         <td>0.0s</td>
       </tr>
       <tr>
         <td>4</td>
         <td><code>加载构建上下文</code><p>(上下文大小：4.60MB)</p></td>
         <td>0.1s</td>
         <td>0.0s</td>
       </tr>
       <tr>
         <td>5</td>
         <td><code>设置工作目录（WORKDIR）</code></td>
         <td>0.1s</td>
         <td>0.0s</td>
       </tr>
       <tr>
         <td>6</td>
         <td><code>复制本地代码到容器</code></td>
         <td>0.0s</td>
         <td>0.0s</td>
       </tr>
       <tr>
         <td>7</td>
         <td><code>运行 yarn install --production</code></td>
         <td>10.0s</td>
         <td>0.0s</td>
       </tr>
       <tr>
         <td>8</td>
         <td><code>导出分层</code></td>
         <td>2.2s</td>
         <td>0.0s</td>
       </tr>
       <tr>
         <td>9</td>
         <td><code>导出最终镜像</code></td>
         <td>3.0s</td>
         <td>0.0s</td>
       </tr>
     </tbody>
   </table>

   回到 `docker image history` 的输出可以发现，Dockerfile 中的每条命令都会成为镜像中的一层。你可能注意到，每当对镜像做出更改时，`yarn` 依赖就需要重新安装。有没有办法避免每次都重装依赖？

   解决思路是：重构 Dockerfile，使依赖缓存在不需要时不会被无谓地失效。对于基于 Node 的应用，依赖由 `package.json`（以及 `yarn.lock`）定义。只有当这些文件变化时才需要重新安装依赖；若未变化，则应复用缓存。因此应先仅复制这两个文件，再安装依赖，最后复制其余文件。这样只有当 `package.json` 或 `yarn.lock` 改动时，依赖层才会被重建。

6. 更新 Dockerfile：先复制 `package.json` 与 `yarn.lock`，安装依赖，再复制其余内容。

    ```dockerfile
    FROM node:22-alpine
    WORKDIR /app
    COPY package.json yarn.lock ./
    RUN yarn install --production 
    COPY . . 
    EXPOSE 3000
    CMD ["node", "src/index.js"]
    ```

7. 在与 Dockerfile 同级目录创建 `.dockerignore` 文件，内容如下：

    ```plaintext
    node_modules
    ```

8. 构建新镜像：

    ```console
    $ docker build .
    ```

    你将看到类似如下输出：

    ```console
    [+] Building 16.1s (10/10) FINISHED
    => [internal] load build definition from Dockerfile                                               0.0s
    => => transferring dockerfile: 175B                                                               0.0s
    => [internal] load .dockerignore                                                                  0.0s
    => => transferring context: 2B                                                                    0.0s
    => [internal] load metadata for docker.io/library/node:22-alpine                                  0.0s
    => [internal] load build context                                                                  0.8s
    => => transferring context: 53.37MB                                                               0.8s
    => [1/5] FROM docker.io/library/node:22-alpine                                                    0.0s
    => CACHED [2/5] WORKDIR /app                                                                      0.0s
    => [3/5] COPY package.json yarn.lock ./                                                           0.2s
    => [4/5] RUN yarn install --production                                                           14.0s
    => [5/5] COPY . .                                                                                 0.5s
    => exporting to image                                                                             0.6s
    => => exporting layers                                                                            0.6s
    => => writing image     
    sha256:d6f819013566c54c50124ed94d5e66c452325327217f4f04399b45f94e37d25        0.0s
    => => naming to docker.io/library/node-app:2.0                                                 0.0s
    ```

    由于 Dockerfile 改动较大，此处所有层重建是正常的。

9. 修改 `src/static/index.html`（例如把标题改成 "The Awesome Todo App"）。

10. 再次构建 Docker 镜像，这次输出会有所不同：

    ```console
    $ docker build -t node-app:3.0 .
    ```

    可能的输出如下：

    ```console
    [+] Building 1.2s (10/10) FINISHED 
    => [internal] load build definition from Dockerfile                                               0.0s
    => => transferring dockerfile: 37B                                                                0.0s
    => [internal] load .dockerignore                                                                  0.0s
    => => transferring context: 2B                                                                    0.0s
    => [internal] load metadata for docker.io/library/node:22-alpine                                  0.0s 
    => [internal] load build context                                                                  0.2s
    => => transferring context: 450.43kB                                                              0.2s
    => [1/5] FROM docker.io/library/node:22-alpine                                                    0.0s
    => CACHED [2/5] WORKDIR /app                                                                      0.0s
    => CACHED [3/5] COPY package.json yarn.lock ./                                                    0.0s
    => CACHED [4/5] RUN yarn install --production                                                     0.0s
    => [5/5] COPY . .                                                                                 0.5s 
    => exporting to image                                                                             0.3s
    => => exporting layers                                                                            0.3s
    => => writing image     
    sha256:91790c87bcb096a83c2bd4eb512bc8b134c757cda0bdee4038187f98148e2eda       0.0s
    => => naming to docker.io/library/node-app:3.0                                                 0.0s
    ```

    你会注意到构建速度明显加快，同时多个步骤复用了缓存层。这意味着你已经在有效使用构建缓存；后续推送与拉取镜像也会更快。

通过这些优化技巧，你可以让 Docker 构建更快、更高效，加速迭代并提升研发效率。

## 延伸阅读

* [使用缓存优化构建](/build/cache/)
* [缓存存储后端](/build/cache/backends/)
* [构建缓存失效](/build/cache/invalidation/)

## 下一步

现在你已经掌握了如何高效使用 Docker 构建缓存，可以继续学习多阶段构建。

{{< button text="多阶段构建" url="multi-stage-builds" >}}
