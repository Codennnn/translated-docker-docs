---
title: Docker Scout 快速开始
linkTitle: 快速开始
weight: 20
keywords: scout, supply chain, vulnerabilities, packages, cves, scan, analysis, analyze
description: 了解如何使用 Docker Scout 分析镜像并修复漏洞
---

Docker Scout 会分析镜像内容，并生成包含已检测到的软件包与漏洞的详细报告。
它还会给出针对这些问题的修复建议。

本指南以一个存在漏洞的容器镜像为例，演示如何使用 Docker Scout 识别并修复漏洞，
对比不同版本镜像的差异，并将结果分享给团队。

## 步骤 1：环境准备

[示例项目](https://github.com/docker/scout-demo-service) 包含一个存在漏洞的 Node.js 应用，
你可以用它来跟随演练。

1. 克隆仓库：

   ```console
   $ git clone https://github.com/docker/scout-demo-service.git
   ```

2. 进入项目目录：

   ```console
   $ cd scout-demo-service
   ```

3. 确认你已登录 Docker 账号：可以运行 `docker login`，或在 Docker Desktop 中登录。

4. 构建镜像并推送到 `<ORG_NAME>/scout-demo:v1`，
   其中 `<ORG_NAME>` 是你的 Docker Hub 命名空间。

   ```console
   $ docker build --push -t <ORG_NAME>/scout-demo:v1 .
   ```

## 步骤 2：启用 Docker Scout

Docker Scout 默认会分析本地镜像。若要分析远程仓库中的镜像，需要先启用。
你可以在 Docker Hub、Docker Scout 仪表板和 CLI 中启用。
详见[概览](/scout)。

1. 通过 `docker login` 登录，或在 Docker Desktop 中选择 **Sign in**。

2. 使用 `docker scout enroll` 将你的组织接入 Docker Scout：

   ```console
   $ docker scout enroll <ORG_NAME>
   ```

3. 使用 `docker scout repo enable` 为目标镜像仓库启用 Docker Scout：

   ```console
   $ docker scout repo enable --org <ORG_NAME> <ORG_NAME>/scout-demo
   ```

## 步骤 3：分析镜像漏洞

构建完成后，使用 `docker scout` 查看 Docker Scout 检测到的漏洞。

本示例应用使用了存在漏洞的 Express 版本。
下面的命令会显示刚刚构建的镜像中影响 Express 的所有 CVE：

```console
$ docker scout cves --only-package express
```

默认情况下，Docker Scout 会分析最近构建的镜像，因此此处无需显式指定镜像名。

在[`CLI 参考`](/reference/cli/docker/scout/cves)中了解更多 `docker scout cves` 命令的信息。

## 步骤 4：修复应用漏洞

通过 Docker Scout 分析发现，**express** 依赖版本过旧，存在一个高危漏洞 CVE-2022-24999。

`express` 4.17.3 版本修复了该漏洞。将 `package.json` 中的版本更新为：

   ```diff
      "dependencies": {
   -    "express": "4.17.1"
   +    "express": "4.17.3"
      }
   ```
   
使用新标签重新构建镜像并推送到 Docker Hub：

   ```console
   $ docker build --push -t <ORG_NAME>/scout-demo:v2 .
   ```

再次运行 `docker scout`，确认高危 CVE-2022-24999 已消失：

```console
$ docker scout cves --only-package express
    ✓ Provenance obtained from attestation
    ✓ Image stored for indexing
    ✓ Indexed 79 packages
    ✓ No vulnerable package detected


  ## Overview

                      │                  Analyzed Image                   
  ────────────────────┼───────────────────────────────────────────────────
    Target            │  mobywhale/scout-demo:v2                   
      digest          │  ef68417b2866                                     
      platform        │ linux/arm64                                       
      provenance      │ https://github.com/docker/scout-demo-service.git  
                      │  7c3a06793fc8f97961b4a40c73e0f7ed85501857         
      vulnerabilities │    0C     0H     0M     0L                        
      size            │ 19 MB                                             
      packages        │ 1                                                 


  ## Packages and Vulnerabilities

  No vulnerable packages detected

```

## 步骤 5：评估策略合规性

仅基于特定软件包检查漏洞很有价值，
但这并不是改进供应链治理的最有效方式。

Docker Scout 还支持“策略评估”，
这是一种更高层的能力，用于发现与修复镜像中的问题。
策略是一组可自定义的规则，帮助组织跟踪镜像是否满足其供应链要求。

由于策略规则通常因组织而异，
你需要指定要对照的组织策略。
使用 `docker scout config` 配置你的 Docker 组织：

```console
$ docker scout config organization <ORG_NAME>
    ✓ Successfully set organization to <ORG_NAME>
```

现在可以运行 `quickview` 命令，查看刚刚构建的镜像在默认策略下的合规概览。
输出类似如下：

```console
$ docker scout quickview

...
Policy status  FAILED  (2/6 policies met, 2 missing data)

  Status │                  Policy                      │           Results
─────────┼──────────────────────────────────────────────┼──────────────────────────────
  ✓      │ No copyleft licenses                         │    0 packages
  !      │ Default non-root user                        │
  !      │ No fixable critical or high vulnerabilities  │    2C    16H     0M     0L
  ✓      │ No high-profile vulnerabilities              │    0C     0H     0M     0L
  ?      │ No outdated base images                      │    No data
  ?      │ Supply chain attestations                    │    No data
```

状态列中的感叹号表示不合规；
问号表示缺少完成评估所需的元数据；
对勾表示符合要求。

## 步骤 6：提升合规性

`quickview` 的输出显示仍有改进空间。
部分策略因缺少溯源与 SBOM 声明而无法完成评估（`No data`），
另一些评估也未通过。

策略评估不只是检查漏洞。
例如 `Default non-root user` 策略，
它通过确保镜像默认不以 `root` 超级用户运行，来提升运行时安全性。

为修复该策略违规，在 Dockerfile 中添加 `USER` 指令，指定一个非 root 用户：

```diff
  CMD ["node","/app/app.js"]
  EXPOSE 3000
+ USER appuser
```

此外，为获取更完整的策略评估结果，
应为镜像附加 SBOM 与溯源声明（attestations）。
Docker Scout 会利用溯源声明了解镜像的构建方式，从而给出更高质量的评估结果。

在为镜像生成并附加声明之前，
需要启用[containerd 镜像存储](/manuals/desktop/features/containerd.md)
（或使用 `docker-container` 驱动创建自定义 builder）。
传统镜像存储不支持清单列表（manifest list），
而溯源声明正是通过清单列表附加到镜像上的。

打开 Docker Desktop 的 **Settings**。
在 **General** 中勾选 **Use containerd for pulling and storing images**，然后点击 **Apply**。
请注意，切换镜像存储会暂时隐藏另一种存储中的镜像与容器，直到你切回去。

启用 containerd 镜像存储后，使用新的 `v3` 标签重新构建镜像。
这次加入 `--provenance=true` 与 `--sbom=true` 参数：

```console
$ docker build --provenance=true --sbom=true --push -t <ORG_NAME>/scout-demo:v3 .
```

## 步骤 7：在仪表板查看

在推送带有声明的更新镜像后，你可以在 Docker Scout 仪表板中从另一种视角查看结果。

1. 打开 [Docker Scout 仪表板](https://scout.docker.com/)。
2. 使用 Docker 账号登录。
3. 在左侧导航中选择 **Images**。

该页面会列出已启用 Scout 的仓库。

在列表中选择要查看的镜像（点击行内非链接区域），打开 **Image details** 侧边栏。

侧边栏会显示该仓库最近一次推送标签的合规概览。

> [!NOTE]
>
> 若策略结果尚未出现，请尝试刷新页面。
> 若这是你首次使用 Docker Scout 仪表板，结果可能需要数分钟才能显示。

返回镜像列表，在 **Most recent image** 列中选择镜像版本。
随后在页面右上角选择 **Update base image** 按钮以查看相关策略。

该策略用于检查你使用的基础镜像是否为最新。
当前状态为不合规，原因是示例镜像的基础镜像 `alpine` 版本过旧。

关闭 **Recommended fixes for base image** 弹窗。在策略列表中，点击对应策略右侧的 **View fixes** 查看违规详情与修复建议。

此处的建议操作是启用
[Docker Scout 的 GitHub 集成](./integrations/source-code-management/github.md)，
以便自动保持基础镜像的最新状态。

> [!TIP]
>
> 本指南所用的演示应用无法直接启用该集成。
> 你可以将代码推送到你自己的 GitHub 仓库中，再尝试此集成。

## 总结

本快速开始介绍了 Docker Scout 在软件供应链管理中的部分能力：

- 如何为你的仓库启用 Docker Scout
- 分析镜像漏洞
- 策略与合规
- 修复漏洞并提升合规性

## 下一步

还有更多可探索的内容：第三方集成、策略自定义，以及运行时环境的实时监控等。

可继续查看以下章节：

- [镜像分析](/manuals/scout/explore/analysis.md)
- [数据来源](/scout/advisory-db-sources)
- [Docker Scout 仪表板](/scout/dashboard)
- [集成](./integrations/_index.md)
- [策略评估](./policy/_index.md)
