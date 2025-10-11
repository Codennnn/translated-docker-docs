---
title: 构建驱动
description: 构建驱动定义了 BuildKit 后端如何以及在何处运行的配置。
keywords: build, buildx, driver, builder, docker-container, kubernetes, remote
aliases:
  - /build/buildx/drivers/
  - /build/building/drivers/
  - /build/buildx/multiple-builders/
  - /build/drivers/
---

构建驱动用于配置 BuildKit 后端的运行方式与运行位置。
驱动的设置可自定义，便于对构建器（builder）进行精细化控制。
Buildx 支持以下驱动：

- `docker`：使用打包在 Docker 守护进程中的 BuildKit 库。
- `docker-container`：通过 Docker 创建专用的 BuildKit 容器。
- `kubernetes`：在 Kubernetes 集群中创建 BuildKit Pod。
- `remote`：直接连接到手动管理的 BuildKit 守护进程。

不同驱动适用于不同的场景。默认的 `docker` 驱动强调简单易用，
对缓存与输出格式等高级特性的支持较为有限，且不可配置。
其他驱动提供了更高的灵活性，更擅长处理复杂的高级用例。

下表概述了各驱动之间的一些差异：

| 特性                         |  `docker`   | `docker-container` | `kubernetes` |      `remote`      |
| :--------------------------- | :---------: | :----------------: | :----------: | :----------------: |
| **自动加载镜像**             |     ✅      |                    |              |                    |
| **缓存导出**                 |     ✅\*     |         ✅         |      ✅      |         ✅         |
| **Tar 包输出**               |             |         ✅         |      ✅      |         ✅         |
| **多架构镜像**               |             |         ✅         |      ✅      |         ✅         |
| **BuildKit 配置**            |             |         ✅         |      ✅      | 外部管理          |

\* _`docker` 驱动并不支持所有缓存导出选项。
更多信息参见[缓存存储后端](/manuals/build/cache/backends/_index.md)。_

## 加载到本地镜像存储

与默认的 `docker` 驱动不同，使用其他驱动构建的镜像不会自动加载到本地镜像存储。
如果未显式指定输出，构建结果只会导出到构建缓存。

要使用非默认驱动构建并将镜像加载到镜像存储，请在构建命令中添加 `--load` 标志：

   ```console
   $ docker buildx build --load -t <image> --builder=container .
   ...
   => exporting to oci image format                                                                                                      7.7s
   => => exporting layers                                                                                                                4.9s
   => => exporting manifest sha256:4e4ca161fa338be2c303445411900ebbc5fc086153a0b846ac12996960b479d3                                      0.0s
   => => exporting config sha256:adf3eec768a14b6e183a1010cb96d91155a82fd722a1091440c88f3747f1f53f                                        0.0s
   => => sending tarball                                                                                                                 2.8s
   => importing to docker
   ```

   启用该选项后，构建完成时镜像会出现在本地镜像存储中：

   ```console
   $ docker image ls
   REPOSITORY                       TAG               IMAGE ID       CREATED             SIZE
   <image>                          latest            adf3eec768a1   2 minutes ago       197MB
   ```

### 默认加载

{{< summary-bar feature_name="Load by default" >}}

你可以将自定义构建驱动配置成类似默认 `docker` 驱动的行为，默认将镜像加载到本地镜像存储。
为此，可在创建构建器时设置 `default-load` 驱动选项：

```console
$ docker buildx create --driver-opt default-load=true
```

请注意，与 `docker` 驱动一致，若你通过 `--output` 指定了其他输出格式，
除非同时显式指定 `--output type=docker` 或使用 `--load` 标志，
否则结果不会被加载到镜像存储。

## 后续阅读

进一步阅读各驱动：

  - [Docker driver](./docker.md)
  - [Docker container driver](./docker-container.md)
  - [Kubernetes driver](./kubernetes.md)
- [Remote driver](./remote.md)
