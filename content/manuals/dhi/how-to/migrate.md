---
title: 将现有应用程序迁移到使用 Docker 加固镜像
linktitle: 迁移应用程序
description: 按照分步指南更新您的 Dockerfile 并采用 Docker 加固镜像，实现安全、最小化和生产就绪的构建。
weight: 50
keywords: 迁移 dockerfile, 加固基础镜像, 多阶段构建, 非 root 容器, 安全容器构建
---

{{< summary-bar feature_name="Docker Hardened Images" >}}

本指南帮助您将现有的 Dockerfile 迁移到使用 Docker 加固镜像（DHIs），可以通过[手动方式](#step-1-更新您的-dockerfile-中的基础镜像)或[使用 Gordon](#使用-gordon) 来完成。
DHIs 是最小化且以安全为重点的镜像，这可能需要调整您的基础镜像、构建过程和运行时配置。

本指南重点介绍框架镜像的迁移，例如使用 Go、Python 或 Node.js 等语言从源代码构建应用程序的镜像。如果您要迁移应用程序镜像，如数据库、代理或其他预构建服务，许多相同的原理仍然适用。

## 迁移注意事项

DHIs 省略了常见的工具（如 shell 和包管理器）以减少攻击面。它们还默认以非 root 用户身份运行。因此，迁移到 DHI 通常需要对您的 Dockerfile 进行以下更改：


| 项目               | 迁移说明                                                                                                                                                                                                                                                                                                                       |
|:-------------------|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 基础镜像           | 在您的 Dockerfile 中将基础镜像替换为 Docker 加固镜像。                                                                                                                                                                                                                                                                          |
| 包管理             | 用于运行时的镜像不包含包管理器。仅在带有 `dev` 标签的镜像中使用包管理器。利用多阶段构建，将必要的构件从构建阶段复制到运行时阶段。                                                                                                                                                                                          |
| 非 root 用户       | 默认情况下，用于运行时的镜像以非 root 用户身份运行。确保必要的文件和目录对非 root 用户可访问。                                                                                                                                                                                                                              |
| 多阶段构建         | 在构建阶段使用带有 `dev` 或 `sdk` 标签的镜像，在运行时阶段使用非开发镜像。                                                                                                                                                                                                                                                  |
| TLS 证书           | DHIs 默认包含标准 TLS 证书。无需安装 TLS 证书。                                                                                                                                                                                                                                                                             |
| 端口               | 用于运行时的 DHIs 默认以非 root 用户身份运行。因此，在 Kubernetes 或 Docker Engine 20.10 之前的版本中运行时，这些镜像中的应用程序无法绑定到特权端口（低于 1024）。为避免问题，请将您的应用程序配置为在容器内监听 1025 或更高端口。                                                                                     |
| 入口点             | DHIs 的入口点可能与 Docker 官方镜像等不同。检查 DHIs 的入口点，如有必要请更新您的 Dockerfile。                                                                                                                                                                                                                              |
| 无 shell           | 用于运行时的 DHIs 不包含 shell。在构建阶段使用开发镜像运行 shell 命令，然后将构件复制到运行时阶段。                                                                                                                                                                                                                        |

如需更多详细信息和故障排除提示，请参阅[故障排除](/manuals/dhi/troubleshoot.md)。

## 迁移现有应用程序

以下步骤概述了迁移过程。

### 步骤 1：更新您的 Dockerfile 中的基础镜像

将应用程序 Dockerfile 中的基础镜像更新为加固镜像。这通常是标记为 `dev` 或 `sdk` 的镜像，因为它具有安装包和依赖项所需的工具。

以下 Dockerfile 的示例差异片段显示了旧的基础镜像被新的加固镜像替换。

```diff
- ## 原始基础镜像
- FROM golang:1.22

+ ## 更新为使用加固基础镜像
+ FROM <your-namespace>/dhi-golang:1.22-dev
```

### 步骤 2：更新您的 Dockerfile 中的运行时镜像

> [!NOTE]
>
> 建议使用多阶段构建以保持最终镜像最小化和安全。单阶段构建受支持，但它们包含完整的 `dev` 镜像，因此会导致更大的镜像和更广泛的攻击面。

为确保最终镜像尽可能最小化，您应该使用[多阶段构建](/manuals/build/building/multi-stage.md)。Dockerfile 中的所有阶段都应该使用加固镜像。虽然中间阶段通常使用标记为 `dev` 或 `sdk` 的镜像，但您的最终运行时阶段应该使用运行时镜像。

利用构建阶段编译您的应用程序，并将生成的构件复制到最终的运行时阶段。这确保您的最终镜像是最小化和安全的。

请参阅[示例 Dockerfile 迁移](#示例-dockerfile-迁移)部分，了解如何更新您的 Dockerfile 的示例。

## 示例 Dockerfile 迁移

以下示例显示了迁移前后的 Dockerfile。每个示例都包括多阶段构建（推荐用于最小化、安全的镜像）和单阶段构建（受支持，但会导致更大的镜像和更广泛的攻击面）。

> [!NOTE]
>
> 多阶段构建推荐用于大多数用例。单阶段构建为简单起见受支持，但在大小和安全性方面需要权衡。

### Go 示例

{{< tabs >}}
{{< tab name="迁移前" >}}

```dockerfile
#syntax=docker/dockerfile:1

FROM golang:latest

WORKDIR /app
ADD . ./
RUN CGO_ENABLED=0 GOOS=linux go build -a -ldflags="-s -w" --installsuffix cgo -o main .

ENTRYPOINT ["/app/main"]
```

{{< /tab >}}
{{< tab name="迁移后（多阶段）" >}}

```dockerfile
#syntax=docker/dockerfile:1

# === 构建阶段：编译 Go 应用程序 ===
FROM <your-namespace>/dhi-golang:1-alpine3.21-dev AS builder

WORKDIR /app
ADD . ./
RUN CGO_ENABLED=0 GOOS=linux go build -a -ldflags="-s -w" --installsuffix cgo -o main .

# === 最终阶段：创建最小化运行时镜像 ===
FROM <your-namespace>/dhi-golang:1-alpine3.21

WORKDIR /app
COPY --from=builder /app/main  /app/main

ENTRYPOINT ["/app/main"]
```

{{< /tab >}}
{{< tab name="迁移后（单阶段）" >}}

```dockerfile
#syntax=docker/dockerfile:1

FROM <your-namespace>/dhi-golang:1-alpine3.21-dev

WORKDIR /app
ADD . ./
RUN CGO_ENABLED=0 GOOS=linux go build -a -ldflags="-s -w" --installsuffix cgo -o main .

ENTRYPOINT ["/app/main"]
```

{{< /tab >}}
{{< /tabs >}}

### Node.js 示例

{{< tabs >}}
{{< tab name="迁移前" >}}

```dockerfile
#syntax=docker/dockerfile:1

FROM node:latest
WORKDIR /usr/src/app

COPY package*.json ./
RUN npm install

COPY image.jpg ./image.jpg
COPY . .

CMD ["node", "index.js"]
```

{{< /tab >}}
{{< tab name="迁移后（多阶段）" >}}

```dockerfile
#syntax=docker/dockerfile:1

#=== 构建阶段：安装依赖项并构建应用程序 ===#
FROM <your-namespace>/dhi-node:23-alpine3.21-dev AS builder
WORKDIR /usr/src/app

COPY package*.json ./
RUN npm install

COPY image.jpg ./image.jpg
COPY . .

#=== 最终阶段：创建最小化运行时镜像 ===#
FROM <your-namespace>/dhi-node:23-alpine3.21
ENV PATH=/app/node_modules/.bin:$PATH

COPY --from=builder --chown=node:node /usr/src/app /app

WORKDIR /app

CMD ["index.js"]
```

{{< /tab >}}
{{< tab name="迁移后（单阶段）" >}}

```dockerfile
#syntax=docker/dockerfile:1

FROM <your-namespace>/dhi-node:23-alpine3.21-dev
WORKDIR /usr/src/app

COPY package*.json ./
RUN npm install

COPY image.jpg ./image.jpg
COPY . .

CMD ["index.js"]
```

{{< /tab >}}
{{< /tabs >}}

### Python 示例

{{< tabs >}}
{{< tab name="迁移前" >}}

```dockerfile
#syntax=docker/dockerfile:1

FROM python:latest AS builder

ENV LANG=C.UTF-8
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1
ENV PATH="/app/venv/bin:$PATH"

WORKDIR /app

RUN python -m venv /app/venv
COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

FROM python:latest

WORKDIR /app

ENV PYTHONUNBUFFERED=1
ENV PATH="/app/venv/bin:$PATH"

COPY image.py image.png ./
COPY --from=builder /app/venv /app/venv

ENTRYPOINT [ "python", "/app/image.py" ]
```

{{< /tab >}}
{{< tab name="迁移后（多阶段）" >}}

```dockerfile
#syntax=docker/dockerfile:1

#=== 构建阶段：安装依赖项并创建虚拟环境 ===#
FROM <your-namespace>/dhi-python:3.13-alpine3.21-dev AS builder

ENV LANG=C.UTF-8
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1
ENV PATH="/app/venv/bin:$PATH"

WORKDIR /app

RUN python -m venv /app/venv
COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

#=== 最终阶段：创建最小化运行时镜像 ===#
FROM <your-namespace>/dhi-python:3.13-alpine3.21

WORKDIR /app

ENV PYTHONUNBUFFERED=1
ENV PATH="/app/venv/bin:$PATH"

COPY image.py image.png ./
COPY --from=builder /app/venv /app/venv

ENTRYPOINT [ "python", "/app/image.py" ]
```

{{< /tab >}}
{{< tab name="迁移后（单阶段）" >}}

```dockerfile
#syntax=docker/dockerfile:1

FROM <your-namespace>/dhi-python:3.13-alpine3.21-dev

ENV LANG=C.UTF-8
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1
ENV PATH="/app/venv/bin:$PATH"

WORKDIR /app

RUN python -m venv /app/venv
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY image.py image.png ./

ENTRYPOINT [ "python", "/app/image.py" ]
```

{{< /tab >}}
{{< /tabs >}}

### 使用 Gordon

或者，您可以请求 [Gordon](/manuals/ai/gordon/_index.md)（Docker 的 AI 助手）协助迁移您的 Dockerfile：

{{% include "gordondhi.md" %}}
