---
title: 读懂镜像分层
keywords: concepts, build, images, container, docker desktop
description: 本文将为你讲解容器镜像的分层结构。
summary: |
  你是否想过镜像究竟如何工作？本指南将帮助你理解镜像分层——
  容器镜像的基本构建单元。你将系统掌握分层是如何创建、叠加与复用，
  以及它们如何帮助你构建高效、可优化的容器。
weight: 1
aliases: 
 - /guides/docker-concepts/building-images/understanding-image-layers/
---

{{< youtube-embed wJwqtAkmtQA >}}

## 概念解析

正如你在[什么是镜像？](../the-basics/what-is-an-image/)中所学到的，容器镜像由多层构成，并且每一层一旦创建就不可变。那么这在实践中意味着什么？这些层又是如何共同构建出容器可用的文件系统的？

### 镜像分层

镜像中的每一层都包含一组对文件系统的变更——新增、删除或修改文件。我们先看一个理论示例：

1. 第一层添加基础命令与包管理器（如 apt）。
2. 第二层安装 Python 运行时与依赖管理工具 pip。
3. 第三层拷贝应用的 `requirements.txt` 文件。
4. 第四层安装该应用所需的依赖。
5. 第五层拷贝应用的源代码。

上述示例可以抽象为：

![展示镜像分层概念的流程图](images/container_image_layers.webp?border=true)

分层带来的一个重要收益是：层可以在不同镜像之间复用。比如，你要再创建一个 Python 应用，就可以复用相同的 Python 基础层。这既能加速构建，也能减少分发镜像所需的存储与带宽。其分层大致如下：

![展示镜像分层复用收益的流程图](images/container_image_layer_reuse.webp?border=true)

通过分层，你可以在他人镜像的基础层上进行扩展，只需添加你自己的应用所需的数据即可。

### 层的叠加

分层之所以可行，得益于“内容可寻址存储”和“联合文件系统”。原理略偏技术，但大致流程如下：

1. 每一层下载完成后，都会被解压到宿主机文件系统中的独立目录。
2. 当你从镜像运行容器时，会创建一个联合文件系统，把各层按顺序叠加，呈现为一个统一的视图。
3. 容器启动时，其根目录会通过 `chroot` 指向这个统一视图所在的位置。

在创建联合文件系统时，除了镜像层，还会为正在运行的容器单独创建一个可写目录。容器对文件系统所做的变更会写入这个目录，而底层镜像层保持不变。这使你可以基于同一镜像同时运行多个容器。

## 动手试试

在本实践中，你将使用 [`docker container commit`](https://docs.docker.com/reference/cli/docker/container/commit/) 命令手动创建新的镜像层。需要注意的是，实际工作中你很少用这种方式创建镜像，通常会[使用 Dockerfile](./writing-a-dockerfile.md)。不过这种方式更有助于你直观理解其工作原理。

### 创建基础镜像

第一步，你将创建一个自己的基础镜像，后续步骤都会基于它进行。

1. [下载并安装](https://www.docker.com/products/docker-desktop/) Docker Desktop。

2. 在终端中运行以下命令以启动一个新容器：

    ```console
    $ docker run --name=base-container -ti ubuntu
    ```

    镜像下载并启动容器后，你会看到一个新的命令提示符。它运行在容器内部，类似如下（容器 ID 会不同）：

    ```console
    root@d8c5ca119fcd:/#
    ```

3. 在容器内运行以下命令安装 Node.js：

    ```console
    $ apt update && apt install -y nodejs
    ```

    该命令会在容器中下载并安装 Node。在联合文件系统的语境里，这些文件系统变更会写入该容器专属的目录。

4. 运行以下命令验证 Node 是否安装成功：

    ```console
    $ node -e 'console.log("Hello world!")'
    ```

    你应当能在终端中看到 “Hello world!” 输出。

5. 现在 Node 已经安装完成，你可以把刚才的改动保存为一个新的镜像层，用它来启动新容器或构建新的镜像。为此，使用 [`docker container commit`](https://docs.docker.com/reference/cli/docker/container/commit/) 命令，在新终端中运行：

    ```console
    $ docker container commit -m "Add node" base-container node-base
    ```

6. 使用 `docker image history` 命令查看镜像的层：

    ```console
    $ docker image history node-base
    ```

    你会看到类似如下输出：

    ```console
    IMAGE          CREATED          CREATED BY                                      SIZE      COMMENT
    d5c1fca2cdc4   10 seconds ago   /bin/bash                                       126MB     Add node
    2b7cc08dcdbb   5 weeks ago      /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B
    <missing>      5 weeks ago      /bin/sh -c #(nop) ADD file:07cdbabf782942af0…   69.2MB
    <missing>      5 weeks ago      /bin/sh -c #(nop)  LABEL org.opencontainers.…   0B
    <missing>      5 weeks ago      /bin/sh -c #(nop)  LABEL org.opencontainers.…   0B
    <missing>      5 weeks ago      /bin/sh -c #(nop)  ARG LAUNCHPAD_BUILD_ARCH     0B
    <missing>      5 weeks ago      /bin/sh -c #(nop)  ARG RELEASE                  0B
    ```

    注意顶部一行中的 “Add node” 注释。这一层包含了你刚安装的 Node.js。

7. 为证明你的镜像已包含 Node，可基于新镜像启动一个容器：

    ```console
    $ docker run node-base node -e "console.log('Hello again')"
    ```

    你应能在终端看到 “Hello again” 的输出，表明 Node 已安装并可正常运行。

8. 完成基础镜像构建后，你可以移除该容器：

    ```console
    $ docker rm -f base-container
    ```

> **基础镜像的定义**
>
> 基础镜像是构建其他镜像的地基。理论上你可以把任何镜像当作基础镜像，
> 但也有一些镜像是专门为“作为基石”而设计，用来作为应用的起点或基座。
>
> 在本例中，你大概率不会直接部署 `node-base`，因为它本身并没有实际功能；
> 但它可以作为你后续构建的基础。

### 构建应用镜像

现在你已有一个基础镜像，可以在其之上继续构建新的镜像。

1. 使用刚创建的 node-base 镜像启动一个新容器：

    ```console
    $ docker run --name=app-container -ti node-base
    ```

2. 在该容器中创建一个 Node 程序：

    ```console
    $ echo 'console.log("Hello from an app")' > app.js
    ```

    使用以下命令运行该程序，你将看到屏幕打印出消息：

    ```console
    $ node app.js
    ```

3. 在另一个终端中，将该容器的更改保存为新的镜像：

    ```console
    $ docker container commit -c "CMD node app.js" -m "Add app" app-container sample-app
    ```

    上述命令不仅创建了一个名为 `sample-app` 的新镜像，还为其添加了额外配置，
    指定容器启动时的默认命令。在本例中，默认命令会自动执行 `node app.js`。

4. 在容器外的终端中查看更新后的分层信息：

    ```console
    $ docker image history sample-app
    ```

    你会看到类似如下输出。注意最上层的注释为 “Add app”，紧接着的下一层为 “Add node”：

    ```console
    IMAGE          CREATED              CREATED BY                                      SIZE      COMMENT
    c1502e2ec875   About a minute ago   /bin/bash                                       33B       Add app
    5310da79c50a   4 minutes ago        /bin/bash                                       126MB     Add node
    2b7cc08dcdbb   5 weeks ago          /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B
    <missing>      5 weeks ago          /bin/sh -c #(nop) ADD file:07cdbabf782942af0…   69.2MB
    <missing>      5 weeks ago          /bin/sh -c #(nop)  LABEL org.opencontainers.…   0B
    <missing>      5 weeks ago          /bin/sh -c #(nop)  LABEL org.opencontainers.…   0B
    <missing>      5 weeks ago          /bin/sh -c #(nop)  ARG LAUNCHPAD_BUILD_ARCH     0B
    <missing>      5 weeks ago          /bin/sh -c #(nop)  ARG RELEASE                  0B
    ```

5. 使用新镜像启动一个容器。由于已经设置了默认命令，可以直接运行：

    ```console
    $ docker run sample-app
    ```

    你应能在终端看到来自该 Node 程序的问候语。

6. 完成后，可用以下命令清理容器：

    ```console
    $ docker rm -f app-container
    ```

## 延伸阅读

如果你想更深入学习本文涉及的内容，推荐参考：

* [`docker image history`](/reference/cli/docker/image/history/)
* [`docker container commit`](/reference/cli/docker/container/commit/)

## 下一步

前文已提示，大多数镜像的构建并不会使用 `docker container commit`。
通常你会使用 Dockerfile 来自动化上述步骤。

{{< button text="编写 Dockerfile" url="writing-a-dockerfile" >}}
