---
description: 如何将 Docker Scout 集成到 Microsoft Azure DevOps Pipelines
keywords: supply chain, security, ci, continuous integration, azure, devops
title: 将 Docker Scout 与 Microsoft Azure DevOps Pipelines 集成
linkTitle: Azure DevOps Pipelines
---

下面的示例在一个已连接 Azure DevOps 的代码仓库中运行，该仓库包含某个 Docker 镜像的定义与内容。每当向 main 分支提交变更时，流水线会构建镜像，并使用 Docker Scout 生成 CVE 报告。

首先，设置整体工作流与所有流水线步骤可用的变量。将以下内容添加到 _azure-pipelines.yml_ 文件：

```yaml
trigger:
  - main

resources:
  - repo: self

variables:
  tag: "$(Build.BuildId)"
  image: "vonwig/nodejs-service"
```

上述配置指定了应用所使用的容器镜像，并使用构建 ID 标记每次新的镜像构建。

继续在 YAML 文件中添加：

```yaml
stages:
  - stage: Build
    displayName: Build image
    jobs:
      - job: Build
        displayName: Build
        pool:
          vmImage: ubuntu-latest
        steps:
          - task: Docker@2
            displayName: Build an image
            inputs:
              command: build
              dockerfile: "$(Build.SourcesDirectory)/Dockerfile"
              repository: $(image)
              tags: |
                $(tag)
          - task: CmdLine@2
            displayName: Find CVEs on image
            inputs:
              script: |
                # Install the Docker Scout CLI
                curl -sSfL https://raw.githubusercontent.com/docker/scout-cli/main/install.sh | sh -s --
                # Login to Docker Hub required for Docker Scout CLI
                echo $(DOCKER_HUB_PAT) | docker login -u $(DOCKER_HUB_USER) --password-stdin
                # Get a CVE report for the built image and fail the pipeline when critical or high CVEs are detected
                docker scout cves $(image):$(tag) --exit-code --only-severity critical,high
```

上述流程会基于检出的 Dockerfile 构建并打标签，下载 Docker Scout CLI，随后对新标签执行 `cves` 命令以生成 CVE 报告，仅显示严重（critical）与高危（high）漏洞，并在命中时使流水线失败。
