---
title: 为扩展添加后端
description: 了解如何为您的扩展添加后端。
keywords: Docker, extensions, sdk, build
aliases:
 - /desktop/extensions-sdk/tutorials/minimal-backend-extension/
 - /desktop/extensions-sdk/build/minimal-backend-extension/
 - /desktop/extensions-sdk/build/set-up/backend-extension-tutorial/
 - /desktop/extensions-sdk/build/backend-extension-tutorial/
---

您的扩展可以包含一个后端部分，前端可以与之交互。本文介绍了为何以及如何添加后端。

开始之前，请确保已安装最新版本的 [Docker Desktop](https://www.docker.com/products/docker-desktop/)。

> 提示
>
> 查看[快速入门指南](../quickstart.md)和 `docker extension init <my-extension>` 命令。它们为您的扩展提供了更好的基础，因为它们更加最新且与您安装的 Docker Desktop 相关。

## 为何要添加后端？

得益于 Docker Extensions SDK，大多数情况下您应该能够直接从[前端](frontend-extension-tutorial.md#use-the-extension-apis-client)使用 Docker CLI 完成所需操作。

尽管如此，在某些情况下您可能需要为扩展添加后端。到目前为止，扩展开发者使用后端的目的包括：
- 在本地数据库中存储数据并通过 REST API 提供服务。
- 存储扩展状态，例如当按钮启动长时间运行的过程时，这样如果您离开扩展用户界面后再回来，前端可以从上次中断的地方继续。

有关扩展后端的更多信息，请参阅[架构](../architecture/_index.md#the-backend)。

## 为扩展添加后端

如果您使用 `docker extension init` 命令创建了扩展，那么您已经有了后端设置。否则，您需要首先创建一个包含代码的 `vm` 目录，并更新 Dockerfile 以将其容器化。

以下是带有后端的扩展文件夹结构：

```bash
.
├── Dockerfile # (1)
├── Makefile
├── metadata.json
├── ui
    └── index.html
└── vm # (2)
    ├── go.mod
    └── main.go
```

1. 包含构建后端并将其复制到扩展容器文件系统所需的全部内容。
2. 包含扩展后端代码的源文件夹。

虽然您可以从空目录或 `vm-ui extension` [示例](https://github.com/docker/extensions-sdk/tree/main/samples)开始，但强烈建议您从 `docker extension init` 命令开始，并根据需要进行修改。

> [!TIP]
>
> `docker extension init` 命令会生成一个 Go 后端。但您仍然可以将其作为自己扩展的起点，并使用任何其他语言，如 Node.js、Python、Java、.Net 或任何其他语言和框架。

在本教程中，后端服务仅公开一个返回 "Hello" 的 JSON 负载的路由。

```json
{ "Message": "Hello" }
```

> [!IMPORTANT]
>
> 我们建议前端和后端通过套接字（在 Windows 上通过命名管道）进行通信，而不是使用 HTTP。这可以防止与主机上运行的任何其他应用程序或容器发生端口冲突。此外，一些 Docker Desktop 用户在受限环境中运行，他们无法在机器上打开端口。在选择后端的语言和框架时，请确保它支持套接字连接。

{{< tabs group="lang" >}}
{{< tab name="Go" >}}

```go
package main

import (
	"flag"
	"log"
	"net"
	"net/http"
	"os"

	"github.com/labstack/echo"
	"github.com/sirupsen/logrus"
)

func main() {
	var socketPath string
	flag.StringVar(&socketPath, "socket", "/run/guest/volumes-service.sock", "Unix domain socket to listen on")
	flag.Parse()

	os.RemoveAll(socketPath)

	logrus.New().Infof("Starting listening on %s\n", socketPath)
	router := echo.New()
	router.HideBanner = true

	startURL := ""

	ln, err := listen(socketPath)
	if err != nil {
		log.Fatal(err)
	}
	router.Listener = ln

	router.GET("/hello", hello)

	log.Fatal(router.Start(startURL))
}

func listen(path string) (net.Listener, error) {
	return net.Listen("unix", path)
}

func hello(ctx echo.Context) error {
	return ctx.JSON(http.StatusOK, HTTPMessageBody{Message: "hello world"})
}

type HTTPMessageBody struct {
	Message string
}
```

{{< /tab >}}
{{< tab name="Node" >}}

> [!IMPORTANT]
>
> 我们还没有 Node 的工作示例。[填写表单](https://docs.google.com/forms/d/e/1FAIpQLSdxJDGFJl5oJ06rG7uqtw1rsSBZpUhv_s9HHtw80cytkh2X-Q/viewform?usp=pp_url&entry.25798127=Node)
> 并告诉我们您是否需要 Node 的示例。

{{< /tab >}}
{{< tab name="Python" >}}

> [!IMPORTANT]
>
> 我们还没有 Python 的工作示例。[填写表单](https://docs.google.com/forms/d/e/1FAIpQLSdxJDGFJl5oJ06rG7uqtw1rsSBZpUhv_s9HHtw80cytkh2X-Q/viewform?usp=pp_url&entry.25798127=Python)
> 并告诉我们您是否需要 Python 的示例。

{{< /tab >}}
{{< tab name="Java" >}}

> [!IMPORTANT]
>
> 我们还没有 Java 的工作示例。[填写表单](https://docs.google.com/forms/d/e/1FAIpQLSdxJDGFJl5oJ06rG7uqtw1rsSBZpUhv_s9HHtw80cytkh2X-Q/viewform?usp=pp_url&entry.25798127=Java)
> 并告诉我们您是否需要 Java 的示例。

{{< /tab >}}
{{< tab name=".NET" >}}

> [!IMPORTANT]
>
> 我们还没有 .NET 的工作示例。[填写表单](https://docs.google.com/forms/d/e/1FAIpQLSdxJDGFJl5oJ06rG7uqtw1rsSBZpUhv_s9HHtw80cytkh2X-Q/viewform?usp=pp_url&entry.25798127=.Net)
> 并告诉我们您是否需要 .NET 的示例。

{{< /tab >}}
{{< /tabs >}}

## 调整 Dockerfile

> [!NOTE]
>
> 使用 `docker extension init` 时，它会创建一个已经包含 Go 后端所需内容的 `Dockerfile`。

{{< tabs group="lang" >}}
{{< tab name="Go" >}}

要在安装扩展时部署 Go 后端，您需要首先配置 `Dockerfile`，使其：
- 构建后端应用程序
- 将二进制文件复制到扩展的容器文件系统
- 在容器启动时启动二进制文件，监听扩展套接字

> [!TIP]
> 
> 为了简化版本管理，您可以重用同一镜像来构建前端、构建后端服务和打包扩展。

```dockerfile
# syntax=docker/dockerfile:1
FROM node:17.7-alpine3.14 AS client-builder
# ... 构建前端应用程序

# 构建 Go 后端
FROM golang:1.17-alpine AS builder
ENV CGO_ENABLED=0
WORKDIR /backend
COPY vm/go.* .
RUN --mount=type=cache,target=/go/pkg/mod \
    --mount=type=cache,target=/root/.cache/go-build \
    go mod download
COPY vm/. .
RUN --mount=type=cache,target=/go/pkg/mod \
    --mount=type=cache,target=/root/.cache/go-build \
    go build -trimpath -ldflags="-s -w" -o bin/service

FROM alpine:3.15
# ... 添加标签并复制前端应用程序

COPY --from=builder /backend/bin/service /
CMD /service -socket /run/guest-services/extension-allthethings-extension.sock
```

{{< /tab >}}
{{< tab name="Node" >}}

> [!IMPORTANT]
>
> 我们还没有 Node 的工作 Dockerfile。[填写表单](https://docs.google.com/forms/d/e/1FAIpQLSdxJDGFJl5oJ06rG7uqtw1rsSBZpUhv_s9HHtw80cytkh2X-Q/viewform?usp=pp_url&entry.25798127=Node)
> 并告诉我们您是否需要 Node 的 Dockerfile。

{{< /tab >}}
{{< tab name="Python" >}}

> [!IMPORTANT]
>
> 我们还没有 Python 的工作 Dockerfile。[填写表单](https://docs.google.com/forms/d/e/1FAIpQLSdxJDGFJl5oJ06rG7uqtw1rsSBZpUhv_s9HHtw80cytkh2X-Q/viewform?usp=pp_url&entry.25798127=Python)
> 并告诉我们您是否需要 Python 的 Dockerfile。

{{< /tab >}}
{{< tab name="Java" >}}

> [!IMPORTANT]
>
> 我们还没有 Java 的工作 Dockerfile。[填写表单](https://docs.google.com/forms/d/e/1FAIpQLSdxJDGFJl5oJ06rG7uqtw1rsSBZpUhv_s9HHtw80cytkh2X-Q/viewform?usp=pp_url&entry.25798127=Java)
> 并告诉我们您是否需要 Java 的 Dockerfile。

{{< /tab >}}
{{< tab name=".NET" >}}

> [!IMPORTANT]
>
> 我们还没有 .NET 的工作 Dockerfile。[填写表单](https://docs.google.com/forms/d/e/1FAIpQLSdxJDGFJl5oJ06rG7uqtw1rsSBZpUhv_s9HHtw80cytkh2X-Q/viewform?usp=pp_url&entry.25798127=.Net)
> 并告诉我们您是否需要 .NET 的 Dockerfile。

{{< /tab >}}
{{< /tabs >}}

## 配置元数据文件

要在 Docker Desktop 的虚拟机内启动扩展的后端服务，您必须在 `metadata.json` 文件的 `vm` 部分配置镜像名称。

```json
{
  "vm": {
    "image": "${DESKTOP_PLUGIN_IMAGE}"
  },
  "icon": "docker.svg",
  "ui": {
    ...
  }
}
```

有关 `metadata.json` 文件中 `vm` 部分的更多信息，请参阅[元数据](../architecture/metadata.md)。

> [!WARNING]
>
> 不要替换 `metadata.json` 文件中的 `${DESKTOP_PLUGIN_IMAGE}` 占位符。安装扩展时，该占位符会自动替换为正确的镜像名称。

## 从前端调用扩展后端

使用[高级前端扩展示例](frontend-extension-tutorial.md)，我们可以调用我们的扩展后端。

使用 Docker Desktop Client 对象，然后使用 `ddClient.extension.vm.service.get` 从后端服务调用 `/hello` 路由，该方法返回响应的主体。

{{< tabs group="framework" >}}
{{< tab name="React" >}}

将 `ui/src/App.tsx` 文件替换为以下代码：

```tsx

// ui/src/App.tsx
import React, { useEffect } from 'react';
import { createDockerDesktopClient } from "@docker/extension-api-client";

//获取 docker desktop 扩展客户端
const ddClient = createDockerDesktopClient();

export function App() {
  const ddClient = createDockerDesktopClient();
  const [hello, setHello] = useState<string>();

  useEffect(() => {
    const getHello = async () => {
      const result = await ddClient.extension.vm?.service?.get('/hello');
      setHello(JSON.stringify(result));
    }
    getHello()
  }, []);

  return (
    <Typography>{hello}</Typography>
  );
}

```

{{< /tab >}}
{{< tab name="Vue" >}}

> [!IMPORTANT]
>
> 我们还没有 Vue 的示例。[填写表单](https://docs.google.com/forms/d/e/1FAIpQLSdxJDGFJl5oJ06rG7uqtw1rsSBZpUhv_s9HHtw80cytkh2X-Q/viewform?usp=pp_url&entry.1333218187=Vue)
> 并告诉我们您是否需要 Vue 的示例。

{{< /tab >}}
{{< tab name="Angular" >}}

> [!IMPORTANT]
>
> 我们还没有 Angular 的示例。[填写表单](https://docs.google.com/forms/d/e/1FAIpQLSdxJDGFJl5oJ06rG7uqtw1rsSBZpUhv_s9HHtw80cytkh2X-Q/viewform?usp=pp_url&entry.1333218187=Angular)
> 并告诉我们您是否需要 Angular 的示例。

{{< /tab >}}
{{< tab name="Svelte" >}}

> [!IMPORTANT]
>
> 我们还没有 Svelte 的示例。[填写表单](https://docs.google.com/forms/d/e/1FAIpQLSdxJDGFJl5oJ06rG7uqtw1rsSBZpUhv_s9HHtw80cytkh2X-Q/viewform?usp=pp_url&entry.1333218187=Svelte)
> 并告诉我们您是否需要 Svelte 的示例。

{{< /tab >}}
{{< /tabs >}}

## 重新构建并更新扩展

由于您已修改扩展的配置并在 Dockerfile 中添加了一个阶段，因此必须重新构建扩展。

```bash
docker build --tag=awesome-inc/my-extension:latest .
```

构建完成后，您需要更新它，或者如果尚未安装，则安装它。

```bash
docker extension update awesome-inc/my-extension:latest
```

现在您可以在 Docker Desktop 仪表板的**容器**视图中看到正在运行的后端服务，并在需要调试时查看日志。

> [!TIP]
>
> 您可能需要在**设置**中打开**显示系统容器**选项才能看到正在运行的后端容器。
> 有关更多信息，请参阅[显示扩展容器](../dev/test-debug.md#show-the-extension-containers)。

打开 Docker Desktop 仪表板并选择**容器**选项卡。您应该能看到显示的后端服务调用的响应。

## 下一步

- 学习如何[共享和发布您的扩展](../extensions/_index.md)。
- 了解更多关于扩展[架构](../architecture/_index.md)的信息。
