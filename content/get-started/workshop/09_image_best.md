---
title: 镜像构建最佳实践
weight: 90
linkTitle: "第 8 部分：镜像构建最佳实践"
keywords: 入门, 安装, 导览, 快速开始, 介绍, 概念, 容器,
  Docker Desktop
description: 为你的应用构建镜像的实用建议
aliases:
 - /get-started/09_image_best/
 - /guides/workshop/09_image_best/
---

## 镜像分层

使用 `docker image history` 命令，你可以查看用于创建镜像中每一层的命令。

1. 使用 `docker image history` 查看你创建的 `getting-started` 镜像的各层信息。

    ```console
    $ docker image history getting-started
    ```

    你将看到类似如下的输出：

    ```plaintext
    IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
    a78a40cbf866        18 seconds ago      /bin/sh -c #(nop)  CMD ["node" "src/index.j…    0B                  
    f1d1808565d6        19 seconds ago      /bin/sh -c yarn install --production            85.4MB              
    a2c054d14948        36 seconds ago      /bin/sh -c #(nop) COPY dir:5dc710ad87c789593…   198kB               
    9577ae713121        37 seconds ago      /bin/sh -c #(nop) WORKDIR /app                  0B                  
    b95baba1cfdb        13 days ago         /bin/sh -c #(nop)  CMD ["node"]                 0B                  
    <missing>           13 days ago         /bin/sh -c #(nop)  ENTRYPOINT ["docker-entry…   0B                  
    <missing>           13 days ago         /bin/sh -c #(nop) COPY file:238737301d473041…   116B                
    <missing>           13 days ago         /bin/sh -c apk add --no-cache --virtual .bui…   5.35MB              
    <missing>           13 days ago         /bin/sh -c #(nop)  ENV YARN_VERSION=1.21.1      0B                  
    <missing>           13 days ago         /bin/sh -c addgroup -g 1000 node     && addu…   74.3MB              
    <missing>           13 days ago         /bin/sh -c #(nop)  ENV NODE_VERSION=12.14.1     0B                  
    <missing>           13 days ago         /bin/sh -c #(nop)  CMD ["/bin/sh"]              0B                  
    <missing>           13 days ago         /bin/sh -c #(nop) ADD file:e69d441d729412d24…   5.59MB   
    ```

    每一行代表镜像中的一层。显示顺序是底部为基础层、顶部为最新的一层。借助该输出，你还能快速看到每层的大小，帮助诊断镜像过大的问题。

2. 你会注意到部分行被截断。加入 `--no-trunc` 标志即可查看完整输出。

    ```console
    $ docker image history --no-trunc getting-started
    ```

## 分层缓存

了解了分层机制后，有一个关键点可以帮助你缩短镜像的构建时间：一旦某一层发生变化，其后的所有下游层都需要重新构建。

先回顾你为该入门应用编写的 Dockerfile：

```dockerfile
# syntax=docker/dockerfile:1
FROM node:lts-alpine
WORKDIR /app
COPY . .
RUN yarn install --production
CMD ["node", "src/index.js"]
```

结合前面的镜像历史输出可知，Dockerfile 中的每条命令都会生成一个新的镜像层。你可能还记得，只要镜像发生变更，yarn 依赖就会被重新安装。每次构建都重复打包相同依赖并不高效。

为此，你需要重构 Dockerfile，以更好地利用依赖的缓存。对于基于 Node 的应用，依赖声明在 `package.json` 中。可以先只拷贝该文件并安装依赖，再拷贝其余代码。这样，仅当 `package.json` 发生变化时，才会重新安装 yarn 依赖。

1. 将 Dockerfile 修改为先拷贝 `package.json`，安装依赖，再拷贝其余文件。

   ```dockerfile
   # syntax=docker/dockerfile:1
   FROM node:lts-alpine
   WORKDIR /app
   COPY package.json yarn.lock ./
   RUN yarn install --production
   COPY . .
   CMD ["node", "src/index.js"]
   ```

2. 使用 `docker build` 构建新镜像。

    ```console
    $ docker build -t getting-started .
    ```

    你会看到类似如下的输出：

    ```plaintext
    [+] Building 16.1s (10/10) FINISHED
    => [internal] load build definition from Dockerfile
    => => transferring dockerfile: 175B
    => [internal] load .dockerignore
    => => transferring context: 2B
    => [internal] load metadata for docker.io/library/node:lts-alpine
    => [internal] load build context
    => => transferring context: 53.37MB
    => [1/5] FROM docker.io/library/node:lts-alpine
    => CACHED [2/5] WORKDIR /app
    => [3/5] COPY package.json yarn.lock ./
    => [4/5] RUN yarn install --production
    => [5/5] COPY . .
    => exporting to image
    => => exporting layers
    => => writing image     sha256:d6f819013566c54c50124ed94d5e66c452325327217f4f04399b45f94e37d25
    => => naming to docker.io/library/getting-started
    ```

3. 现在修改 `src/static/index.html` 文件，例如把 `<title>` 改成 "The Awesome Todo App"。

4. 再次执行 `docker build -t getting-started .` 构建镜像。这一次，你会看到不同的输出：

    ```plaintext
    [+] Building 1.2s (10/10) FINISHED
    => [internal] load build definition from Dockerfile
    => => transferring dockerfile: 37B
    => [internal] load .dockerignore
    => => transferring context: 2B
    => [internal] load metadata for docker.io/library/node:lts-alpine
    => [internal] load build context
    => => transferring context: 450.43kB
    => [1/5] FROM docker.io/library/node:lts-alpine
    => CACHED [2/5] WORKDIR /app
    => CACHED [3/5] COPY package.json yarn.lock ./
    => CACHED [4/5] RUN yarn install --production
    => [5/5] COPY . .
    => exporting to image
    => => exporting layers
    => => writing image     sha256:91790c87bcb096a83c2bd4eb512bc8b134c757cda0bdee4038187f98148e2eda
    => => naming to docker.io/library/getting-started
    ```

    首先，构建明显更快了。同时有多个步骤复用了缓存的镜像层。推送与拉取该镜像（及其更新）也会更快。

## 多阶段构建

多阶段构建是一种非常强大的方法，允许你通过多个阶段来创建镜像，带来多重好处：

- 将构建时依赖与运行时依赖分离
- 仅包含运行所需内容，从而减小镜像体积

### Maven/Tomcat 示例

在构建基于 Java 的应用时，需要 JDK 来将源码编译为字节码。但是在生产运行时并不需要 JDK。同时，像 Maven、Gradle 这样的构建工具也只在构建阶段需要，最终镜像里无需包含。多阶段构建正好能解决这些问题。

```dockerfile
# syntax=docker/dockerfile:1
FROM maven AS build
WORKDIR /app
COPY . .
RUN mvn package

FROM tomcat
COPY --from=build /app/target/file.war /usr/local/tomcat/webapps 
```

在这个示例中，你使用一个名为 `build` 的阶段用 Maven 完成 Java 构建；在第二个阶段（从 `FROM tomcat` 开始）从 `build` 阶段复制产物。最终镜像只包含最后一个阶段的内容（也可以通过 `--target` 指定要构建的阶段）。

### React 示例

在构建 React 应用时，需要 Node 运行时将 JS（通常是 JSX）、SASS 等编译为静态的 HTML、JS 与 CSS。如果没有服务端渲染，生产环境甚至不需要 Node 运行时；你可以把静态资源放入一个精简的 nginx 容器中发布。

```dockerfile
# syntax=docker/dockerfile:1
FROM node:lts AS build
WORKDIR /app
COPY package* yarn.lock ./
RUN yarn install
COPY public ./public
COPY src ./src
RUN yarn run build

FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html
```

在上面的 Dockerfile 中，先用 `node:lts` 镜像执行构建（最大化利用分层缓存），再将构建产物复制到 nginx 容器中。

## 小结

本节介绍了几条镜像构建的最佳实践，包括分层缓存与多阶段构建。

相关资料：
 - [Dockerfile reference](/reference/dockerfile/)
 - [Dockerfile best practices](/manuals/build/building/best-practices.md)

## 下一步

下一节将带你继续了解学习容器的更多资源。

{{< button text="接下来做什么" url="10_what_next.md" >}}
