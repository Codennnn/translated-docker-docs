---
title: 自定义 Compose Bridge 
linkTitle: 自定义
weight: 20
description: 了解如何使用 Go 模板与 Compose 扩展自定义 Compose Bridge 的转换
keywords: docker compose bridge, customize compose bridge, compose bridge templates, compose to kubernetes, compose bridge transformation, go templates docker
---

{{< summary-bar feature_name="Compose bridge" >}}

本文介绍 Compose Bridge 如何利用模板高效地将 Docker Compose 文件转换为 Kubernetes 清单；同时说明如何根据你的实际需求定制这些模板，或自行构建转换。

## 工作原理 

Compose Bridge 使用转换（transformation）机制，将 Compose 模型转换为其它形式。 

每个转换被打包为一个 Docker 镜像：它接收完整解析后的 Compose 模型（位于 `/in/compose.yaml`），并在 `/out` 目录下生成任意目标格式的文件。

Compose Bridge 内置了使用 Go 模板实现的默认 Kubernetes 转换，你可以通过替换或扩展模板来进行自定义。

### 语法

Compose Bridge 借助模板将 Compose 配置文件转换为 Kubernetes 清单。模板是使用 [Go 模板语法](https://pkg.go.dev/text/template) 的纯文本文件，支持插入逻辑与数据，使其能够根据 Compose 模型进行动态渲染与适配。

执行模板时，必须输出 YAML 文件（Kubernetes 清单的标准格式）。可以一次生成多个文件，只需用 `---` 进行分隔。

每个 YAML 输出文件以自定义头标识开头，例如：

```yaml
#! manifest.yaml
```

下面的示例中，模板会遍历 `compose.yaml` 中定义的所有服务；对每个服务，生成一份专属的 Kubernetes 清单文件，文件名由服务名派生，并包含指定的配置。

```yaml
{{ range $name, $service := .services }}
---
#! {{ $name }}-manifest.yaml
# 生成的代码，请勿编辑
key: value
## ...
{{ end }}
```

### 输入

你可以通过运行 `docker compose config` 生成输入模型。该规范化的 YAML 输出会作为 Compose Bridge 转换的输入。在模板中，使用点号（dot notation）访问 `compose.yaml` 中的数据，可方便地访问嵌套结构。例如，要访问某个服务的部署模式，可使用 `service.deploy.mode`：

 ```yaml
# 遍历 YAML 序列
{{ range $name, $service := .services }}
  # 使用点号访问嵌套属性
  {{ if eq $service.deploy.mode "global" }}
kind: DaemonSet
  {{ end }}
{{ end }}
```

你可以查阅 [Compose 规范的 JSON 模式](https://github.com/compose-spec/compose-go/blob/main/schema/compose-spec.json)，以全面了解 Compose 模型。该模式描述了模型中的所有可能配置及其数据类型。 

### 辅助函数

作为 Go 模板语法的一部分，Compose Bridge 提供了一组 YAML 辅助函数，用于在模板中高效处理数据：

- `seconds`：将[时长](/reference/compose-file/extension.md#specifying-durations)转换为整数
- `uppercase`：将字符串转为大写
- `title`：将字符串的每个单词首字母大写
- `safe`：将字符串转换为安全的标识符（除小写 a-z 外的字符均替换为 `-`）
- `truncate`：移除列表开头的 N 个元素
- `join`：使用分隔符将列表元素连接为单个字符串
- `base64`：将字符串编码为 base64（Kubernetes 中用于编码 Secret）
- `map`：根据 `"value -> newValue"` 映射字符串转换值 
- `indent`：将字符串内容缩进 N 个空格
- `helmValue`：将字符串内容作为模板值写入最终文件

下面示例中，模板会检查服务是否设置了健康检查间隔；若存在，则使用 `seconds` 函数将其转换为秒，并赋给 `periodSeconds` 属性。

```yaml
{{ if $service.healthcheck.interval }}
            periodSeconds: {{ $service.healthcheck.interval | seconds }}{{ end }}
{{ end }}
```

## 自定义

Kubernetes 是一个高度灵活的平台，Compose 概念到 Kubernetes 资源定义的映射方式并不唯一。Compose Bridge 允许你根据自身基础设施的决策与偏好，对转换进行不同程度的定制。

### 修改默认模板

你可以通过运行 `docker compose bridge transformations create --from docker/compose-bridge-kubernetes my-template` 提取默认转换 `docker/compose-bridge-kubernetes` 所使用的模板，随后按需调整。

模板会被提取到以模板名命名的目录中（此处即 `my-template`）。  
该目录包含用于分发模板的 Dockerfile，以及存放模板文件的子目录。  
你可以自由地编辑、删除已有文件，或[新增模板](#add-your-own-templates)，以生成满足需求的 Kubernetes 清单。  
随后，使用生成的 Dockerfile 将改动打包为新的转换镜像，供 Compose Bridge 使用：

```console
$ docker build --tag mycompany/transform --push .
```

接下来即可将该转换用作替代：

```console
$ docker compose bridge convert --transformations mycompany/transform 
```

### 添加自定义模板

对于不在默认转换覆盖范围内的资源，你可以自行编写模板。`compose.yaml` 模型可能不包含生成目标清单所需的全部配置属性；此时可以借助 Compose 自定义扩展来更完整地描述应用，从而实现与平台无关的转换。

例如，如果在 `compose.yaml` 的服务定义中添加了 `x-virtual-host` 元数据，可以使用如下自定义属性生成 Ingress 规则：

```yaml
{{ $project := .name }}
#! {{ $name }}-ingress.yaml
# 生成的代码，请勿编辑
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: virtual-host-ingress
  namespace: {{ $project }}
spec:
  rules:  
{{ range $name, $service := .services }}
{{ range index $service "x-virtual-host" }}
  - host: ${{ . }}
    http:
      paths:
      - path: "/"
        backend:
          service:
            name: ${{ name }}
            port:
              number: 80  
{{ end }}
{{ end }}
```

将模板打包为 Docker 镜像后，你就可以在进行 Compose 到 Kubernetes 的转换时，连同其它转换一起使用该自定义模板：

```console
$ docker compose bridge convert \
    --transformation docker/compose-bridge-kubernetes \
    --transformation mycompany/transform 
```

### 构建自定义转换

虽然通过模板可以以较小的成本完成定制，但在某些场景下，你可能需要进行更大幅度的修改，或复用已有的转换工具。

Compose Bridge 的转换是一个 Docker 镜像，它从 `/in/compose.yaml` 读取 Compose 模型，并在 `/out` 目录生成目标平台的清单。基于这一定义，你可以很容易地将其它工具封装为替代转换，例如使用 [Kompose](https://kompose.io/)：

```Dockerfile
FROM alpine

# 从 GitHub 发布页获取 kompose
RUN apk add --no-cache curl
ARG VERSION=1.32.0
RUN ARCH=$(uname -m | sed 's/armv7l/arm/g' | sed 's/aarch64/arm64/g' | sed 's/x86_64/amd64/g') && \
    curl -fsL \
    "https://github.com/kubernetes/kompose/releases/download/v${VERSION}/kompose-linux-${ARCH}" \
    -o /usr/bin/kompose
RUN chmod +x /usr/bin/kompose

CMD ["/usr/bin/kompose", "convert", "-f", "/in/compose.yaml", "--out", "/out"]
```

该 Dockerfile 打包了 Kompose，并根据 Compose Bridge 的转换约定定义了运行命令。
