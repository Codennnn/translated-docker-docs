---
description: 了解支撑存储驱动的相关技术。
keywords: 容器, 存储, 驱动, btrfs, overlayfs, vfs, zfs
title: 存储驱动
weight: 40
aliases:
  - /storage/storagedriver/imagesandcontainers/
  - /storage/storagedriver/
  - /engine/userguide/storagedriver/imagesandcontainers/
---

要想高效使用存储驱动，你需要理解 Docker 如何构建并存储镜像，以及容器如何使用这些镜像。有了这些背景，你就能在数据持久化方案上做出更合适的选择，并尽量避免性能问题。

## 存储驱动 vs Docker 卷

Docker 使用存储驱动来保存镜像层，以及容器可写层中的数据。容器删除后，其可写层不会保留，适合存放运行期产生的临时数据。存储驱动在空间占用方面做了优化，但（取决于具体驱动）写入速度通常低于原生文件系统，尤其是使用写时复制（copy-on-write, CoW）文件系统的驱动。写入密集型的应用（如数据库）会受到额外的性能开销影响，特别是当只读层中存在大量既有数据时。

对于写入密集型数据、需要跨越容器生命周期持久化的数据，或需要在多个容器之间共享的数据，请使用 Docker 卷。参见[卷](../volumes.md)一节，了解如何利用卷来实现持久化并提升性能。

## 镜像与层

Docker 镜像由一组层构成。每一层对应 Dockerfile 中的一条指令。除最后一层外，其余各层都是只读的。看下面这个 Dockerfile：

```dockerfile
# syntax=docker/dockerfile:1

FROM ubuntu:22.04
LABEL org.opencontainers.image.authors="org@example.com"
COPY . /app
RUN make /app
RUN rm -r $HOME/.cache
CMD python /app/app.py
```

这个 Dockerfile 共包含 4 条命令。任何修改文件系统的命令都会创建新的一层。`FROM` 语句首先基于 `ubuntu:22.04` 镜像创建一层。`LABEL` 仅修改镜像的元数据，不产生新层。`COPY` 从当前目录拷贝文件到镜像中。第一个 `RUN` 使用 `make` 构建应用，并将结果写入新层。第二个 `RUN` 删除缓存目录，同样写入新层。最后，`CMD` 指定容器启动时运行的命令，它只修改镜像元数据，也不会产生新层。

每一层只记录相对于前一层的差异。请注意，新增与删除文件都会生成新层。以上例子中，`$HOME/.cache` 目录被删除，但它仍存在于更早的层中，并计入镜像总体体积。参见[编写 Dockerfile 的最佳实践](/manuals/build/building/best-practices.md)和[多阶段构建](/manuals/build/building/multi-stage.md)，了解如何优化 Dockerfile 以生成更高效的镜像。

这些层自下而上叠加在一起。创建新容器时，会在最上方添加一个可写层，通常称为“容器层”。运行中的容器对文件系统的所有修改（新增、修改、删除）都会写入这个很薄的可写层。下图展示了一个基于 `ubuntu:15.04` 镜像的容器层次结构。

![Layers of a container based on the Ubuntu image](images/container-layers.webp?w=450&h=300)

存储驱动负责这些层之间如何协作的具体实现。Docker 提供了多种存储驱动，它们在不同场景各有优缺点。

## 容器与层

容器与镜像的关键区别在于顶部的可写层。容器中新增或修改的数据都会写入该可写层。删除容器时，这一可写层也会被删除，底层镜像不会被改变。

由于每个容器都有自己的可写层，所有更改都存放在容器层中，因此多个容器可以共享同一个底层镜像，同时各自维护独立的数据状态。下图展示了多个容器共享同一 Ubuntu 15.04 镜像的情形。

![Containers sharing the same image](images/sharing-layers.webp?w=600&h=300)

Docker 使用存储驱动来管理镜像层与容器可写层的内容。不同的驱动实现方式不同，但都基于可堆叠的镜像层与写时复制（CoW）策略。

> [!NOTE]
>
> 如果需要多个容器访问完全相同的数据，请使用 Docker 卷。参见[卷](../volumes.md)一节了解更多。

## 磁盘上的容器大小

要查看正在运行容器的大致体积，可使用 `docker ps -s` 命令。输出中有两列与大小相关：

- `size`：该容器可写层在磁盘上占用的数据量。
- `virtual size`：该容器使用的只读镜像数据与其可写层 `size` 之和。多个容器可能共享部分或全部只读镜像数据。两个基于同一镜像创建的容器共享 100% 的只读数据；而不同镜像但存在公共层的容器共享这些公共层。因此，不能简单将所有 `virtual size` 相加，否则会显著高估总磁盘占用。

所有正在运行容器在磁盘上的总体占用，由各容器的 `size` 与 `virtual size` 综合决定。如果多个容器基于同一个镜像启动，那么这些容器的总占用大致为：所有容器 `size` 之和，再加上一个镜像大小（即 `virtual size - size`）。

此外，容器还会通过以下方式占用磁盘空间，这些并未计入上面的数值：

- 由[日志驱动](/manuals/engine/logging/_index.md)保存的日志文件。如果容器产生日志量很大且未配置日志轮转，这部分可能相当可观。
- 容器使用的卷与绑定挂载。
- 容器配置文件占用的磁盘空间（通常很小）。
- 写入磁盘的内存页（如果启用了交换）。
- 检查点文件（如果你使用实验性的检查点/恢复功能）。

## 写时复制（CoW）策略

写时复制是一种通过共享与按需拷贝文件来提升效率的策略。如果某个文件或目录已经存在于镜像的较低层，其他层（包括可写层）在读取时会直接复用该文件。首次需要修改该文件时（无论在镜像构建阶段还是容器运行时），存储驱动会将其拷贝到当前层并在此基础上修改。这样可以最小化 I/O 并控制后续各层的体积。下面将更详细地解释这些优势。

### 共享让镜像更小

当你使用 `docker pull` 从仓库拉取镜像，或从本地尚不存在的镜像创建容器时，各层会被分别拉取并存储到 Docker 的本地存储目录（Linux 主机上通常为 `/var/lib/docker/`）。如下所示：

```console
$ docker pull ubuntu:22.04
22.04: Pulling from library/ubuntu
f476d66f5408: Pull complete
8882c27f669e: Pull complete
d9af21273955: Pull complete
f5029279ec12: Pull complete
Digest: sha256:6120be6a2b7ce665d0cbddc3ce6eae60fe94637c6a66985312d1f02f63cc0bcd
Status: Downloaded newer image for ubuntu:22.04
docker.io/library/ubuntu:22.04
```

每一层都会存放在宿主机本地存储区域的独立目录中。要查看这些层在文件系统中的表现，可以列出 `/var/lib/docker/<storage-driver>` 的内容。下例使用的是 `overlay2` 存储驱动：

```console
$ ls /var/lib/docker/overlay2
16802227a96c24dcbeab5b37821e2b67a9f921749cd9a2e386d5a6d5bc6fc6d3
377d73dbb466e0bc7c9ee23166771b35ebdbe02ef17753d79fd3571d4ce659d7
3f02d96212b03e3383160d31d7c6aeca750d2d8a1879965b89fe8146594c453d
ec1ec45792908e90484f7e629330666e7eee599f08729c93890a7205a6ba35f5
l
```

这些目录名与层 ID 并不一一对应。

假设你有两份不同的 Dockerfile。第一份用于构建名为 `acme/my-base-image:1.0` 的镜像。

```dockerfile
# syntax=docker/dockerfile:1
FROM alpine
RUN apk add --no-cache bash
```

第二份基于 `acme/my-base-image:1.0`，并新增了一些层：

```dockerfile
# syntax=docker/dockerfile:1
FROM acme/my-base-image:1.0
COPY . /app
RUN chmod +x /app/hello.sh
CMD /app/hello.sh
```

第二个镜像包含第一个镜像的所有层，并在此基础上新增了由 `COPY` 与 `RUN` 指令产生的层，以及一个读写的容器层。由于第一镜像的各层本地已存在，Docker 无需再次拉取；两个镜像会共享它们的公共层。

基于这两份 Dockerfile 构建镜像后，你可以通过 `docker image ls` 与 `docker image history` 验证共享层的加密 ID 是否一致。

1. 新建目录 `cow-test/` 并进入。

2. 在 `cow-test/` 下创建文件 `hello.sh`，内容如下：

   ```bash
   #!/usr/bin/env bash
   echo "Hello world"
   ```

3. 将前面第一份 Dockerfile 的内容保存为 `Dockerfile.base`。

4. 将第二份 Dockerfile 的内容保存为 `Dockerfile`。

5. 在 `cow-test/` 目录中构建第一个镜像。注意命令末尾的 `.`，它用于设置构建上下文，告知 Docker 在何处查找需要添加到镜像中的文件。

   ```console
   $ docker build -t acme/my-base-image:1.0 -f Dockerfile.base .
   [+] Building 6.0s (11/11) FINISHED
   => [internal] load build definition from Dockerfile.base                                      0.4s
   => => transferring dockerfile: 116B                                                           0.0s
   => [internal] load .dockerignore                                                              0.3s
   => => transferring context: 2B                                                                0.0s
   => resolve image config for docker.io/docker/dockerfile:1                                     1.5s
   => [auth] docker/dockerfile:pull token for registry-1.docker.io                               0.0s
   => CACHED docker-image://docker.io/docker/dockerfile:1@sha256:9e2c9eca7367393aecc68795c671... 0.0s
   => [internal] load .dockerignore                                                              0.0s
   => [internal] load build definition from Dockerfile.base                                      0.0s
   => [internal] load metadata for docker.io/library/alpine:latest                               0.0s
   => CACHED [1/2] FROM docker.io/library/alpine                                                 0.0s
   => [2/2] RUN apk add --no-cache bash                                                          3.1s
   => exporting to image                                                                         0.2s
   => => exporting layers                                                                        0.2s
   => => writing image sha256:da3cf8df55ee9777ddcd5afc40fffc3ead816bda99430bad2257de4459625eaa   0.0s
   => => naming to docker.io/acme/my-base-image:1.0                                              0.0s
   ```

6. 构建第二个镜像。

   ```console
   $ docker build -t acme/my-final-image:1.0 -f Dockerfile .

   [+] Building 3.6s (12/12) FINISHED
   => [internal] load build definition from Dockerfile                                            0.1s
   => => transferring dockerfile: 156B                                                            0.0s
   => [internal] load .dockerignore                                                               0.1s
   => => transferring context: 2B                                                                 0.0s
   => resolve image config for docker.io/docker/dockerfile:1                                      0.5s
   => CACHED docker-image://docker.io/docker/dockerfile:1@sha256:9e2c9eca7367393aecc68795c671...  0.0s
   => [internal] load .dockerignore                                                               0.0s
   => [internal] load build definition from Dockerfile                                            0.0s
   => [internal] load metadata for docker.io/acme/my-base-image:1.0                               0.0s
   => [internal] load build context                                                               0.2s
   => => transferring context: 340B                                                               0.0s
   => [1/3] FROM docker.io/acme/my-base-image:1.0                                                 0.2s
   => [2/3] COPY . /app                                                                           0.1s
   => [3/3] RUN chmod +x /app/hello.sh                                                            0.4s
   => exporting to image                                                                          0.1s
   => => exporting layers                                                                         0.1s
   => => writing image sha256:8bd85c42fa7ff6b33902ada7dcefaaae112bf5673873a089d73583b0074313dd    0.0s
   => => naming to docker.io/acme/my-final-image:1.0                                              0.0s
   ```

7. 查看镜像大小。

   ```console
   $ docker image ls

   REPOSITORY             TAG     IMAGE ID         CREATED               SIZE
   acme/my-final-image    1.0     8bd85c42fa7f     About a minute ago    7.75MB
   acme/my-base-image     1.0     da3cf8df55ee     2 minutes ago         7.75MB
   ```

8. 查看各镜像的历史记录。

   ```console
   $ docker image history acme/my-base-image:1.0

   IMAGE          CREATED         CREATED BY                                      SIZE      COMMENT
   da3cf8df55ee   5 minutes ago   RUN /bin/sh -c apk add --no-cache bash # bui…   2.15MB    buildkit.dockerfile.v0
   <missing>      7 weeks ago     /bin/sh -c #(nop)  CMD ["/bin/sh"]              0B
   <missing>      7 weeks ago     /bin/sh -c #(nop) ADD file:f278386b0cef68136…   5.6MB
   ```

   其中部分步骤大小为 `0B`，这表示仅修改了元数据，不会生成镜像层，也不占用除元数据外的体积。以上输出表明该镜像包含 2 个镜像层。

   ```console
   $ docker image history  acme/my-final-image:1.0

   IMAGE          CREATED         CREATED BY                                      SIZE      COMMENT
   8bd85c42fa7f   3 minutes ago   CMD ["/bin/sh" "-c" "/app/hello.sh"]            0B        buildkit.dockerfile.v0
   <missing>      3 minutes ago   RUN /bin/sh -c chmod +x /app/hello.sh # buil…   39B       buildkit.dockerfile.v0
   <missing>      3 minutes ago   COPY . /app # buildkit                          222B      buildkit.dockerfile.v0
   <missing>      4 minutes ago   RUN /bin/sh -c apk add --no-cache bash # bui…   2.15MB    buildkit.dockerfile.v0
   <missing>      7 weeks ago     /bin/sh -c #(nop)  CMD ["/bin/sh"]              0B
   <missing>      7 weeks ago     /bin/sh -c #(nop) ADD file:f278386b0cef68136…   5.6MB
   ```

   可以看到，第一个镜像的所有步骤也包含在最终镜像中。最终镜像由第一个镜像的两层加上第二个镜像新增的两层组成。

   在 `docker history` 的输出中，`<missing>` 表示这些步骤要么是在其他系统上构建并属于从 Docker Hub 拉取的 `alpine` 镜像的一部分，要么是使用 BuildKit 构建的。早期的“经典”构建器会为每一步生成一个“中间镜像”用于缓存，`IMAGE` 列会显示该镜像的 ID。
   
   BuildKit 使用自有的缓存机制，不再需要中间镜像来做缓存。参见 [BuildKit](/manuals/build/buildkit/_index.md) 了解更多增强点。

9. 查看各镜像包含的层

   使用 `docker image inspect` 查看各镜像中层的加密 ID：

   ```console
   $ docker image inspect --format "{{json .RootFS.Layers}}" acme/my-base-image:1.0
   [
     "sha256:72e830a4dff5f0d5225cdc0a320e85ab1ce06ea5673acfe8d83a7645cbd0e9cf",
     "sha256:07b4a9068b6af337e8b8f1f1dae3dd14185b2c0003a9a1f0a6fd2587495b204a"
   ]
   ```
   
   ```console
   $ docker image inspect --format "{{json .RootFS.Layers}}" acme/my-final-image:1.0
   [
     "sha256:72e830a4dff5f0d5225cdc0a320e85ab1ce06ea5673acfe8d83a7645cbd0e9cf",
     "sha256:07b4a9068b6af337e8b8f1f1dae3dd14185b2c0003a9a1f0a6fd2587495b204a",
     "sha256:cc644054967e516db4689b5282ee98e4bc4b11ea2255c9630309f559ab96562e",
     "sha256:e84fb818852626e89a09f5143dbc31fe7f0e0a6a24cd8d2eb68062b904337af4"
   ]
   ```

   可以看到，两者的前两层完全一致，第二个镜像额外增加了两层。共享的镜像层在 `/var/lib/docker/` 中只会存储一次，并且在向镜像仓库推送或拉取时也会被共享，从而减少网络带宽与存储占用。

   > [!TIP]
   >
   > 使用 `--format` 可以自定义 Docker 命令输出。
   > 
   > 上述示例使用 `docker image inspect` 搭配 `--format` 查看各层 ID，并将其格式化为 JSON 数组。`--format` 能帮助你在不借助 `awk` 或 `sed` 的情况下，直接提取并格式化关键信息。更多格式化用法参见[格式化命令与日志输出](/manuals/engine/cli/formatting.md)。我们还使用了 [`jq` 工具](https://stedolan.github.io/jq/) 对 JSON 做了更友好的展示。

### 拷贝让容器更高效

启动容器时，系统会在其他层之上添加一个很薄的可写容器层。容器对文件系统的任何更改都会写入该层；未被修改的文件不会被拷贝到可写层，从而使可写层尽可能小。

当容器中的现有文件被修改时，存储驱动会执行一次写时复制操作。不同驱动的具体步骤有所差异。以 `overlay2` 为例，大致流程如下：

*  在镜像各层中查找需要更新的文件。从最新一层开始，逐层向下追溯到基础层。命中结果会加入缓存，以加速后续操作。
*  对首次找到的文件副本执行 `copy_up`，将其拷贝到容器的可写层。
*  随后的修改都发生在这个副本上；容器将看不到位于更低层的只读副本。

Btrfs、ZFS 等驱动在写时复制上的行为不同。稍后在各驱动的详细说明中你可以看到更多实现差异。

写入数据较多的容器会比其他容器消耗更多空间，因为大多数写操作都会占用可写顶层中的新空间。注意，即使只是修改文件的元数据（如权限或所有者），也可能触发 `copy_up`，从而将该文件复制到可写层。

> [!TIP]
>
> 写入密集型应用请使用卷。
>
> 不要在容器内部直接存放写入密集型应用的数据。例如数据库类应用在只读层有既有数据时尤其容易出现性能问题。
> 
> 推荐使用独立于运行中容器的 Docker 卷，它为 I/O 效率而设计；卷也可在容器间共享，并且不会增加容器可写层的体积。参见[使用卷](../volumes.md)了解更多。

`copy_up` 操作会带来可感知的性能开销，开销大小与所用存储驱动有关。当文件很大、层数较多或目录层级很深时，这种影响会更明显。好在每个文件在第一次被修改时才会触发 `copy_up`。

下面的步骤基于我们之前构建的 `acme/my-final-image:1.0` 启动 5 个容器，并观察它们的空间占用，以验证写时复制的行为。

1. 在 Docker 宿主机的终端中运行以下 `docker run` 命令。末尾输出的是各容器的 ID：

   ```console
   $ docker run -dit --name my_container_1 acme/my-final-image:1.0 bash \
     && docker run -dit --name my_container_2 acme/my-final-image:1.0 bash \
     && docker run -dit --name my_container_3 acme/my-final-image:1.0 bash \
     && docker run -dit --name my_container_4 acme/my-final-image:1.0 bash \
     && docker run -dit --name my_container_5 acme/my-final-image:1.0 bash

   40ebdd7634162eb42bdb1ba76a395095527e9c0aa40348e6c325bd0aa289423c
   a5ff32e2b551168b9498870faf16c9cd0af820edf8a5c157f7b80da59d01a107
   3ed3c1a10430e09f253704116965b01ca920202d52f3bf381fbb833b8ae356bc
   939b3bf9e7ece24bcffec57d974c939da2bdcc6a5077b5459c897c1e2fa37a39
   cddae31c314fbab3f7eabeb9b26733838187abc9a2ed53f97bd5b04cd7984a5a
   ```

2. 使用带 `--size` 选项的 `docker ps` 验证 5 个容器正在运行，并查看各自大小：

   
   ```console
   $ docker ps --size --format "table {{.ID}}\t{{.Image}}\t{{.Names}}\t{{.Size}}"

   CONTAINER ID   IMAGE                     NAMES            SIZE
   cddae31c314f   acme/my-final-image:1.0   my_container_5   0B (virtual 7.75MB)
   939b3bf9e7ec   acme/my-final-image:1.0   my_container_4   0B (virtual 7.75MB)
   3ed3c1a10430   acme/my-final-image:1.0   my_container_3   0B (virtual 7.75MB)
   a5ff32e2b551   acme/my-final-image:1.0   my_container_2   0B (virtual 7.75MB)
   40ebdd763416   acme/my-final-image:1.0   my_container_1   0B (virtual 7.75MB)
   ```
   
   可见所有容器共享镜像的只读层（7.75MB），由于未向容器文件系统写入数据，因此没有额外的存储占用。

   {{< accordion title="进阶：容器的元数据与日志占用" >}}
   
   > [!NOTE]
   >
   > 该步骤需要在 Linux 机器上进行，Docker Desktop 无法完成，因为它需要访问 Docker 守护进程的文件存储。
   
   `docker ps` 的输出仅反映容器可写层的磁盘占用，不包含各容器的元数据与日志文件信息。
   
   你可以查看 Docker 守护进程的存储目录（默认 `/var/lib/docker`）以获取更多细节：
   
   ```console
   $ sudo du -sh /var/lib/docker/containers/*
   
   36K  /var/lib/docker/containers/3ed3c1a10430e09f253704116965b01ca920202d52f3bf381fbb833b8ae356bc
   36K  /var/lib/docker/containers/40ebdd7634162eb42bdb1ba76a395095527e9c0aa40348e6c325bd0aa289423c
   36K  /var/lib/docker/containers/939b3bf9e7ece24bcffec57d974c939da2bdcc6a5077b5459c897c1e2fa37a39
   36K  /var/lib/docker/containers/a5ff32e2b551168b9498870faf16c9cd0af820edf8a5c157f7b80da59d01a107
   36K  /var/lib/docker/containers/cddae31c314fbab3f7eabeb9b26733838187abc9a2ed53f97bd5b04cd7984a5a
   ```
   
   可以看到，每个容器仅占用约 36K 的空间。

   {{< /accordion >}}

3. 按容器维度的存储变化

   为了演示这一点，向 `my_container_1`、`my_container_2`、`my_container_3` 的可写层写入一个单词 `hello`：

   ```console
   $ for i in {1..3}; do docker exec my_container_$i sh -c 'printf hello > /out.txt'; done
   ```
   
   随后再次执行 `docker ps` 可以看到，这些容器各自多了 5 字节的占用。这些数据属于各容器私有，不会被共享；容器的只读层不受影响，依然被所有容器共享。

   
   ```console
   $ docker ps --size --format "table {{.ID}}\t{{.Image}}\t{{.Names}}\t{{.Size}}"

   CONTAINER ID   IMAGE                     NAMES            SIZE
   cddae31c314f   acme/my-final-image:1.0   my_container_5   0B (virtual 7.75MB)
   939b3bf9e7ec   acme/my-final-image:1.0   my_container_4   0B (virtual 7.75MB)
   3ed3c1a10430   acme/my-final-image:1.0   my_container_3   5B (virtual 7.75MB)
   a5ff32e2b551   acme/my-final-image:1.0   my_container_2   5B (virtual 7.75MB)
   40ebdd763416   acme/my-final-image:1.0   my_container_1   5B (virtual 7.75MB)
   ```

上述示例展示了写时复制如何帮助容器更高效。它不仅节省空间，还能缩短容器启动时间。创建容器（或从同一镜像创建多个容器）时，Docker 只需创建一个薄薄的可写容器层即可。

如果每次创建容器都要复制整套底层镜像栈，容器创建时间和磁盘占用都会显著增加。这类似虚拟机的做法——每台虚拟机都有一个或多个虚拟磁盘。[`vfs` 存储](vfs-driver.md)不提供 CoW 文件系统或其他优化，使用它时，每个容器都会得到镜像数据的完整拷贝。

## 相关信息

* [卷](../volumes.md)
* [选择存储驱动](select-storage-driver.md)
