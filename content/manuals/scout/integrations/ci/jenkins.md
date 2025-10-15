---
description: 如何将 Docker Scout 集成到 Jenkins
keywords: supply chain, security, ci, continuous integration, jenkins
title: 将 Docker Scout 与 Jenkins 集成
linkTitle: Jenkins
---

您可以在 `Jenkinsfile` 中添加以下阶段与步骤定义，将 Docker Scout 作为 Jenkins 流水线的一部分运行。该流水线需要一个包含 Docker Hub 用户名与密码的 `DOCKER_HUB` 凭据，以及为镜像与标签定义的环境变量。

```groovy
pipeline {
    agent {
        // Agent details
    }

    environment {
        DOCKER_HUB = credentials('jenkins-docker-hub-credentials')
        IMAGE_TAG  = 'myorg/scout-demo-service:latest'
    }

    stages {
        stage('Analyze image') {
            steps {
                // Install Docker Scout
                sh 'curl -sSfL https://raw.githubusercontent.com/docker/scout-cli/main/install.sh | sh -s -- -b /usr/local/bin'

                // Log into Docker Hub
                sh 'echo $DOCKER_HUB_PSW | docker login -u $DOCKER_HUB_USR --password-stdin'

                // Analyze and fail on critical or high vulnerabilities
                sh 'docker-scout cves $IMAGE_TAG --exit-code --only-severity critical,high'
            }
        }
    }
}
```

上述配置会安装 Docker Scout，登录 Docker Hub，然后运行 Docker Scout 为指定镜像与标签生成 CVE 报告。仅显示严重（critical）与高危（high）漏洞。

> [!NOTE]
>
> 如果遇到与镜像缓存相关的 `permission denied` 错误，请尝试将 [`DOCKER_SCOUT_CACHE_DIR`](/manuals/scout/how-tos/configure-cli.md) 环境变量设置为可写目录。或者，使用 `DOCKER_SCOUT_NO_CACHE=true` 完全禁用本地缓存。
