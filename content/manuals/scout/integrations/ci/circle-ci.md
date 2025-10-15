---
description: 如何将 Docker Scout 集成到 Circle CI
keywords: supply chain, security, ci, continuous integration, circle ci
title: 将 Docker Scout 与 Circle CI 集成
linkTitle: Circle CI
---

下面的示例在 CircleCI 中触发时运行。触发后，它会检出 "docker/scout-demo-service:latest" 镜像与标签，然后使用 Docker Scout 创建 CVE 报告。

在 _.circleci/config.yml_ 文件中添加以下内容。

首先，设置整体工作流。在 YAML 文件中添加：

```yaml
version: 2.1

jobs:
  build:
    docker:
      - image: cimg/base:stable
    environment:
      IMAGE_TAG: docker/scout-demo-service:latest
```

这定义了工作流使用的容器镜像以及镜像的环境变量。

继续在 YAML 文件中添加工作流步骤：

```yaml
steps:
  # Checkout the repository files
  - checkout
  
  # Set up a separate Docker environment to run `docker` commands in
  - setup_remote_docker:
      version: 20.10.24

  # Install Docker Scout and login to Docker Hub
  - run:
      name: Install Docker Scout
      command: |
        env
        curl -sSfL https://raw.githubusercontent.com/docker/scout-cli/main/install.sh | sh -s -- -b /home/circleci/bin
        echo $DOCKER_HUB_PAT | docker login -u $DOCKER_HUB_USER --password-stdin

  # Build the Docker image
  - run:
      name: Build Docker image
      command: docker build -t $IMAGE_TAG .
  
  # Run Docker Scout          
  - run:
      name: Scan image for CVEs
      command: |
        docker-scout cves $IMAGE_TAG --exit-code --only-severity critical,high
```

上述配置会检出仓库文件，然后设置一个独立的 Docker 环境来运行命令。

它会安装 Docker Scout，登录 Docker Hub，构建 Docker 镜像，然后运行 Docker Scout 生成 CVE 报告。仅显示严重（critical）与高危（high）漏洞。

最后，为工作流及其任务添加名称：

```yaml
workflows:
  build-docker-image:
    jobs:
      - build
```
