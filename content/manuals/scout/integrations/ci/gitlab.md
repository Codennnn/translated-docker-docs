---
description: 如何将 Docker Scout 集成到 GitLab CI
keywords: supply chain, security, ci, continuous integration, gitlab
title: 将 Docker Scout 与 GitLab CI/CD 集成
linkTitle: GitLab CI/CD
---

下面的示例在包含 Docker 镜像定义与内容的仓库中运行 GitLab CI。当有提交时，流水线会构建镜像。如果提交到默认分支，则使用 Docker Scout 生成 CVE 报告；如果提交到其他分支，则使用 Docker Scout 将新版本与当前已发布版本进行对比。

## 步骤

首先，设置整体工作流。虽然很多内容与 Docker Scout 无关，但它们是创建可对比镜像所必需的。

在仓库根目录的 `.gitlab-ci.yml` 文件中添加以下内容：

```yaml
docker-build:
  image: docker:latest
  stage: build
  services:
    - docker:dind
  before_script:
    - docker login -u "$CI_REGISTRY_USER" -p "$CI_REGISTRY_PASSWORD" $CI_REGISTRY

    # Install curl and the Docker Scout CLI
    - |
      apk add --update curl
      curl -sSfL https://raw.githubusercontent.com/docker/scout-cli/main/install.sh | sh -s -- 
      apk del curl 
      rm -rf /var/cache/apk/*
    # Login to Docker Hub required for Docker Scout CLI
    - echo "$DOCKER_HUB_PAT" | docker login -u "$DOCKER_HUB_USER" --password-stdin
```

上述配置使用 Docker-in-Docker 模式构建 Docker 镜像，即在容器内运行 Docker。

然后下载 `curl` 与 Docker Scout CLI 插件，并使用仓库设置中定义的环境变量登录 Docker 仓库。

继续在 YAML 文件中添加：

```yaml
script:
  - |
    if [[ "$CI_COMMIT_BRANCH" == "$CI_DEFAULT_BRANCH" ]]; then
      tag=""
      echo "Running on default branch '$CI_DEFAULT_BRANCH': tag = 'latest'"
    else
      tag=":$CI_COMMIT_REF_SLUG"
      echo "Running on branch '$CI_COMMIT_BRANCH': tag = $tag"
    fi
  - docker build --pull -t "$CI_REGISTRY_IMAGE${tag}" .
  - |
    if [[ "$CI_COMMIT_BRANCH" == "$CI_DEFAULT_BRANCH" ]]; then
      # Get a CVE report for the built image and fail the pipeline when critical or high CVEs are detected
      docker scout cves "$CI_REGISTRY_IMAGE${tag}" --exit-code --only-severity critical,high    
    else
      # Compare image from branch with latest image from the default branch and fail if new critical or high CVEs are detected
      docker scout compare "$CI_REGISTRY_IMAGE${tag}" --to "$CI_REGISTRY_IMAGE:latest" --exit-code --only-severity critical,high --ignore-unchanged
    fi

  - docker push "$CI_REGISTRY_IMAGE${tag}"
```

上述流程实现了前面提到的逻辑。如果提交到默认分支，Docker Scout 会生成 CVE 报告；如果提交到其他分支，Docker Scout 会将新版本与当前已发布版本进行对比。仅显示严重（critical）与高危（high）漏洞，并忽略自上次分析以来未发生变化的漏洞。

最后在 YAML 文件中添加：

```yaml
rules:
  - if: $CI_COMMIT_BRANCH
    exists:
      - Dockerfile
```

这些规则确保流水线仅在提交包含 Dockerfile 且提交到 CI 分支时才运行。

## 视频演示

以下是使用 GitLab 设置工作流的视频演示：

<iframe class="border-0 w-full aspect-video mb-8" allow="fullscreen" src="https://www.loom.com/embed/451336c4508c42189532108fc37b2560?sid=f912524b-276d-417d-b44a-c2d39719aa1a"></iframe>
