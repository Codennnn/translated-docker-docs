---
title: 配置 Docker 套接字异常和高级设置
linkTitle: 配置高级设置
description: 配置增强容器隔离的 Docker 套接字异常和高级设置
keywords: 增强容器隔离, docker 套接字, 配置, testcontainers, 管理员设置
aliases:
 - /desktop/hardened-desktop/enhanced-container-isolation/config/
 - /security/for-admins/hardened-desktop/enhanced-container-isolation/config/
weight: 20
---

{{< summary-bar feature_name="Hardened Docker Desktop" >}}

本页面介绍如何为增强容器隔离（ECI）配置 Docker 套接字异常和其他高级设置。这些配置使 Testcontainers 等可信工具能够与 ECI 协同工作，同时保持安全性。

## Docker 套接字挂载权限

默认情况下，增强容器隔离会阻止容器挂载 Docker 套接字，以防止对 Docker 引擎的恶意访问。但是，某些工具需要访问 Docker 套接字。

常见需要访问 Docker 套接字的场景包括：

- **测试框架**：如 Testcontainers，用于管理测试容器
- **构建工具**：如 Paketo buildpacks，用于创建临时构建容器
- **CI/CD 工具**：作为部署管道的一部分管理容器的工具
- **开发实用程序**：用于容器管理的 Docker CLI 容器

## 配置套接字异常

使用设置管理来配置 Docker 套接字异常：

{{< tabs >}}
{{< tab name="管理控制台" >}}

1. 登录 [Docker Home](https://app.docker.com) 并从左上角账户下拉菜单中选择您的组织。
1. 转到 **管理控制台** > **桌面设置管理**。
1. [创建或编辑设置策略](/manuals/enterprise/security/hardened-desktop/settings-management/configure-admin-console.md)。
1. 找到 **增强容器隔离** 设置。
1. 使用您的可信镜像和命令限制配置 **Docker 套接字访问控制**。

{{< /tab >}}
{{< tab name="JSON 文件" >}}

创建一个 [`admin-settings.json` 文件](/manuals/enterprise/security/hardened-desktop/settings-management/configure-json-file.md) 并添加：

```json
{
  "configurationFileVersion": 2,
  "enhancedContainerIsolation": {
    "locked": true,
    "value": true,
    "dockerSocketMount": {
      "imageList": {
        "images": [
          "docker.io/localstack/localstack:*",
          "docker.io/testcontainers/ryuk:*",
          "docker:cli"
        ],
        "allowDerivedImages": true
      },
      "commandList": {
        "type": "deny",
        "commands": ["push", "build"]
      }
    }
  }
}
```

{{< /tab >}}
{{< /tabs >}}

## 镜像允许列表配置

`imageList` 定义了哪些容器镜像可以挂载 Docker 套接字。

### 镜像引用格式

| 格式  | 描述 |
| :---------------------- | :---------- |
| `<image_name>[:<tag>]`  | 镜像名称，可选标签。如果省略标签，则使用 `:latest` 标签。如果标签是通配符 `*`，则表示"该镜像的任何标签"。 |
| `<image_name>@<digest>` | 镜像名称，带有特定的仓库摘要（例如，由 `docker buildx imagetools inspect <image>` 报告）。这意味着只允许匹配该名称和摘要的镜像。 |

### 配置示例

测试工具的基本允许列表：

```json
"imageList": {
  "images": [
    "docker.io/testcontainers/ryuk:*",
    "docker:cli",
    "alpine:latest"
  ]
}
```

通配符允许列表（Docker Desktop 4.36 及更高版本）：

```json
"imageList": {
  "images": ["*"]
}
```

> [!WARNING]
>
> 使用 `"*"` 允许所有容器挂载 Docker 套接字，这会降低安全性。仅在无法明确列出允许的镜像时使用。

### 安全验证

Docker Desktop 通过以下方式验证允许的镜像：

1. 从仓库下载允许镜像的摘要
1. 在容器启动时将容器镜像摘要与允许列表进行比较
1. 阻止摘要与允许镜像不匹配的容器

这可以防止通过重新标记未经授权的镜像来绕过限制：

```console
$ docker tag malicious-image docker:cli
$ docker run -v /var/run/docker.sock:/var/run/docker.sock docker:cli
# 此操作失败，因为摘要与真实的 docker:cli 镜像不匹配
```

## 派生镜像支持

对于像 Paketo buildpacks 这样创建临时本地镜像的工具，您可以允许从可信基础镜像派生的镜像。

### 启用派生镜像

```json
"imageList": {
  "images": [
    "paketobuildpacks/builder:base"
  ],
  "allowDerivedImages": true
}
```

当 `allowDerivedImages` 为 true 时，从允许的基础镜像（在 Dockerfile 中使用 `FROM`）构建的本地镜像也会获得 Docker 套接字访问权限。

### 派生镜像要求

- **仅限本地镜像**：派生镜像不得存在于远程仓库中
- **基础镜像可用**：必须先在本地拉取父镜像
- **性能影响**：为验证增加最多 1 秒的容器启动时间
- **版本兼容性**：完整的通配符支持需要 Docker Desktop 4.36+

## 命令限制

### 拒绝列表（推荐）

阻止指定命令，同时允许所有其他命令：

```json
"commandList": {
  "type": "deny",
  "commands": ["push", "build", "image*"]
}
```

### 允许列表

仅允许指定命令，同时阻止所有其他命令：

```json
"commandList": {
  "type": "allow",
  "commands": ["ps", "container*", "volume*"]
}
```

### 命令通配符

| 通配符 | 阻止/允许 |
| :---------------- | :---------- |
| `"container\*"`     | 所有 "docker container ..." 命令 |
| `"image\*"`         | 所有 "docker image ..." 命令 |
| `"volume\*"`        | 所有 "docker volume ..." 命令 |
| `"network\*"`       | 所有 "docker network ..." 命令 |
| `"build\*"`         | 所有 "docker build ..." 命令 |
| `"system\*"`        | 所有 "docker system ..." 命令 |

### 命令阻止示例

当执行被阻止的命令时：

```console
/ # docker push myimage
Error response from daemon: enhanced container isolation: docker command "/v1.43/images/myimage/push?tag=latest" is blocked; if you wish to allow it, configure the docker socket command list in the Docker Desktop settings.
```

## 常见配置示例

### Testcontainers 设置

对于使用 Testcontainers 的 Java/Python 测试：

```json
"dockerSocketMount": {
  "imageList": {
    "images": [
      "docker.io/testcontainers/ryuk:*",
      "testcontainers/*:*"
    ]
  },
  "commandList": {
    "type": "deny",
    "commands": ["push", "build"]
  }
}
```

### CI/CD 管道工具

用于受控的 CI/CD 容器管理：

```json
"dockerSocketMount": {
  "imageList": {
    "images": [
      "docker:cli",
      "your-registry.com/ci-tools/*:*"
    ]
  },
  "commandList": {
    "type": "allow",
    "commands": ["ps", "container*", "image*"]
  }
}
```

### 开发环境

用于使用 Docker-in-Docker 的本地开发：

```json
"dockerSocketMount": {
  "imageList": {
    "images": [
      "docker:dind",
      "docker:cli"
    ]
  },
  "commandList": {
    "type": "deny",
    "commands": ["system*"]
  }
}
```

## 安全建议

### 镜像允许列表最佳实践

- **严格限制**：仅允许您绝对信任和需要的镜像
- **谨慎使用通配符**：标签通配符（`*`）虽然方便，但不如特定标签安全
- **定期审查**：定期审查和更新您的允许列表
- **摘要固定**：在关键环境中使用摘要引用以获得最大安全性

### 命令限制

- **默认拒绝**：从拒绝列表开始，阻止 `push` 和 `build` 等危险命令
- **最小权限原则**：仅允许您的工具实际需要的命令
- **监控使用情况**：跟踪哪些命令被阻止以完善您的配置

### 监控和维护

- **定期验证**：在 Docker Desktop 更新后测试您的配置，因为镜像摘要可能会发生变化。
- **处理摘要不匹配**：如果允许的镜像被意外阻止：
    ```console
    $ docker image rm <image>
    $ docker pull <image>
    ```

这可以解决上游镜像更新时的摘要不匹配问题。

## 后续步骤

- 查看[增强容器隔离限制](/manuals/enterprise/security/hardened-desktop/enhanced-container-isolation/limitations.md)。
- 查看[增强容器隔离常见问题](/manuals/enterprise/security/hardened-desktop/enhanced-container-isolation/faq.md)。
