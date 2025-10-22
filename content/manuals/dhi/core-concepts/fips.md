---
title: FIPS 140 合规支持
description: 了解 Docker 加固镜像如何通过经过验证的加密模块支持 FIPS 140，帮助组织满足合规要求。
keywords: docker fips, fips 140 镜像, fips docker 镜像, docker 合规, 安全容器镜像
---

## FIPS 140 是什么？

[FIPS 140](https://csrc.nist.gov/publications/detail/fips/140/3/final) 是美国政府制定的密码模块安全标准，用于保护敏感信息，在政府、医疗、金融等受监管行业被广泛采纳。  
标准由 [NIST 密码模块验证计划（CMVP）](https://csrc.nist.gov/projects/cryptographic-module-validation-program) 负责认证，确保密码算法与实现达到严格的安全基准。

## 为何必须关注 FIPS 合规

在需要保护敏感数据的受监管场景（政府、医疗、金融、国防）中，FIPS 140 合规是硬性或强监管要求。采用通过验证的密码模块可：

- **满足 FedRAMP 等联邦/行业法规**——强制要求使用 FIPS 140 验证的密码学组件；
- **提升审计就绪度**——提供可验证的证据链，证明系统采用标准化、安全的密码实践；
- **降低安全风险**——自动屏蔽 MD5 等不安全算法，确保跨环境行为一致。

## Docker 加固镜像如何助力 FIPS 合规

Docker 加固镜像（DHI）提供专门的 **FIPS 变体**，内置已通过 FIPS 140 验证的密码模块，帮助组织“开箱即用”地满足合规要求：

- 镜像在构建阶段即集成已验证的密码模块，无需自行编译或补丁；
- Docker 官方维护并持续更新，确保验证状态不过期；
- 提供**签名合规声明（attestation）**，可用于内部审计与合规报告。

> [!NOTE]
> 使用 FIPS 镜像只是“合规拼图”的一块，最终合规仍需结合整体系统架构与使用方式。

## 快速识别支持 FIPS 的镜像

在 Docker 加固镜像目录中：

1. 打开 [镜像浏览页](../how-to/explore.md)，勾选 **FIPS** 过滤器；
2. 或在单条镜像详情页查看 **FIPS 合规** 标签。

符合要求的镜像标签以 `-fips` 结尾，例如 `3.13-fips`。

## 查看 FIPS 合规声明

使用 Docker Scout CLI 一键获取镜像内嵌的 FIPS 声明，验证所含密码模块：

```bash
docker scout attest get \
  --predicate-type https://docker.com/dhi/fips/v0.1 \
  --predicate \
  <命名空间>/dhi-<镜像>:<标签>
```

示例：

```bash
docker scout attest get \
  --predicate-type https://docker.com/dhi/fips/v0.1 \
  --predicate \
  docs/dhi-python:3.13-fips
```

返回的 JSON 片段示例：

```json
[
  {
    "certification": "CMVP #4985",
    "certificationUrl": "https://csrc.nist.gov/projects/cryptographic-module-validation-program/certificate/4985",
    "name": "OpenSSL FIPS Provider",
    "package": "pkg:dhi/openssl-provider-fips@3.1.2",
    "standard": "FIPS 140-3",
    "status": "active",
    "sunsetDate": "2030-03-10",
    "version": "3.1.2"
  }
]
```

每条记录均包含认证编号、标准版本、有效期与官方证书链接，方便审计时一键溯源。