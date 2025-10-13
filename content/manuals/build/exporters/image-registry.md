---
title: 镜像与仓库导出器
description: |
  镜像与仓库导出器会创建一个镜像，可加载到本地镜像存储，或推送到远端仓库。
keywords: build, buildx, buildkit, exporter, image, registry
aliases:
  - /build/building/exporters/image-registry/
---

`image` 导出器会将构建结果输出为容器镜像格式；`registry` 导出器与其相同，
但会通过设置 `push=true` 自动将结果推送到仓库。

## 用法概览

使用 `image` 与 `registry` 导出器构建容器镜像：

```console
$ docker buildx build --output type=image[,parameters] .
$ docker buildx build --output type=registry[,parameters] .
```

下表说明了在 `type=image` 时可通过 `--output` 传入的参数：

| 参数                    | 类型                                   | 默认值  | 说明                                                                                                                                                                                                                               |
| ---------------------- | -------------------------------------- | ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `name`                 | String                                 |         | 指定镜像名称（可多个）。                                                                                                                                                                                                            |
| `push`                 | `true`,`false`                         | `false` | 创建镜像后推送。                                                                                                                                                                                                                    |
| `push-by-digest`       | `true`,`false`                         | `false` | 不带名称，仅按摘要（digest）推送镜像。                                                                                                                                                                                              |
| `registry.insecure`    | `true`,`false`                         | `false` | 允许推送到不安全的仓库。                                                                                                                                                                                                            |
| `dangling-name-prefix` | `<value>`                              |         | 以 `prefix@<digest>` 命名镜像，用于匿名镜像场景。                                                                                                                                                                                   |
| `name-canonical`       | `true`,`false`                         |         | 追加规范化名称 `name@<digest>`。                                                                                                                                                                                                    |
| `compression`          | `uncompressed`,`gzip`,`estargz`,`zstd` | `gzip`  | 压缩类型，参见[压缩][1]。                                                                                                                                                                                                           |
| `compression-level`    | `0..22`                                |         | 压缩级别，参见[压缩][1]。                                                                                                                                                                                                           |
| `force-compression`    | `true`,`false`                         | `false` | 强制应用压缩，参见[压缩][1]。                                                                                                                                                                                                       |
| `rewrite-timestamp`    | `true`,`false`                         | `false` | 将文件时间戳重写为 `SOURCE_DATE_EPOCH` 值。如何指定该值，参见[构建可复现性][4]。                                                                                                                                                     |
| `oci-mediatypes`       | `true`,`false`                         | `false` | 在导出器清单中使用 OCI 媒体类型，参见[OCI 媒体类型][2]。                                                                                                                                                                            |
| `oci-artifact`         | `true`,`false`                         | `false` | 将证明（attestation）按 OCI Artifact 格式化，参见[OCI 媒体类型][2]。                                                                                                                                                                |
| `unpack`               | `true`,`false`                         | `false` | 创建后解包镜像（用于 containerd）。                                                                                                                                                                                                 |
| `store`                | `true`,`false`                         | `true`  | 将结果镜像存入 worker（例如 containerd）的镜像存储，并确保镜像的所有 blob 已在内容存储中可用。若 worker 没有镜像存储（如使用 OCI workers），则忽略。                                                                              |
| `annotation.<key>`     | String                                 |         | 为已构建的镜像附加注解，对应 `key` 与 `value`，参见[注解][3]。                                                                                                                                                                      |

[1]: _index.md#compression
[2]: _index.md#oci-media-types
[3]: #annotations
[4]: https://github.com/moby/buildkit/blob/master/docs/build-repro.md
[5]: /manuals/build/metadata/attestations/_index.md#attestations-as-oci-artifacts

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

关于 `image` 与 `registry` 导出器的更多信息，参见
[BuildKit README](https://github.com/moby/buildkit/blob/master/README.md#imageregistry)。
