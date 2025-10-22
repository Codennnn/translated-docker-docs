---
title: 扫描 Docker 加固镜像
linktitle: 扫描镜像
description: 了解如何使用 Docker Scout、Grype 或 Trivy 扫描 Docker 加固镜像中的已知漏洞。
keywords: 扫描容器镜像, docker scout cves, grype 扫描器, trivy 容器扫描器, vex 证明
weight: 45
---

{{< summary-bar feature_name="Docker Hardened Images" >}}

Docker 加固镜像（DHIs）默认设计为安全镜像，但如同任何容器镜像一样，定期扫描仍是漏洞管理流程中的重要环节。

您可以使用与标准镜像相同的工具来扫描 DHIs，例如 Docker Scout、Grype 和 Trivy。DHIs 遵循相同的格式和标准，确保与您的安全工具链兼容。扫描前，必须先将镜像镜像化到您在 Docker Hub 的组织中。

> [!NOTE]
>
> [Docker Scout](/manuals/scout/_index.md) 会自动为 Docker Hub 上所有已镜像的 Docker 加固镜像仓库启用，无需额外费用。您可以直接在 Docker Hub UI 中您组织的仓库下查看扫描结果。

## Docker Scout

Docker Scout 已集成到 Docker Desktop 和 Docker CLI 中，可提供漏洞洞察、CVE 摘要以及直接的修复指导链接。

### 使用 Docker Scout 扫描 DHI

要使用 Docker Scout 扫描 Docker 加固镜像，请运行以下命令：

```console
$ docker scout cves <your-namespace>/dhi-<image>:<tag> --platform <platform>
```

示例输出：

```plaintext
    v SBOM obtained from attestation, 101 packages found
    v Provenance obtained from attestation
    v VEX statements obtained from attestation
    v No vulnerable package detected
    ...
```

如需更详细的筛选和 JSON 输出，请参阅 [Docker Scout CLI 参考](../../../reference/cli/docker/scout/_index.md)。

### 在 CI/CD 中自动化 DHI 扫描

将 Docker Scout 集成到 CI/CD 流水线中，可在构建过程中自动验证基于 Docker 加固镜像构建的镜像是否不含已知漏洞。这种主动方式可确保镜像在整个开发生命周期中持续保持安全完整性。

#### GitHub Actions 工作流示例

以下是一个示例 GitHub Actions 工作流，用于构建镜像并使用 Docker Scout 进行扫描：

```yaml {collapse="true"}
name: DHI 漏洞扫描

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ "**" ]

env:
  REGISTRY: docker.io
  IMAGE_NAME: ${{ github.repository }}
  SHA: ${{ github.event.pull_request.head.sha || github.event.after }}

jobs:
  scan:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
      pull-requests: write

    steps:
      - name: 检出仓库
        uses: actions/checkout@v3

      - name: 设置 Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: 登录 Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: 构建 Docker 镜像
        run: |
          docker build -t ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ env.SHA }} .

      - name: 运行 Docker Scout CVE 扫描
        uses: docker/scout-action@v1
        with:
          command: cves
          image: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ env.SHA }}
          only-severities: critical,high
          exit-code: true
```

`exit-code: true` 参数可确保在检测到任何严重或高危漏洞时工作流失败，防止部署不安全的镜像。

有关在 CI 中使用 Docker Scout 的更多详情，请参阅[将 Docker Scout 与其他系统集成](/manuals/scout/integrations/_index.md)。

## Grype

[Grype](https://github.com/anchore/grype) 是一款开源扫描器，可针对 NVD 和发行版公告等漏洞数据库检查容器镜像。

### 使用 Grype 扫描 DHI

安装 Grype 后，您可以通过拉取镜像并运行扫描命令来扫描 Docker 加固镜像：

```console
$ docker pull <your-namespace>/dhi-<image>:<tag>
$ grype <your-namespace>/dhi-<image>:<tag>
```

示例输出：

```plaintext
NAME               INSTALLED              FIXED-IN     TYPE  VULNERABILITY     SEVERITY    EPSS%  RISK
libperl5.36        5.36.0-7+deb12u2       (won't fix)  deb   CVE-2023-31484    High        79.45    1.1
perl               5.36.0-7+deb12u2       (won't fix)  deb   CVE-2023-31484    High        79.45    1.1
perl-base          5.36.0-7+deb12u2       (won't fix)  deb   CVE-2023-31484    High        79.45    1.1
...
```

扫描时应包含 `--vex` 标志以应用 VEX 声明，过滤掉已知的不可利用 CVE。更多信息请参阅 [VEX 部分](#使用-vex-过滤已知不可利用的-cves)。

## Trivy

[Trivy](https://github.com/aquasecurity/trivy) 是一款针对容器和其他制品的开源漏洞扫描器，可检测操作系统包和应用程序依赖中的漏洞。

### 使用 Trivy 扫描 DHI

安装 Trivy 后，您可以通过拉取镜像并运行扫描命令来扫描 Docker 加固镜像：

```console
$ docker pull <your-namespace>/dhi-<image>:<tag>
$ trivy image <your-namespace>/dhi-<image>:<tag>
```

示例输出：

```plaintext
Report Summary

┌──────────────────────────────────────────────────────────────────────────────┬────────────┬─────────────────┬─────────┐
│                                    Target                                    │    Type    │ Vulnerabilities │ Secrets │
├──────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ <namespace>/dhi-<image>:<tag> (debian 12.11)                                 │   debian   │       66        │    -    │
├──────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ opt/python-3.13.4/lib/python3.13/site-packages/pip-25.1.1.dist-info/METADATA │ python-pkg │        0        │    -    │
└──────────────────────────────────────────────────────────────────────────────┴────────────┴─────────────────┴─────────┘
```

扫描时应包含 `--vex` 标志以应用 VEX 声明，过滤掉已知的不可利用 CVE。更多信息请参阅 [VEX 部分](#使用-vex-过滤已知不可利用的-cves)。

## 使用 VEX 过滤已知不可利用的 CVE

Docker 加固镜像包含已签名的 VEX（Vulnerability Exploitability eXchange，漏洞可利用性交换）证明，用于识别与镜像运行时行为无关的漏洞。

使用 Docker Scout 时，这些 VEX 声明会自动应用，无需手动配置。

如需为支持的工具手动创建 VEX 证明的 JSON 文件：

```console
$ docker scout vex get <your-namespace>/dhi-<image>:<tag> --output vex.json
```

> [!NOTE]
>
> `docker scout vex get` 命令需要 [Docker Scout CLI](https://github.com/docker/scout-cli/) 1.18.3 或更高版本。

例如：

```console
$ docker scout vex get docs/dhi-python:3.13 --output vex.json
```

这会创建一个包含指定镜像 VEX 声明的 `vex.json` 文件。然后您可以将此文件与支持 VEX 的工具一起使用，以过滤掉已知的不可利用 CVE。

例如，对于 Grype 和 Trivy，您可以在扫描时使用 `--vex` 标志来应用 VEX 声明：

```console
$ grype <your-namespace>/dhi-<image>:<tag> --vex vex.json
```