---
title: 在 CI 中使用 Docker Build Cloud
linkTitle: 持续集成（CI）
weight: 30
description: 通过在 CI 中使用 Docker Build Cloud 加速持续集成流水线
keywords: 构建, 云构建, ci, gha, gitlab, buildkite, jenkins, circle ci
aliases:
  - /build/cloud/ci/
---

在 CI 中使用 Docker Build Cloud 可以加速您的构建流水线，这意味着更少的等待时间和上下文切换。您可以像往常一样控制 CI 工作流，并将构建执行委托给 Docker Build Cloud。

在 CI 中使用 Docker Build Cloud 进行构建包含以下步骤：

1. 登录 Docker 账户。
2. 设置 Buildx 并连接到构建器。
3. 运行构建。

在 CI 中使用 Docker Build Cloud 时，建议您直接将构建结果推送到仓库，而不是先加载镜像再推送。直接推送可以加快构建速度并避免不必要的文件传输。

如果您只想构建并丢弃输出，请将结果导出到构建缓存或构建时不为镜像添加标签。当您使用 Docker Build Cloud 时，如果您构建的是带标签的镜像，Buildx 会自动加载构建结果。详情请参阅[加载构建结果](./usage/#loading-build-results)。

> [!NOTE]
>
> Docker Build Cloud 上的构建有 90 分钟的超时限制。运行时间超过 90 分钟的构建将被自动取消。

## 为 CI/CD 设置凭证

要使您的 CI/CD 系统能够使用 Docker Build Cloud 构建和推送镜像，需要提供访问令牌和用户名。您使用的令牌类型和用户名取决于您的账户类型和权限。

- 如果您是组织管理员或有权限创建[组织访问令牌 (OAT)](/manuals/enterprise/security/access-tokens.md)，请使用 OAT 并将 `DOCKER_ACCOUNT` 设置为您的 Docker Hub 组织名称。
- 如果您没有权限创建 OAT 或使用的是个人账户，请使用[个人访问令牌 (PAT)](/security/access-tokens/) 并将 `DOCKER_ACCOUNT` 设置为您的 Docker Hub 用户名。

### 创建访问令牌

#### 组织账户

如果您是组织管理员：

- 创建一个[组织访问令牌 (OAT)](/manuals/enterprise/security/access-tokens.md)。该令牌必须具有以下权限：
    1. **cloud-connect** 作用域
    2. **读取公共仓库**权限
    3. **仓库访问**权限，并为目标仓库设置**镜像推送**权限：
        - 展开**仓库**下拉菜单。
        - 选择**添加仓库**并选择您的目标仓库。
        - 为该仓库设置**镜像推送**权限。

如果您不是组织管理员：

- 请向您的组织管理员申请具有上述权限的访问令牌，或使用个人访问令牌。

#### 个人账户

- 创建一个具有以下权限的[个人访问令牌 (PAT)](/security/access-tokens/)：
   1. **读写**访问权限。
        - 注意：使用 Docker Build Cloud 构建只需要读取权限，但您需要写入权限才能将镜像推送到 Docker Hub 仓库。


## CI 平台示例

> [!NOTE]
>
> 在您的 CI/CD 配置中，设置以下变量/密钥：
> - `DOCKER_ACCESS_TOKEN` — 您的访问令牌（PAT 或 OAT）。请使用密钥存储令牌。
> - `DOCKER_ACCOUNT` — 您的 Docker Hub 组织名称（对于 OAT）或用户名（对于 PAT）
> - `CLOUD_BUILDER_NAME` — 您在 [Docker Build Cloud 仪表板](https://app.docker.com/build/)中创建的云构建器名称
>
> 这可确保您的构建能够正确通过 Docker Build Cloud 进行身份验证。

### GitHub Actions

```yaml
name: ci

on:
  push:
    branches:
      - "main"

jobs:
  docker:
    runs-on: ubuntu-latest
    steps:
      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ vars.DOCKER_ACCOUNT }}
          password: ${{ secrets.DOCKER_ACCESS_TOKEN }}
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
        with:
          driver: cloud
          endpoint: "${{ vars.DOCKER_ACCOUNT }}/${{ vars.CLOUD_BUILDER_NAME }}" # 例如，"acme/default"
          install: true
      
      - name: Build and push
        uses: docker/build-push-action@v6
        with:
          tags: "<IMAGE>" # 例如，"acme/my-image:latest"
          # 对于拉取请求，将结果导出到构建缓存。
          # 否则，推送到仓库。
          outputs: ${{ github.event_name == 'pull_request' && 'type=cacheonly' || 'type=registry' }}
```

### GitLab

```yaml
default:
  image: docker:24-dind
  services:
    - docker:24-dind
  before_script:
    - docker info
    - echo "$DOCKER_ACCESS_TOKEN" | docker login --username "$DOCKER_ACCOUNT" --password-stdin
    - |
      apk add curl jq
      ARCH=${CI_RUNNER_EXECUTABLE_ARCH#*/}
      BUILDX_URL=$(curl -s https://raw.githubusercontent.com/docker/actions-toolkit/main/.github/buildx-lab-releases.json | jq -r ".latest.assets[] | select(endswith(\"linux-$ARCH\"))")
      mkdir -vp ~/.docker/cli-plugins/
      curl --silent -L --output ~/.docker/cli-plugins/docker-buildx $BUILDX_URL
      chmod a+x ~/.docker/cli-plugins/docker-buildx
    - docker buildx create --use --driver cloud ${DOCKER_ACCOUNT}/${CLOUD_BUILDER_NAME}

variables:
  IMAGE_NAME: <IMAGE>
  DOCKER_ACCOUNT: <DOCKER_ACCOUNT> # 您的 Docker Hub 组织名称（或使用个人账户时的用户名）
  CLOUD_BUILDER_NAME: <CLOUD_BUILDER_NAME> # 您在 [Docker Build Cloud 仪表板](https://app.docker.com/build/)中创建的云构建器名称

# 构建多平台镜像并推送到仓库
build_push:
  stage: build
  script:
    - |
      docker buildx build \
        --platform linux/amd64,linux/arm64 \
        --tag "${IMAGE_NAME}:${CI_COMMIT_SHORT_SHA}" \
        --push .

# 构建镜像并丢弃结果
build_cache:
  stage: build
  script:
    - |
      docker buildx build \
        --platform linux/amd64,linux/arm64 \
        --tag "${IMAGE_NAME}:${CI_COMMIT_SHORT_SHA}" \
        --output type=cacheonly \
        .
```

### Circle CI

```yaml
version: 2.1

jobs:
  # 构建多平台镜像并推送到仓库
  build_push:
    machine:
      image: ubuntu-2204:current
    steps:
      - checkout

      - run: |
          mkdir -vp ~/.docker/cli-plugins/
          ARCH=amd64
          BUILDX_URL=$(curl -s https://raw.githubusercontent.com/docker/actions-toolkit/main/.github/buildx-lab-releases.json | jq -r ".latest.assets[] | select(endswith(\"linux-$ARCH\"))")
          curl --silent -L --output ~/.docker/cli-plugins/docker-buildx $BUILDX_URL
          chmod a+x ~/.docker/cli-plugins/docker-buildx

      - run: echo "$DOCKER_ACCESS_TOKEN" | docker login --username $DOCKER_ --password-stdin
      - run: docker buildx create --use --driver cloud "${DOCKER_ACCOUNT}/${CLOUD_BUILDER_NAME}"

      - run: |
          docker buildx build \
          --platform linux/amd64,linux/arm64 \
          --push \
          --tag "<IMAGE>" .

  # 构建镜像并丢弃结果
  build_cache:
    machine:
      image: ubuntu-2204:current
    steps:
      - checkout

      - run: |
          mkdir -vp ~/.docker/cli-plugins/
          ARCH=amd64
          BUILDX_URL=$(curl -s https://raw.githubusercontent.com/docker/actions-toolkit/main/.github/buildx-lab-releases.json | jq -r ".latest.assets[] | select(endswith(\"linux-$ARCH\"))")
          curl --silent -L --output ~/.docker/cli-plugins/docker-buildx $BUILDX_URL
          chmod a+x ~/.docker/cli-plugins/docker-buildx

      - run: echo "$DOCKER_ACCESS_TOKEN" | docker login --username $DOCKER_ --password-stdin
      - run: docker buildx create --use --driver cloud "${DOCKER_ACCOUNT}/${CLOUD_BUILDER_NAME}"

      - run: |
          docker buildx build \
          --tag temp \
          --output type=cacheonly \
          .

workflows:
  pull_request:
    jobs:
      - build_cache
  release:
    jobs:
      - build_push
```

### Buildkite

以下示例展示了如何使用 Docker Build Cloud 设置 Buildkite 流水线。该示例假设流水线名称为 `build-push-docker`，并且您使用环境钩子管理 Docker 访问令牌，但您可以根据需要自由调整。

将以下 `environment` 钩子添加到代理的钩子目录：

```bash
#!/bin/bash
set -euo pipefail

if [[ "$BUILDKITE_PIPELINE_NAME" == "build-push-docker" ]]; then
 export DOCKER_ACCESS_TOKEN="<DOCKER_ACCESS_TOKEN>"
fi
```

创建使用 `docker-login` 插件的 `pipeline.yml`：

```yaml
env:
  DOCKER_ACCOUNT: <DOCKER_ACCOUNT> # 您的 Docker Hub 组织名称（或使用个人账户时的用户名）
  CLOUD_BUILDER_NAME: <CLOUD_BUILDER_NAME> # 您在 [Docker Build Cloud 仪表板](https://app.docker.com/build/)中创建的云构建器名称
  IMAGE_NAME: <IMAGE>

steps:
  - command: ./build.sh
    key: build-push
    plugins:
      - docker-login#v2.1.0:
          username: DOCKER_ACCOUNT
          password-env: DOCKER_ACCESS_TOKEN # 环境钩子中的变量名
```

创建 `build.sh` 脚本：

```bash
DOCKER_DIR=/usr/libexec/docker

# 获取最新 buildx 二进制文件的下载链接。
# 将 $ARCH 设置为 CPU 架构（例如 amd64, arm64）
UNAME_ARCH=`uname -m`
case $UNAME_ARCH in
  aarch64)
    ARCH="arm64";
    ;;
  amd64)
    ARCH="amd64";
    ;;
  *)
    ARCH="amd64";
    ;;
esac
BUILDX_URL=$(curl -s https://raw.githubusercontent.com/docker/actions-toolkit/main/.github/buildx-lab-releases.json | jq -r ".latest.assets[] | select(endswith(\"linux-$ARCH\"))")

# 下载支持 Build Cloud 的 docker buildx
curl --silent -L --output $DOCKER_DIR/cli-plugins/docker-buildx $BUILDX_URL
chmod a+x ~/.docker/cli-plugins/docker-buildx

# 连接到您的构建器并将其设置为默认构建器
docker buildx create --use --driver cloud "${DOCKER_ACCOUNT}/${CLOUD_BUILDER_NAME}"

# 仅缓存的镜像构建
docker buildx build \
    --platform linux/amd64,linux/arm64 \
    --tag "$IMAGE_NAME:$BUILDKITE_COMMIT" \
    --output type=cacheonly \
    .

# 构建、标记并推送多架构 docker 镜像
docker buildx build \
    --platform linux/amd64,linux/arm64 \
    --push \
    --tag "$IMAGE_NAME:$BUILDKITE_COMMIT" \
    .
```

### Jenkins

```groovy
pipeline {
  agent any

  environment {
    ARCH = 'amd64'
    DOCKER_ACCESS_TOKEN = credentials('docker-access-token')
    DOCKER_ACCOUNT = credentials('docker-account')
    CLOUD_BUILDER_NAME = '<CLOUD_BUILDER_NAME>'
    IMAGE_NAME = '<IMAGE>'
  }

  stages {
    stage('Build') {
      environment {
        BUILDX_URL = sh (returnStdout: true, script: 'curl -s https://raw.githubusercontent.com/docker/actions-toolkit/main/.github/buildx-lab-releases.json | jq -r ".latest.assets[] | select(endswith(\\"linux-$ARCH\\"))"').trim()
      }
      steps {
        sh 'mkdir -vp ~/.docker/cli-plugins/'
        sh 'curl --silent -L --output ~/.docker/cli-plugins/docker-buildx $BUILDX_URL'
        sh 'chmod a+x ~/.docker/cli-plugins/docker-buildx'
        sh 'echo "$DOCKER_ACCESS_TOKEN" | docker login --username $DOCKER_ACCOUNT --password-stdin'
        sh 'docker buildx create --use --driver cloud "${DOCKER_ACCOUNT}/${CLOUD_BUILDER_NAME}"'
        // 仅缓存构建
        sh 'docker buildx build --platform linux/amd64,linux/arm64 --tag "$IMAGE_NAME" --output type=cacheonly .'
        // 构建并推送多平台镜像
        sh 'docker buildx build --platform linux/amd64,linux/arm64 --push --tag "$IMAGE_NAME" .'
      }
    }
  }
}
```

### Travis CI

```yaml
language: minimal 
dist: jammy 

services:
  - docker

env:
  global:
    - IMAGE_NAME=<IMAGE> # 例如，"acme/my-image:latest"

before_install: |
  echo "$DOCKER_ACCESS_TOKEN" | docker login --username "$DOCKER_ACCOUNT" --password-stdin

install: |
  set -e 
  BUILDX_URL=$(curl -s https://raw.githubusercontent.com/docker/actions-toolkit/main/.github/buildx-lab-releases.json | jq -r ".latest.assets[] | select(endswith(\"linux-$TRAVIS_CPU_ARCH\"))")
  mkdir -vp ~/.docker/cli-plugins/
  curl --silent -L --output ~/.docker/cli-plugins/docker-buildx $BUILDX_URL
  chmod a+x ~/.docker/cli-plugins/docker-buildx
  docker buildx create --use --driver cloud "${DOCKER_ACCOUNT}/${CLOUD_BUILDER_NAME}"

script: |
  docker buildx build \
  --platform linux/amd64,linux/arm64 \
  --push \
  --tag "$IMAGE_NAME" .
```

### BitBucket Pipelines 

```yaml
# 前提条件：$DOCKER_ACCOUNT, $CLOUD_BUILDER_NAME, $DOCKER_ACCESS_TOKEN 已设置为部署变量
# 此流水线假设使用 $BITBUCKET_REPO_SLUG 作为镜像名称

image: atlassian/default-image:3

pipelines:
  default:
    - step:
        name: 构建多平台镜像
        script:
          - mkdir -vp ~/.docker/cli-plugins/
          - ARCH=amd64
          - BUILDX_URL=$(curl -s https://raw.githubusercontent.com/docker/actions-toolkit/main/.github/buildx-lab-releases.json | jq -r ".latest.assets[] | select(endswith(\"linux-$ARCH\"))")
          - curl --silent -L --output ~/.docker/cli-plugins/docker-buildx $BUILDX_URL
          - chmod a+x ~/.docker/cli-plugins/docker-buildx
          - echo "$DOCKER_ACCESS_TOKEN" | docker login --username $DOCKER_ACCOUNT --password-stdin
          - docker buildx create --use --driver cloud "${DOCKER_ACCOUNT}/${CLOUD_BUILDER_NAME}"
          - IMAGE_NAME=$BITBUCKET_REPO_SLUG
          - docker buildx build
            --platform linux/amd64,linux/arm64
            --push
            --tag "$IMAGE_NAME" .
        services:
          - docker
```

### Shell 脚本

```bash
#!/bin/bash

# 获取最新 buildx 二进制文件的下载链接。将 $ARCH 设置为 CPU 架构（例如 amd64, arm64）
ARCH=amd64
BUILDX_URL=$(curl -s https://raw.githubusercontent.com/docker/actions-toolkit/main/.github/buildx-lab-releases.json | jq -r ".latest.assets[] | select(endswith(\"linux-$ARCH\"))")

# 下载支持 Build Cloud 的 docker buildx
mkdir -vp ~/.docker/cli-plugins/
curl --silent -L --output ~/.docker/cli-plugins/docker-buildx $BUILDX_URL
chmod a+x ~/.docker/cli-plugins/docker-buildx

# 使用访问令牌登录 Docker Hub。请参阅 https://docs.docker.com/build-cloud/ci/#creating-access-tokens
echo "$DOCKER_ACCESS_TOKEN" | docker login --username $DOCKER_ACCOUNT --password-stdin

# 连接到您的构建器并将其设置为默认构建器
docker buildx create --use --driver cloud "${DOCKER_ACCOUNT}/${CLOUD_BUILDER_NAME}"

# 仅缓存的镜像构建
docker buildx build \
    --tag temp \
    --output type=cacheonly \
    .

# 构建、标记并推送多架构 docker 镜像
docker buildx build \
    --platform linux/amd64,linux/arm64 \
    --push \
    --tag "<IMAGE>" \
    .
```

### Docker Compose

如果您想在 CI 中使用 `docker compose build` 与 Docker Build Cloud，请使用此实现。

```bash
#!/bin/bash

# 获取最新 buildx 二进制文件的下载链接。将 $ARCH 设置为 CPU 架构（例如 amd64, arm64）
ARCH=amd64
BUILDX_URL=$(curl -s https://raw.githubusercontent.com/docker/actions-toolkit/main/.github/buildx-lab-releases.json | jq -r ".latest.assets[] | select(endswith(\"linux-$ARCH\"))")
COMPOSE_URL=$(curl -sL \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer <GITHUB_TOKEN>" \
  -H "X-GitHub-Api-Version: 2022-11-28" \
  https://api.github.com/repos/docker/compose-desktop/releases \
  | jq "[ .[] | select(.prerelease==false and .draft==false) ] | .[0].assets.[] | select(.name | endswith(\"linux-${ARCH}\")) | .browser_download_url")

# 下载支持 Build Cloud 的 docker buildx
mkdir -vp ~/.docker/cli-plugins/
curl --silent -L --output ~/.docker/cli-plugins/docker-buildx $BUILDX_URL
curl --silent -L --output ~/.docker/cli-plugins/docker-compose $COMPOSE_URL
chmod a+x ~/.docker/cli-plugins/docker-buildx
chmod a+x ~/.docker/cli-plugins/docker-compose

# 使用访问令牌登录 Docker Hub。请参阅 https://docs.docker.com/build-cloud/ci/#creating-access-tokens
echo "$DOCKER_ACCESS_TOKEN" | docker login --username $DOCKER_ACCOUNT --password-stdin

# 连接到您的构建器并将其设置为默认构建器
docker buildx create --use --driver cloud "${DOCKER_ACCOUNT}/${CLOUD_BUILDER_NAME}"

# 构建镜像
docker compose build
```
