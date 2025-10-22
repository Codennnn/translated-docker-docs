---
title: 如何测试加固镜像
description: 了解如何验证和测试 Docker 加固镜像的安全性、完整性和功能性。
keywords: 加固镜像, 测试, 验证, 安全扫描, 合规性检查
weight: 40
---

测试 Docker 加固镜像是确保其满足您的安全、合规和功能要求的关键步骤。本指南介绍了测试加固镜像的各种方法和最佳实践。

## 安全性测试

### 漏洞扫描

使用容器安全扫描工具验证加固镜像的安全状态：

```bash
# 使用 Docker Scout 扫描镜像
docker scout cves <image-name>

# 使用 Trivy 扫描镜像
trivy image <image-name>

# 使用 Grype 扫描镜像
grype <image-name>
```

### 配置安全检查

验证镜像配置是否符合安全最佳实践：

```bash
# 使用 Docker Bench for Security
docker run --rm --net host --pid host --userns host --cap-add audit_control \
  -e DOCKER_CONTENT_TRUST=$DOCKER_CONTENT_TRUST \
  -v /etc:/etc:ro \
  -v /usr/bin/containerd:/usr/bin/containerd:ro \
  -v /usr/bin/runc:/usr/bin/runc:ro \
  -v /usr/lib/systemd:/usr/lib/systemd:ro \
  -v /var/lib:/var/lib:ro \
  -v /var/run/docker.sock:/var/run/docker.sock:ro \
  --label docker_bench_security \
  docker/docker-bench-security
```

## 完整性验证

### 签名验证

验证镜像的数字签名和来源：

```bash
# 启用 Docker Content Trust
export DOCKER_CONTENT_TRUST=1

# 拉取并验证签名镜像
docker pull <signed-image>

# 使用 Cosign 验证签名
cosign verify --key <public-key> <image-name>
```

### SBOM 验证

检查软件物料清单（SBOM）：

```bash
# 使用 Syft 生成 SBOM
syft <image-name> -o spdx-json

# 使用 Docker Scout 查看 SBOM
docker scout sbom <image-name>
```

## 功能性测试

### 基本功能测试

验证镜像的基本功能：

```bash
# 运行基本容器测试
docker run --rm <image-name> <test-command>

# 检查镜像层信息
docker history <image-name>

# 检查镜像配置
docker inspect <image-name>
```

### 应用程序兼容性测试

测试您的应用程序在加固镜像上的运行情况：

```dockerfile
# 创建测试 Dockerfile
FROM <hardened-image>
COPY . /app
WORKDIR /app
RUN <build-commands>
CMD <start-command>
```

```bash
# 构建和测试应用程序
docker build -t test-app .
docker run --rm test-app <test-suite>
```

## 合规性测试

### CIS 基准测试

使用 CIS（Center for Internet Security）基准测试镜像：

```bash
# 使用 InSpec 运行 CIS 基准测试
inspec exec https://github.com/dev-sec/cis-docker-benchmark.git
```

### STIG 合规性检查

对于需要 STIG（Security Technical Implementation Guide）合规的环境：

```bash
# 使用 SCAP 工具进行 STIG 检查
oscap xccdf eval --profile stig-rhel8-disa \
  --results results.xml \
  --report report.html \
  ssg-rhel8-ds.xml
```

## 性能测试

### 镜像大小分析

分析镜像大小和层结构：

```bash
# 使用 dive 分析镜像层
dive <image-name>

# 比较镜像大小
docker images | grep <image-pattern>
```

### 启动时间测试

测量容器启动时间：

```bash
# 测量启动时间
time docker run --rm <image-name> echo "Container started"

# 使用 hyperfine 进行基准测试
hyperfine 'docker run --rm <image-name> <command>'
```

## 自动化测试

### CI/CD 集成

将测试集成到 CI/CD 流水线中：

```yaml
# GitHub Actions 示例
name: Test Hardened Images
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Security Scan
        run: |
          docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
            aquasec/trivy image <image-name>
      - name: Functional Test
        run: |
          docker run --rm <image-name> <test-command>
```

### 测试脚本示例

创建自动化测试脚本：

```bash
#!/bin/bash
# test-hardened-image.sh

IMAGE_NAME=$1
TEST_RESULTS="test-results.json"

echo "Testing hardened image: $IMAGE_NAME"

# 安全扫描
echo "Running security scan..."
trivy image --format json --output security-scan.json $IMAGE_NAME

# 功能测试
echo "Running functional tests..."
docker run --rm $IMAGE_NAME /bin/sh -c "echo 'Functional test passed'" || exit 1

# 合规性检查
echo "Running compliance checks..."
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  docker/docker-bench-security > compliance-report.txt

echo "All tests completed successfully!"
```

## 测试最佳实践

### 测试策略

1. **分层测试**：从基础安全测试到应用程序特定测试
2. **自动化优先**：尽可能自动化所有测试
3. **持续测试**：在 CI/CD 流水线中集成测试
4. **文档记录**：记录测试结果和发现的问题

### 测试环境

1. **隔离环境**：在隔离的测试环境中运行测试
2. **生产模拟**：测试环境应尽可能模拟生产环境
3. **多平台测试**：在不同的平台和架构上测试镜像

### 结果分析

1. **基线建立**：建立安全和性能基线
2. **趋势分析**：跟踪测试结果的趋势变化
3. **风险评估**：评估发现的问题的风险级别
4. **修复跟踪**：跟踪问题的修复状态

通过系统性的测试方法，您可以确保 Docker 加固镜像满足您的安全、合规和功能要求，为生产部署提供可靠的基础。
