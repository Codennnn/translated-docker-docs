---
title: 使用 VEX 创建例外（Exception）
description: 使用 VEX 文档为镜像中的漏洞创建例外条目。
keywords: Docker, vulnerability, exception, create, VEX
aliases:
  - /scout/guides/vex/
---

Vulnerability Exploitability eXchange（VEX）是一种标准格式，用于在软件包或产品的上下文中记录漏洞信息。
Docker Scout 支持使用 VEX 文档，为镜像中的漏洞创建[例外（exception）](/manuals/scout/explore/exceptions.md)。

> [!NOTE]
> 你也可以通过 Docker Scout Dashboard 或 Docker Desktop 创建例外。
> 这些图形界面提供了更友好的创建流程，便于针对多个镜像集中管理；
> 还可以一次性为多个镜像或整个组织创建例外。
> 详情参见《[使用 GUI 创建例外](/manuals/scout/how-tos/create-exceptions-gui.md)》。

## 前提条件

要使用 OpenVEX 文档创建例外，你需要：

- 最新版本的 Docker Desktop 或 Docker Scout CLI 插件
- [`vexctl`](https://github.com/openvex/vexctl) 命令行工具
- 已启用 [containerd 镜像存储](/manuals/desktop/features/containerd.md)
- 对存放镜像的仓库具有写入权限

## VEX 简介

VEX 标准由美国网络安全与基础设施安全局（CISA）的工作组制定。VEX 的核心是“可利用性评估”（exploitability assessment），
用于描述某个产品相对于特定 CVE 的状态。VEX 中可能的漏洞状态包括：

- Not affected：不受影响，无需采取修复措施。
- Affected：受影响，建议采取行动修复或缓解该漏洞。
- Fixed：已修复，这些产品版本包含修复。
- Under investigation：调查中，暂不确定这些产品版本是否受影响，将在后续版本中更新。

VEX 有多种实现与格式。Docker Scout 支持 [OpenVEX](https://github.com/openvex/spec) 实现。
无论具体实现如何，其核心目标都是提供一个描述漏洞影响的框架。VEX 的关键组成部分（与实现无关）包括：

VEX 文档
: 一类安全通告文档，用于存放 VEX 声明。
  文档的具体格式取决于所采用的实现。

VEX 声明
: 描述产品中的漏洞状态、是否可被利用，以及是否存在修复或缓解方式。

理由与影响（Justification & Impact）
: 根据漏洞状态，声明可包含理由或影响说明，解释产品为何受影响或不受影响。

行动声明（Action statements）
: 描述如何修复或缓解该漏洞。

## `vexctl` 示例

下面的示例命令会创建一份 VEX 文档，其语义为：

- 该 VEX 文档描述的软件产品是 Docker 镜像 `example/app:v1`
- 该镜像包含 npm 包 `express@4.17.1`
- 该 npm 包受到已知漏洞 `CVE-2022-24999` 的影响
- 镜像不受该 CVE 影响，因为运行该镜像的容器不会执行存在漏洞的代码路径

```console
$ vexctl create \
  --author="author@example.com" \
  --product="pkg:docker/example/app@v1" \
  --subcomponents="pkg:npm/express@4.17.1" \
  --vuln="CVE-2022-24999" \
  --status="not_affected" \
  --justification="vulnerable_code_not_in_execute_path" \
  --file="CVE-2022-24999.vex.json"
```

以下是该示例中各选项的说明：

`--author`
: VEX 文档作者的邮箱。

`--product`
: Docker 镜像的 Package URL（PURL）。PURL 是标准化的镜像标识符，定义见 PURL
  [规范](https://github.com/package-url/purl-spec/blob/master/PURL-TYPES.rst#docker)。

  Docker 镜像的 PURL 以 `pkg:docker` 类型前缀开头，随后是镜像仓库名与版本（镜像标签或 SHA256 摘要）。
  与 `example/app:v1` 这样的标签写法不同，PURL 使用 `@` 将仓库名与版本分隔。

`--subcomponents`
: 镜像内存在漏洞的软件包的 PURL。在本例中，漏洞存在于 npm 包中，因此 `--subcomponents` 的 PURL
  即该 npm 包的名称与版本标识（`pkg:npm/express@4.17.1`）。

  若同一漏洞存在于多个软件包中，可在一次 `create` 命令中多次指定 `--subcomponents`。

  也可以省略 `--subcomponents`，此时 VEX 声明适用于整个镜像。

`--vuln`
: 该 VEX 声明所涉及的 CVE 编号。

`--status`
: 漏洞的状态标签，用于描述软件（`--product`）与 CVE（`--vuln`）之间的关系。
  在 OpenVEX 中可选取值包括：

  - `not_affected`
  - `affected`
  - `fixed`
  - `under_investigation`

  在本例中，VEX 声明断言该 Docker 镜像对该漏洞 `not_affected`（不受影响）。
  只有 `not_affected` 状态会触发对该 CVE 的抑制（即从分析结果中过滤）。
  其他状态有助于文档记录，但不会用于创建例外。
  关于各状态标签的完整定义，参见 OpenVEX 规范中的
  [Status Labels](https://github.com/openvex/spec/blob/main/OPENVEX-SPEC.md#status-labels)。

`--justification`
: 对 `not_affected` 状态给出理由，说明产品为何不受该漏洞影响。
  本例给出的理由为 `vulnerable_code_not_in_execute_path`，表示产品的实际使用路径不会执行到存在漏洞的代码。

  在 OpenVEX 中，理由（justification）可取如下五个值：

  - `component_not_present`
  - `vulnerable_code_not_present`
  - `vulnerable_code_not_in_execute_path`
  - `vulnerable_code_cannot_be_controlled_by_adversary`
  - `inline_mitigations_already_exist`

  这些取值的定义与使用场景，参见 OpenVEX 规范中的
  [Status Justifications](https://github.com/openvex/spec/blob/main/OPENVEX-SPEC.md#status-justifications)。

`--file`
: 输出的 VEX 文档文件名。

## 示例 JSON 文档

以下是该命令生成的 OpenVEX JSON：

```json
{
  "@context": "https://openvex.dev/ns/v0.2.0",
  "@id": "https://openvex.dev/docs/public/vex-749f79b50f5f2f0f07747c2de9f1239b37c2bda663579f87a35e5f0fdfc13de5",
  "author": "author@example.com",
  "timestamp": "2024-05-27T13:20:22.395824+02:00",
  "version": 1,
  "statements": [
    {
      "vulnerability": {
        "name": "CVE-2022-24999"
      },
      "timestamp": "2024-05-27T13:20:22.395829+02:00",
      "products": [
        {
          "@id": "pkg:docker/example/app@v1",
          "subcomponents": [
            {
              "@id": "pkg:npm/express@4.17.1"
            }
          ]
        }
      ],
      "status": "not_affected",
      "justification": "vulnerable_code_not_in_execute_path"
    }
  ]
}
```

理解 VEX 文档的规范结构可能略显繁琐。[OpenVEX 规范](https://github.com/openvex/spec)
详细描述了文档与声明的格式与属性。若需完整细节，请参阅该规范，了解可用字段以及如何创建格式正确的 OpenVEX 文档。

关于 `vexctl` CLI 工具的参数、语法及安装方法，请参见其
[`vexctl` GitHub 仓库](https://github.com/openvex/vexctl)。

## 验证 VEX 文档

要测试你创建的 VEX 文档是否规范且能产生预期结果，可在本地镜像分析中使用 CLI：
通过 `docker scout cves` 搭配 `--vex-location` 参数来应用这些 VEX 文档。

如下命令会在本地执行镜像分析，并纳入 `--vex-location` 指定位置中的所有 VEX 文档。
本例中，CLI 会在当前工作目录下查找 VEX 文档：

```console
$ docker scout cves <IMAGE> --vex-location .
```

`docker scout cves` 的输出会将 `--vex-location` 下发现的 VEX 声明一并纳入结果。
例如，被标记为 `not_affected` 的 CVE 会从结果中过滤。如果输出似乎未采用 VEX 声明，
则说明这些 VEX 文档可能在某些方面无效。

排查要点包括：

- Docker 镜像的 PURL 必须以 `pkg:docker/` 开头，后接镜像名称。
- 在 Docker 镜像 PURL 中，镜像名称与版本通过 `@` 分隔。
  例如镜像 `example/myapp:1.0` 的 PURL 为：`pkg:docker/example/myapp@1.0`。
- 记得指定 `author`（OpenVEX 中为必填字段）。
- [OpenVEX 规范](https://github.com/openvex/spec) 说明了何时以及如何使用 `justification`、
  `impact_statement` 等字段。错误使用会导致文档无效。请确保你的 VEX 文档遵循该规范。

## 将 VEX 文档附加到镜像

创建好 VEX 文档后，你可以通过以下方式将其附加到镜像：

- 将文档作为[证明（attestation）](#attestation)进行附加
- 将文档嵌入[镜像文件系统](#image-filesystem)

一旦将 VEX 文档添加到镜像后，就无法移除。
若以证明（attestation）的方式附加，可创建新的 VEX 文档并再次附加到镜像，从而覆盖之前的文档（但不会移除证明本身）。
如果 VEX 文档被嵌入到镜像文件系统中，则需要重新构建镜像才能更换该文档。

### 证明（Attestation） {#attestation}

要以证明形式附加 VEX 文档，可使用 `docker scout attestation add` 命令。
在使用 VEX 为镜像附加例外时，推荐采用证明的方式。

你可以为已推送到仓库的镜像附加证明，无需重新构建或推送。
此外，将例外作为证明附加到镜像后，使用者可以直接从仓库查看该镜像的例外信息。

为镜像附加证明：

1. Build the image and push it to a registry.

   ```console
   $ docker build --provenance=true --sbom=true --tag <IMAGE> --push .
   ```

2. 将例外作为证明附加到镜像。

   ```console
   $ docker scout attestation add \
     --file <cve-id>.vex.json \
     --predicate-type https://openvex.dev/ns/v0.2.0 \
     <IMAGE>
   ```

   命令参数说明：

   - `--file`：VEX 文档的位置与文件名
   - `--predicate-type`：OpenVEX 的 in-toto `predicateType`

### 镜像文件系统 {#image-filesystem}

如果在构建镜像前就已经确定了例外，直接将 VEX 文档嵌入镜像文件系统也是一种可行方式。
做法也相对简单：在 Dockerfile 中 `COPY` 该 VEX 文档到镜像即可。

该方式的不足在于后续无法变更或更新例外。镜像层是不可变的，写入镜像文件系统的内容将永久保留。
若希望具备更好的灵活性，建议以[证明](#attestation)的方式附加文档。

> [!NOTE]
> 对于带有证明（attestations）的镜像，嵌入在镜像文件系统中的 VEX 文档不会被采纳。
> 只要镜像存在任意证明，Docker Scout 仅会在证明中查找例外，而不会读取镜像文件系统中的文档。
>
> 如果你想使用嵌入在镜像文件系统中的 VEX 文档，必须先从镜像中移除证明。
> 注意，某些情况下镜像会自动添加来源（provenance）证明。为确保不添加任何证明，
> 在构建镜像时可显式设置 `--provenance=false` 与 `--sbom=false`。

要将 VEX 文档嵌入镜像文件系统，请在构建阶段 `COPY` 该文件。
以下示例演示如何把构建上下文中 `.vex/` 下的所有 VEX 文档复制到镜像内的 `/var/lib/db`：

```dockerfile
# syntax=docker/dockerfile:1

FROM alpine
COPY .vex/* /var/lib/db/
```

VEX 文档文件名需符合 `*.vex.json` 的通配规则。文档可存放在镜像文件系统的任意位置。

请注意，拷贝的文件必须出现在最终镜像的文件系统中；
对于多阶段构建，这些文档需要在最终阶段保留下来。

