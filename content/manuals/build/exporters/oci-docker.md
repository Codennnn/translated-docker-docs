---
title: OCI 与 Docker 导出器
keywords: build, buildx, buildkit, exporter, oci, docker
description: >
  OCI 与 Docker 导出器会在本地文件系统上创建镜像布局的 tar 包
aliases:
  - /build/building/exporters/oci-docker/
---

`oci` 导出器会将构建结果导出为
[OCI 镜像布局](https://github.com/opencontainers/image-spec/blob/main/image-layout.md)
的 tar 包；`docker` 导出器行为相同，但导出为 Docker 镜像布局。

[`docker` 驱动](/manuals/build/builders/drivers/docker.md)不支持这些导出器。
如需生成这些输出，必须使用 `docker-container` 或其他驱动。

## 用法概览

使用 `oci` 与 `docker` 导出器构建容器镜像：

```console
$ docker buildx build --output type=oci[,parameters] .
```

```console
$ docker buildx build --output type=docker[,parameters] .
```

下表描述了可用参数：

| 参数                 | 类型                                   | 默认值  | 说明                                                                                                                             |
| ------------------- | -------------------------------------- | ------- | -------------------------------------------------------------------------------------------------------------------------------- |
| `name`              | String                                 |         | 指定镜像名称（可多个）。                                                                                                         |
| `dest`              | String                                 |         | 目标路径。                                                                                                                       |
| `tar`               | `true`,`false`                         | `true`  | 将输出打包为 tarball 布局。                                                                                                      |
| `compression`       | `uncompressed`,`gzip`,`estargz`,`zstd` | `gzip`  | 压缩类型，参见[压缩][1]。                                                                                                        |
| `compression-level` | `0..22`                                |         | 压缩级别，参见[压缩][1]。                                                                                                        |
| `force-compression` | `true`,`false`                         | `false` | 强制应用压缩，参见[压缩][1]。                                                                                                    |
| `oci-mediatypes`    | `true`,`false`                         |         | 在导出器清单中使用 OCI 媒体类型。`type=oci` 默认 `true`，`type=docker` 默认 `false`。参见[OCI 媒体类型][2]。                     |
| `annotation.<key>`  | String                                 |         | 为已构建的镜像附加注解，对应 `key` 与 `value`，参见[注解][3]。                                                                  |

[1]: _index.md#compression
[2]: _index.md#oci-media-types
[3]: #annotations

## 注解

这些导出器支持通过 `annotation` 参数添加 OCI 注解，后接点号表示法的注解名称。
如下示例设置了 `org.opencontainers.image.title` 注解：

```console
$ docker buildx build \
    --output "type=<type>,name=<registry>/<image>,annotation.org.opencontainers.image.title=<title>" .
```

关于注解的更多信息，参见
[BuildKit 文档](https://github.com/moby/buildkit/blob/master/docs/annotations.md)。

## 延伸阅读

关于 `oci` 或 `docker` 导出器的更多信息，参见
[BuildKit README](https://github.com/moby/buildkit/blob/master/README.md#docker-tarball)。
