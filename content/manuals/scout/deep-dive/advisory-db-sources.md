---
description: 详解 Docker Scout 分析背后的通告数据库与 CVE-到-软件包匹配服务。
keywords: scout, scanning, analysis, vulnerabilities, Hub, supply chain, security, packages, repositories, ecosystem
title: 通告数据库来源与匹配服务
aliases:
  /scout/advisory-db-sources/
---

可靠的信息来源是 Docker Scout 能够对你的软件制品给出准确、相关评估的关键。
鉴于业界信息来源与方法论多样，漏洞评估结果存在差异在所难免。
本文说明 Docker Scout 的通告数据库以及其 CVE-到-软件包的匹配方法如何应对这些差异。

## 通告数据库来源

Docker Scout 汇聚来自多方的数据源以构建漏洞数据库。
这些数据会持续更新，以确保你的安全态势能实时反映为最新可用的信息。

Docker Scout 使用以下软件包仓库与安全追踪源：

<!-- vale off -->

- [AlmaLinux Security Advisory](https://errata.almalinux.org/)
- [Alpine secdb](https://secdb.alpinelinux.org/)
- [Amazon Linux Security Center](https://alas.aws.amazon.com/)
- [Bitnami Vulnerability Database](https://github.com/bitnami/vulndb)
- [CISA Known Exploited Vulnerability Catalog](https://www.cisa.gov/known-exploited-vulnerabilities-catalog)
- [CISA Vulnrichment](https://github.com/cisagov/vulnrichment)
- [Chainguard Security Feed](https://packages.cgr.dev/chainguard/osv/all.json)
- [Debian Security Bug Tracker](https://security-tracker.debian.org/tracker/)
- [Exploit Prediction Scoring System (EPSS)](https://api.first.org/epss/)
- [GitHub Advisory Database](https://github.com/advisories/)
- [GitLab Advisory Database](https://gitlab.com/gitlab-org/advisories-community/)
- [Golang VulnDB](https://github.com/golang/vulndb)
- [National Vulnerability Database](https://nvd.nist.gov/)
- [Oracle Linux Security](https://linux.oracle.com/security/)
- [Photon OS 3.0 Security Advisories](https://github.com/vmware/photon/wiki/Security-Updates-3)
- [Python Packaging Advisory Database](https://github.com/pypa/advisory-database)
- [RedHat Security Data](https://www.redhat.com/security/data/metrics/)
- [Rocky Linux Security Advisory](https://errata.rockylinux.org/)
- [RustSec Advisory Database](https://github.com/rustsec/advisory-db)
- [SUSE Security CVRF](http://ftp.suse.com/pub/projects/security/cvrf/)
- [Ubuntu CVE Tracker](https://people.canonical.com/~ubuntu-security/cve/)
- [Wolfi Security Feed](https://packages.wolfi.dev/os/security.json)
- [inTheWild, a community-driven open database of vulnerability exploitation](https://github.com/gmatuz/inthewilddb)

<!-- vale on -->

当你为 Docker 组织启用 Docker Scout 时，平台会为你预配一个新的数据库实例。
该数据库用于保存镜像的 SBOM 及其他元数据。
当安全通告发布了某个漏洞的新增信息时，平台会将这些信息与你的 SBOM 进行交叉比对，
从而判断其对你的影响。

关于镜像分析工作机制的更多细节，请参阅[镜像分析](/manuals/scout/explore/analysis.md)。

## 严重性与评分优先级

在确定 CVE 的严重性与评分时，Docker Scout 遵循两条主要原则：

   - 数据来源优先级
   - CVSS 版本偏好

就数据来源优先级，Docker Scout 遵循如下顺序：

  1. 供应商通告：优先使用与软件包与版本相匹配的官方来源数据。例如，Debian 包优先使用 Debian 的数据。

  2. NIST 评分数据：若供应商未提供该 CVE 的评分，回退至 NIST 的评分数据。

关于 CVSS 版本，一旦确定了数据来源，在同时存在 v4 与 v3 时，优先使用更现代、更精确的 v4 评分模型。

## 漏洞匹配

传统工具通常依赖较为宽泛的 [CPE（Common Product Enumeration）](https://en.wikipedia.org/wiki/Common_Platform_Enumeration) 匹配，
这容易导致较多的误报。

Docker Scout 采用 [PURL（Package URL）](https://github.com/package-url/purl-spec) 将软件包与 CVE 进行匹配，
以更精确地识别漏洞。
PURL 大幅降低误报概率，聚焦于真正受影响的软件包。

## 支持的软件包生态

Docker Scout 当前支持以下软件包生态：

- .NET
- GitHub packages
- Go
- Java
- JavaScript
- PHP
- Python
- RPM
- Ruby
- `alpm`（Arch Linux）
- `apk`（Alpine Linux）
- `deb`（Debian 及其衍生版）
