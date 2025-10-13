---
title: 注解
description: 注解用于为 OCI 镜像补充元数据
keywords: build, buildkit, annotations, metadata
aliases:
- /build/building/annotations/
---

<!-- vale Docker.Spacing = NO -->

注解用于为镜像提供描述性的元数据。你可以把任意信息记录为注解并附加到镜像上，帮助使用者与工具更好地理解镜像的来源、内容，以及正确的使用方式。

注解与 [标签][labels] 类似，某种程度上也有重叠：二者的共同点都是为资源附加元数据。一般来说，可以这样区分它们：

- 注解用于描述 OCI 镜像的组成部分，例如 [清单][manifests]、[索引][indexes] 和 [描述符][descriptors]。
- 标签用于描述 Docker 资源，例如镜像、容器、网络和卷。

OCI 镜像[规范][specification]定义了注解的格式，并提供了一组预定义的注解键。遵循该规范，像 Docker Scout 这样的工具就能以一致且自动化的方式展示镜像的元数据信息。

不要将注解与[证明][attestations]混淆：

- 证明（attestation）包含镜像是如何构建的以及其包含了哪些内容的信息。证明会作为镜像索引上的独立清单进行附加，且目前未被 OCI 标准化。
- 注解包含关于镜像的任意元数据。注解可以以“标签”的形式附加在镜像的[配置][config]上，或者作为属性附加到镜像索引或清单中。

## 添加注解

你可以在构建时添加注解，也可以在创建镜像清单或索引时添加。

> [!NOTE]
>
> Docker Engine 的镜像存储不支持加载带有注解的镜像。若需要使用注解进行构建，请确保将镜像直接推送到仓库，可使用 `--push` CLI 标志或[仓库导出器](/manuals/build/exporters/image-registry.md)。

在命令行中指定注解时，可在 `docker build` 命令中使用 `--annotation` 标志：

```console
$ docker build --push --annotation "foo=bar" .
```

如果你使用的是 [Bake](/manuals/build/bake/_index.md)，可以通过 `annotations` 属性为某个目标指定注解：

```hcl
target "default" {
  output = ["type=registry"]
  annotations = ["foo=bar"]
}
```

若想在 GitHub Actions 中为构建的镜像添加注解，请参见
[使用 GitHub Actions 添加镜像注解](/manuals/build/ci/github-actions/annotations.md)。

你也可以在使用 `docker buildx imagetools create` 创建镜像时添加注解。该命令仅支持为索引或清单描述符添加注解，参见
[CLI 参考](/reference/cli/docker/buildx/imagetools/create.md#annotation)。

## 查看注解

要查看“镜像索引”上的注解，使用 `docker buildx imagetools inspect`。该命令会展示索引以及其中各描述符（对清单的引用）的注解。下面的示例展示了描述符上的 `org.opencontainers.image.documentation` 注解，以及索引上的 `org.opencontainers.image.authors` 注解。

```console {hl_lines=["10-12","19-21"]}
$ docker buildx imagetools inspect <IMAGE> --raw
{
  "schemaVersion": 2,
  "mediaType": "application/vnd.oci.image.index.v1+json",
  "manifests": [
    {
      "mediaType": "application/vnd.oci.image.manifest.v1+json",
      "digest": "sha256:d20246ef744b1d05a1dd69d0b3fa907db007c07f79fe3e68c17223439be9fefb",
      "size": 911,
      "annotations": {
        "org.opencontainers.image.documentation": "https://foo.example/docs",
      },
      "platform": {
        "architecture": "amd64",
        "os": "linux"
      }
    },
  ],
  "annotations": {
    "org.opencontainers.image.authors": "dvdksn"
  }
}
```

要查看某个清单的注解，继续使用 `docker buildx imagetools inspect`，并指定 `<IMAGE>@<DIGEST>`，其中 `<DIGEST>` 是该清单的摘要：

```console {hl_lines="22-25"}
$ docker buildx imagetools inspect <IMAGE>@sha256:d20246ef744b1d05a1dd69d0b3fa907db007c07f79fe3e68c17223439be9fefb --raw
{
  "schemaVersion": 2,
  "mediaType": "application/vnd.oci.image.manifest.v1+json",
  "config": {
    "mediaType": "application/vnd.oci.image.config.v1+json",
    "digest": "sha256:4368b6959a78b412efa083c5506c4887e251f1484ccc9f0af5c406d8f76ece1d",
    "size": 850
  },
  "layers": [
    {
      "mediaType": "application/vnd.oci.image.layer.v1.tar+gzip",
      "digest": "sha256:2c03dbb20264f09924f9eab176da44e5421e74a78b09531d3c63448a7baa7c59",
      "size": 3333033
    },
    {
      "mediaType": "application/vnd.oci.image.layer.v1.tar+gzip",
      "digest": "sha256:4923ad480d60a548e9b334ca492fa547a3ce8879676685b6718b085de5aaf142",
      "size": 61887305
    }
  ],
  "annotations": {
    "index,manifest:org.opencontainers.image.vendor": "foocorp",
    "org.opencontainers.image.source": "https://git.example/foo.git",
  }
}
```

## 指定注解级别

默认情况下，注解会添加到镜像清单上。你可以通过在注解字符串前加上类型前缀，来指定注解要附加到哪个级别（OCI 镜像组件）：

```console
$ docker build --annotation "<TYPE>:<KEY>=<VALUE>" .
```

支持的类型包括：

- `manifest`：为清单添加注解。
- `index`：为根索引添加注解。
- `manifest-descriptor`：为索引中的清单描述符添加注解。
- `index-descriptor`：为镜像布局中的索引描述符添加注解。

例如，将注解 `foo=bar` 附加到镜像索引：

```console
$ docker build --tag <IMAGE> --push --annotation "index:foo=bar" .
```

注意，你指定的组件必须在本次构建产物中实际存在，否则构建会失败。比如，下面的命令就不可行，因为 `docker` 导出器不会生成索引：

```console
$ docker build --output type=docker --annotation "index:foo=bar" .
```

同理，以下示例在某些情况下也不可行，因为在显式禁用可溯源信息（provenance）证明时，buildx 可能会默认生成 `docker` 输出：

```console
$ docker build --provenance=false --annotation "index:foo=bar" .
```

你也可以通过逗号分隔指定多个类型，从而将同一注解添加到多个级别。下面的示例会同时在镜像索引与镜像清单上添加 `foo=bar` 注解：

```console
$ docker build --tag <IMAGE> --push --annotation "index,manifest:foo=bar" .
```

此外，还可以在类型前缀中用中括号指定平台限定符，以仅为匹配特定操作系统与架构的组件添加注解。下面的示例仅将 `foo=bar` 注解添加到 `linux/amd64` 清单：

```console
$ docker build --tag <IMAGE> --push --annotation "manifest[linux/amd64]:foo=bar" .
```

## 相关信息

相关阅读：

- [使用 GitHub Actions 添加镜像注解](/manuals/build/ci/github-actions/annotations.md)
- [注解的 OCI 规范][specification]

参考资料：

- [`docker buildx build --annotation`](/reference/cli/docker/buildx/build.md#annotation)
- [Bake 文件参考：`annotations`](/manuals/build/bake/reference.md#targetannotations)
- [`docker buildx imagetools create --annotation`](/reference/cli/docker/buildx/imagetools/create.md#annotation)

<!-- links -->

[specification]: https://github.com/opencontainers/image-spec/blob/main/annotations.md
[attestations]: /manuals/build/metadata/attestations/_index.md
[config]: https://github.com/opencontainers/image-spec/blob/main/config.md
[descriptors]: https://github.com/opencontainers/image-spec/blob/main/descriptor.md
[indexes]: https://github.com/opencontainers/image-spec/blob/main/image-index.md
[labels]: /manuals/engine/manage-resources/labels.md
[manifests]: https://github.com/opencontainers/image-spec/blob/main/manifest.md
