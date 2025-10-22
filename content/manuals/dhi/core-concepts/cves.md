---
title: 常见漏洞与暴露（CVE）
linktitle: CVE
description: 了解 CVE 的概念、Docker 加固镜像如何降低暴露风险，以及如何使用主流工具扫描镜像漏洞。
keywords: docker cve 扫描, grype 漏洞扫描器, trivy 镜像扫描, vex 证明, 安全容器镜像
---

## CVE 是什么？

CVE（Common Vulnerabilities and Exposures）是公开发布的软硬件安全缺陷。每个 CVE 都拥有唯一编号（如 CVE-2024-12345）和标准化描述，方便组织统一追踪与修复漏洞。

在 Docker 场景中，CVE 多出现在基础镜像或应用依赖里，轻则小 Bug，重则远程代码执行、权限提升等高危风险。

## 为什么要重视 CVE？

定期扫描并更新镜像、修复 CVE，是保持环境安全与合规的关键。忽视 CVE 可能导致：

- 未授权访问：攻击者利用漏洞直接拿到系统权限。  
- 数据泄露：敏感信息被窃取或曝光。  
- 业务中断：漏洞被用来制造拒绝服务或停机。  
- 合规违规：已知漏洞未修复，审计直接亮红灯。

## Docker 加固镜像（DHI）如何降低 CVE 风险

DHI 从设计阶段就把“安全”写进 DNA，带来三大优势：

- 攻击面极小：采用 distroless 思路，仅保留运行时必备组件，镜像体积最多缩小 95%，无用软件越少，漏洞自然越少。

- 修复速度极快：Docker 官方企业级 SLA 保障，高危与严重级别 CVE 通常在第一时间打补丁，无需你手动干预。

- 风险前置管理：镜像内置 CVE 与 VEX（Vulnerability Exploitability eXchange）源，团队可提前获知威胁并快速处置。

## 如何扫描镜像中的 CVE

再坚固的镜像也要定期“体检”。除了 Docker Desktop 与 CLI 集成的 Docker Scout，主流开源工具 Grype、Trivy 同样能帮你快速发现问题。

### Docker Scout

Docker Scout 已集成在 Docker Desktop 与 CLI，可提供漏洞概览、CVE 清单及一键修复指引。

#### 用 Scout 扫描 DHI

```console
$ docker scout cves <your-namespace>/dhi-<image>:<tag>
```

示例输出：

```plaintext
    v SBOM obtained from attestation, 101 packages found
    v Provenance obtained from attestation
    v VEX statements obtained from attestation
    v No vulnerable package detected
    ...
```

更多过滤与 JSON 输出格式，请参考 [Docker Scout CLI 手册](../../../reference/cli/docker/scout/_index.md)。

### Grype

[Grype](https://github.com/anchore/grype) 是 Anchore 开源的漏洞扫描器，同步 NVD 及各大发行版安全公告。

#### 用 Grype 扫描 DHI

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

### Trivy

[Trivy](https://github.com/aquasecurity/trivy) 是 Aqua 开源的多引擎扫描器，覆盖系统包与应用依赖。

#### 用 Trivy 扫描 DHI

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
│ <namespace>/dhi-<image>:<tag> (debian 12.11)                               │   debian   │       66        │    -    │
├──────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ opt/python-3.13.4/lib/python3.13/site-packages/pip-25.1.1.dist-info/METADATA │ python-pkg │        0        │    -    │
└──────────────────────────────────────────────────────────────────────────────┴────────────┴─────────────────┴─────────┘
```

## 利用 VEX 过滤“不可利用”CVE

DHI 自带已签名的 [VEX（漏洞可利用性交换）](./vex.md) 证明，明确标注哪些 CVE 在镜像实际运行环境中无法触发。

使用 Docker Scout 时，VEX 会自动生效，无需额外配置。

如需手动获取 VEX 文件供其他工具使用：

```console
$ docker scout vex get <your-namespace>/dhi-<image>:<tag> --output vex.json
```

> [!NOTE]
>
> `docker scout vex get` 需 Docker Scout CLI ≥ 1.18.3。

示例：

```console
$ docker scout vex get docs/dhi-python:3.13 --output vex.json
```

生成的 `vex.json` 可直接用于 Grype、Trivy 等支持 VEX 的工具，在扫描时过滤掉已知不可利用的 CVE：

```console
$ grype <your-namespace>/dhi-<image>:<tag> --vex vex.json
```