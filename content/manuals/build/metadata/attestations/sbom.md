---
title: SBOM 证明（SBOM attestations）
keywords: build, attestations, sbom, spdx, metadata, packages
description: |
  SBOM 证明用于描述镜像中包含的软件构件，以及用于创建该镜像的构件。
aliases:
  - /build/attestations/sbom/
---

SBOM 证明通过校验镜像所包含的软件构件及其用于构建镜像的来源，帮助实现
[软件供应链透明性](/guides/docker-scout/s3c.md)。用于描述软件构件的
[SBOM](/guides/docker-scout/sbom.md) 中通常包含以下元数据：

- 构件名称
- 版本
- 许可证类型
- 作者
- 唯一包标识符

在构建阶段为镜像建立索引，相比在最终镜像上进行扫描更有优势。作为构建流程
的一部分进行扫描时，你还能发现用于构建镜像的软件（这些软件可能并不会出现在
最终镜像中）。

Docker 通过符合 SLSA 的构建流程（基于 BuildKit 与证明机制）来支持 SBOM 的
生成与证明。[BuildKit](/manuals/build/buildkit/_index.md) 生成的 SBOM 符合
SPDX 标准，并以 JSON 编码的 SPDX 文档附加到最终镜像，采用
[in-toto SPDX 谓词](https://github.com/in-toto/attestation/blob/main/spec/predicates/spdx.md)
定义的格式。本文将介绍如何使用 Docker 工具创建、管理与验证 SBOM 证明。

## 创建 SBOM 证明

要创建 SBOM 证明，请在 `docker buildx build` 命令中传入
`--attest type=sbom` 选项：

```console
$ docker buildx build --tag <namespace>/<image>:<version> \
    --attest type=sbom --push .
```

你也可以使用简写选项 `--sbom=true` 来替代 `--attest type=sbom`。

若要在 GitHub Actions 中添加 SBOM 证明，请参见
[通过 GitHub Actions 添加证明](/manuals/build/ci/github-actions/attestations.md)。

## 验证 SBOM 证明

在将镜像推送到仓库之前，务必先验证生成的 SBOM。

验证时，你可以使用 `local` 导出器进行构建。使用 `local` 导出器会将构建
结果保存到本地文件系统，而不是创建镜像。证明会写入导出根目录下的一个 JSON
文件。

```console
$ docker buildx build \
  --sbom=true \
  --output type=local,dest=out .
```

导出结果的根目录会生成名为 `sbom.spdx.json` 的 SBOM 文件：

```console
$ ls -1 ./out | grep sbom
sbom.spdx.json
```

## 构建参数

默认情况下，BuildKit 只会扫描镜像的最终阶段。生成的 SBOM 不包含在较早阶段
安装的构建期依赖，或存在于构建上下文中的依赖。这可能导致你忽略这些依赖的
漏洞，从而影响最终构建产物的安全性。

例如，你可能使用[多阶段构建](/manuals/build/building/multi-stage.md)，并在
最终阶段以 `FROM scratch` 开头以获得更小的镜像体积：

```dockerfile
FROM alpine AS build
# build the software ...

FROM scratch
COPY --from=build /path/to/bin /bin
ENTRYPOINT [ "/bin" ]
```

扫描使用上述 Dockerfile 构建得到的最终镜像，并不会暴露 `build` 阶段所用到的
构建期依赖。

若要在 SBOM 中包含 Dockerfile 的构建期依赖，可设置构建参数
`BUILDKIT_SBOM_SCAN_CONTEXT` 与 `BUILDKIT_SBOM_SCAN_STAGE`。这会将扫描
范围扩展到构建上下文以及其他阶段。

你可以将这些参数作为全局参数设置（在 Dockerfile 语法指令之后、第一条
`FROM` 之前），也可以在各个阶段分别设置。若设置为全局参数，其值会传播到
Dockerfile 的每个阶段。

`BUILDKIT_SBOM_SCAN_CONTEXT` 与 `BUILDKIT_SBOM_SCAN_STAGE` 属于特殊的构建
参数：它们不支持变量替换，且不能通过 Dockerfile 内的环境变量进行设置。
必须在 Dockerfile 中使用显式的 `ARG` 指令来设置这些值。

### 扫描构建上下文

若需扫描构建上下文，请将 `BUILDKIT_SBOM_SCAN_CONTEXT` 设为 `true`：

```dockerfile
# syntax=docker/dockerfile:1
ARG BUILDKIT_SBOM_SCAN_CONTEXT=true
FROM alpine AS build
# ...
```

你可以使用 `--build-arg` CLI 选项覆盖 Dockerfile 中的默认值：

```console
$ docker buildx build --tag <image>:<version> \
    --attest type=sbom \
    --build-arg BUILDKIT_SBOM_SCAN_CONTEXT=false .
```

注意：如果未在 Dockerfile 中通过 `ARG` 声明该参数，仅通过 CLI 传入该选项
是不会生效的。必须先在 Dockerfile 中声明相应的 `ARG`，之后才能通过
`--build-arg` 覆盖上下文扫描行为。

### 扫描构建阶段

若要扫描除最终阶段之外的更多阶段，可将 `BUILDKIT_SBOM_SCAN_STAGE` 参数设为
true（可作为全局参数，或在需要扫描的特定阶段单独设置）。下表展示了该参数
的不同配置方式：

| 值                                   | 说明                                   |
| ----------------------------------- | -------------------------------------- |
| `BUILDKIT_SBOM_SCAN_STAGE=true`     | 为当前阶段启用扫描                     |
| `BUILDKIT_SBOM_SCAN_STAGE=false`    | 为当前阶段禁用扫描                     |
| `BUILDKIT_SBOM_SCAN_STAGE=base,bin` | 仅为名为 `base` 与 `bin` 的阶段启用扫描 |

仅会扫描实际参与构建的阶段。不作为目标阶段依赖的阶段既不会被构建，也不会被
扫描。

下面的 Dockerfile 示例使用多阶段构建，借助
[Hugo](https://gohugo.io/) 构建一个静态网站：

```dockerfile
# syntax=docker/dockerfile:1
FROM alpine as hugo
ARG BUILDKIT_SBOM_SCAN_STAGE=true
WORKDIR /src
COPY <<config.yml ./
title: My Hugo website
config.yml
RUN apk add --upgrade hugo && hugo

FROM scratch
COPY --from=hugo /src/public /
```

在 `hugo` 阶段设置 `ARG BUILDKIT_SBOM_SCAN_STAGE=true`，可确保最终生成的
SBOM 中包含“创建该网站时使用了 Alpine Linux 与 Hugo”的信息。

使用 `local` 导出器构建该镜像会生成两个 JSON 文件：

```console
$ docker buildx build \
  --sbom=true \
  --output type=local,dest=out .
$ ls -1 out | grep sbom
sbom-hugo.spdx.json
sbom.spdx.json
```

## 查看 SBOM

若要查看通过 `image` 导出器导出的 SBOM，可使用
[`imagetools inspect`](/reference/cli/docker/buildx/imagetools/inspect.md)。

配合 `--format` 选项可指定输出模板。所有与 SBOM 相关的数据都可通过 `.SBOM`
属性访问。例如，获取 SPDX 格式的原始 SBOM 内容：

```console
$ docker buildx imagetools inspect <namespace>/<image>:<version> \
    --format "{{ json .SBOM.SPDX }}"
{
  "SPDXID": "SPDXRef-DOCUMENT",
  ...
}
```

> [!TIP]
>
> 若镜像为多平台构建，可使用
> `--format '{{ json (index .SBOM "linux/amd64").SPDX }}'`
> 查看特定平台索引对应的 SBOM。

你也可以使用 Go 模板的完整能力来构造更复杂的表达式。例如，列出所有已安装
的软件包及其版本：

```console
$ docker buildx imagetools inspect <namespace>/<image>:<version> \
    --format "{{ range .SBOM.SPDX.packages }}{{ .name }}@{{ .versionInfo }}{{ println }}{{ end }}"
adduser@3.118ubuntu2
apt@2.0.9
base-files@11ubuntu5.6
base-passwd@3.5.47
...
```

## SBOM 生成器

BuildKit 通过扫描器插件来生成 SBOM。默认使用的是
[BuildKit Syft scanner](https://github.com/docker/buildkit-syft-scanner)
插件。该插件基于开源工具
[Anchore 的 Syft](https://github.com/anchore/syft) 构建，用于生成 SBOM。

你可以通过 `generator` 选项指定实现了
[BuildKit SBOM 扫描器协议](https://github.com/moby/buildkit/blob/master/docs/attestations/sbom-protocol.md)
的镜像，从而选择不同的插件：

```console
$ docker buildx build --attest type=sbom,generator=<image> .
```

> [!TIP]
>
> 你也可以使用 Docker Scout 的 SBOM 生成器，参见
> [Docker Scout SBOMs](/manuals/scout/how-tos/view-create-sboms.md)。

## SBOM 证明示例

下面的 JSON 示例展示了一个 SBOM 证明的可能结构：

```json
{
  "_type": "https://in-toto.io/Statement/v0.1",
  "predicateType": "https://spdx.dev/Document",
  "subject": [
    {
      "name": "pkg:docker/<registry>/<image>@<tag/digest>?platform=<platform>",
      "digest": {
        "sha256": "e8275b2b76280af67e26f068e5d585eb905f8dfd2f1918b3229db98133cb4862"
      }
    }
  ],
  "predicate": {
    "SPDXID": "SPDXRef-DOCUMENT",
    "creationInfo": {
      "created": "2022-12-16T15:27:25.517047753Z",
      "creators": ["Organization: Anchore, Inc", "Tool: syft-v0.60.3"],
      "licenseListVersion": "3.18"
    },
    "dataLicense": "CC0-1.0",
    "documentNamespace": "https://anchore.com/syft/dir/run/src/core/sbom-cba61a72-fa95-4b60-b63f-03169eac25ca",
    "name": "/run/src/core/sbom",
    "packages": [
      {
        "SPDXID": "SPDXRef-b074348b8f56ea64",
        "downloadLocation": "NOASSERTION",
        "externalRefs": [
          {
            "referenceCategory": "SECURITY",
            "referenceLocator": "cpe:2.3:a:org:repo:\\(devel\\):*:*:*:*:*:*:*",
            "referenceType": "cpe23Type"
          },
          {
            "referenceCategory": "PACKAGE_MANAGER",
            "referenceLocator": "pkg:golang/github.com/org/repo@(devel)",
            "referenceType": "purl"
          }
        ],
        "filesAnalyzed": false,
        "licenseConcluded": "NONE",
        "licenseDeclared": "NONE",
        "name": "github.com/org/repo",
        "sourceInfo": "acquired package info from go module information: bin/server",
        "versionInfo": "(devel)"
      },
      {
        "SPDXID": "SPDXRef-1b96f57f8fed62d8",
        "checksums": [
          {
            "algorithm": "SHA256",
            "checksumValue": "0c13f1f3c1636491f716c2027c301f21f9dbed7c4a2185461ba94e3e58443408"
          }
        ],
        "downloadLocation": "NOASSERTION",
        "externalRefs": [
          {
            "referenceCategory": "SECURITY",
            "referenceLocator": "cpe:2.3:a:go-chi:chi\\/v5:v5.0.0:*:*:*:*:*:*:*",
            "referenceType": "cpe23Type"
          },
          {
            "referenceCategory": "SECURITY",
            "referenceLocator": "cpe:2.3:a:go_chi:chi\\/v5:v5.0.0:*:*:*:*:*:*:*",
            "referenceType": "cpe23Type"
          },
          {
            "referenceCategory": "SECURITY",
            "referenceLocator": "cpe:2.3:a:go:chi\\/v5:v5.0.0:*:*:*:*:*:*:*",
            "referenceType": "cpe23Type"
          },
          {
            "referenceCategory": "PACKAGE_MANAGER",
            "referenceLocator": "pkg:golang/github.com/go-chi/chi/v5@v5.0.0",
            "referenceType": "purl"
          }
        ],
        "filesAnalyzed": false,
        "licenseConcluded": "NONE",
        "licenseDeclared": "NONE",
        "name": "github.com/go-chi/chi/v5",
        "sourceInfo": "acquired package info from go module information: bin/server",
        "versionInfo": "v5.0.0"
      }
    ],
    "relationships": [
      {
        "relatedSpdxElement": "SPDXRef-1b96f57f8fed62d8",
        "relationshipType": "CONTAINS",
        "spdxElementId": "SPDXRef-043f7360d3c66bc31ba45388f16423aa58693289126421b71d884145f8837fe1"
      },
      {
        "relatedSpdxElement": "SPDXRef-b074348b8f56ea64",
        "relationshipType": "CONTAINS",
        "spdxElementId": "SPDXRef-043f7360d3c66bc31ba45388f16423aa58693289126421b71d884145f8837fe1"
      }
    ],
    "spdxVersion": "SPDX-2.2"
  }
}
```
