---
title: Dockerfile 概览
weight: 20
description: 了解 Dockerfile，以及如何将其与镜像配合用于构建与打包软件
keywords: build, buildx, buildkit, getting started, dockerfile
aliases:
- /build/hellobuild/
- /build/building/packaging/
---

<!-- vale Docker.We = NO -->

## Dockerfile

一切从 Dockerfile 开始。

Docker 通过读取 Dockerfile 中的指令来构建镜像。Dockerfile 是一个包含构建指令的文本文件，用于描述如何从源码构建你的应用。Dockerfile 指令语法由规范定义，详见 [Dockerfile 参考](/reference/dockerfile.md)。

以下是最常见的指令类型：

| 指令                                                      | 说明                                                                                                                                                                                                     |
|-----------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [`FROM <image>`](/reference/dockerfile.md#from)           | 为你的镜像定义基础镜像。                                                                                                                                                                                 |
| [`RUN <command>`](/reference/dockerfile.md#run)           | 在当前镜像之上创建新的一层执行命令并提交结果。`RUN` 也支持以 shell 形式运行命令。                                                                                                                         |
| [`WORKDIR <directory>`](/reference/dockerfile.md#workdir) | 为其后的 `RUN`、`CMD`、`ENTRYPOINT`、`COPY` 与 `ADD` 指令设置工作目录。                                                                                                                                    |
| [`COPY <src> <dest>`](/reference/dockerfile.md#copy)      | 将 `<src>` 中的新文件或目录复制到容器文件系统的 `<dest>` 路径。                                                                                                                                            |
| [`CMD <command>`](/reference/dockerfile.md#cmd)           | 定义基于该镜像启动容器时默认运行的程序。每个 Dockerfile 只能有一个 `CMD`；如果存在多个定义，仅最后一个会生效。                                                                                             |

Dockerfile 是构建镜像的关键输入，可基于你的配置实现自动化、分层的镜像构建。Dockerfile 可以从简单开始，随着需求增加逐步演进以支持更复杂的场景。

### Filename

Dockerfile 的默认文件名为 `Dockerfile`（无扩展名）。使用默认文件名可以在运行 `docker build` 时无需额外指定相关参数。

某些项目可能需要为特定目的准备不同的 Dockerfile。常见约定是命名为 `<something>.Dockerfile`。你可以在 `docker build` 命令中通过 `--file` 选项指定 Dockerfile 文件名。关于 `--file` 的更多信息，参见
[`docker build` CLI 参考](/reference/cli/docker/buildx/build.md#file)。

> [!NOTE]
>
> 建议你的项目主 Dockerfile 使用默认文件名（`Dockerfile`）。

## Docker images

Docker 镜像由多层组成。每一层对应 Dockerfile 中的一条构建指令。各层按顺序堆叠，每层都是相对于前一层的变更（delta）。

### Example

下面展示一个使用 Docker 构建应用的典型工作流。

以下示例代码是一个使用 Flask 框架编写的简易 Python“Hello World”应用：

```python
from flask import Flask
app = Flask(__name__)

@app.route("/")
def hello():
    return "Hello World!"
```

如果不使用 Docker Build 来交付和部署该应用，你需要确保：

- 服务器已安装所需的运行时依赖
- 将 Python 代码上传到服务器的文件系统
- 服务器以正确参数启动你的应用

下面的 Dockerfile 会创建一个容器镜像，预装所有依赖，并可自动启动你的应用：

```dockerfile
# syntax=docker/dockerfile:1
FROM ubuntu:22.04

# install app dependencies
RUN apt-get update && apt-get install -y python3 python3-pip
RUN pip install flask==3.0.*

# install app
COPY hello.py /

# final configuration
ENV FLASK_APP=hello
EXPOSE 8000
CMD ["flask", "run", "--host", "0.0.0.0", "--port", "8000"]
```

下面按模块解析这个 Dockerfile 的作用：

- [Dockerfile 语法](#dockerfile-syntax)
- [基础镜像](#base-image)
- [环境准备](#environment-setup)
- [注释](#comments)
- [安装依赖](#installing-dependencies)
- [复制文件](#copying-files)
- [设置环境变量](#setting-environment-variables)
- [暴露端口](#exposed-ports)
- [启动应用](#starting-the-application)

### Dockerfile syntax

Dockerfile 的第一行通常是 [`# syntax` 解析指令](/reference/dockerfile.md#syntax)。
该指令虽为可选，但用于告知构建器在解析 Dockerfile 时采用的语法；同时也允许启用 [BuildKit](../buildkit/_index.md#getting-started) 的旧版 Docker 在构建开始前选择特定的 [Dockerfile 前端](../buildkit/frontend.md)。
[解析指令](/reference/dockerfile.md#parser-directives) 必须出现在任何注释、空白或 Dockerfile 指令之前，且应当是 Dockerfile 的第一行。

```dockerfile
# syntax=docker/dockerfile:1
```

> [!TIP]
>
> 建议使用 `docker/dockerfile:1`，它始终指向 v1 语法的最新发布版本。BuildKit 会在构建前自动检查语法更新，确保你使用的是最新版本。

### Base image

语法指令之后的一行用于指定要使用的基础镜像：

```dockerfile
FROM ubuntu:22.04
```

[`FROM` 指令](/reference/dockerfile.md#from) 将基础镜像设置为 Ubuntu 22.04。后续的所有指令都在该基础镜像（Ubuntu 环境）中执行。
`ubuntu:22.04` 使用了镜像命名的 `name:tag` 约定。在构建镜像时，你同样会用这种方式为镜像命名。你可以在项目中通过 Dockerfile 的 `FROM` 指令引入大量公共镜像，以此复用其能力。

[Docker Hub](https://hub.docker.com/search?image_filter=official&q=&type=image) 提供了数量可观的官方镜像，供你直接使用。

### Environment setup

下面这行会在基础镜像中执行一个构建命令：

```dockerfile
# install app dependencies
RUN apt-get update && apt-get install -y python3 python3-pip
```

该 [`RUN` 指令](/reference/dockerfile.md#run) 会在 Ubuntu 中启动一个 shell，更新 APT 包索引，并在容器内安装所需的 Python 工具。

### Comments

注意 `# install app dependencies` 这一行。这是一条注释。Dockerfile 中的注释以 `#` 开头。随着 Dockerfile 的演进，合理的注释有助于为后续阅读与维护（包括未来的你）记录其工作方式。

> [!NOTE]
>
> 你可能注意到，注释与文件第一行的[语法指令](#dockerfile-syntax)都使用了相同的符号 `#`。只有当模式匹配指令且出现在 Dockerfile 开头时，才会被解析为指令；否则会被视为普通注释。

### Installing dependencies

第二条 `RUN` 指令安装该 Python 应用所需的 `flask` 依赖：

```dockerfile
RUN pip install flask==3.0.*
```

该指令的前提是构建容器中已安装 `pip`。第一条 `RUN` 命令已完成 `pip` 的安装，保证我们可以使用它来安装 Flask Web 框架。

### Copying files

接下来使用 [`COPY` 指令](/reference/dockerfile.md#copy) 将本地构建上下文中的 `hello.py` 文件复制到镜像的根目录：

```dockerfile
COPY hello.py /
```

[构建上下文](./context.md) 是指你在 Dockerfile 指令（如 `COPY`、`ADD`）中可以访问的文件集合。

执行 `COPY` 后，`hello.py` 会被添加到构建容器的文件系统中。

### Setting environment variables

如果你的应用使用环境变量，可以通过 [`ENV` 指令](/reference/dockerfile.md#env) 在构建过程中设置它们：

```dockerfile
ENV FLASK_APP=hello
```

这会设置一个稍后需要的 Linux 环境变量。示例中使用的 Flask 框架会依赖该变量来启动应用。没有它，Flask 将无法找到并运行我们的应用。

### Exposed ports

[`EXPOSE` 指令](/reference/dockerfile.md#expose) 用于声明最终镜像有一个服务监听在 `8000` 端口。

```dockerfile
EXPOSE 8000
```

该指令并非必需，但属于良好实践，有助于工具与团队成员理解应用行为。

### Starting the application

最后，[`CMD` 指令](/reference/dockerfile.md#cmd) 用于设置基于该镜像启动容器时执行的命令：

```dockerfile
CMD ["flask", "run", "--host", "0.0.0.0", "--port", "8000"]
```

上述命令会启动 Flask 开发服务器，监听在所有地址的 `8000` 端口。这里使用的是 `CMD` 的“exec 形式”。你也可以使用“shell 形式”：

```dockerfile
CMD flask run --host 0.0.0.0 --port 8000
```

两种形式之间存在一些细微差异，例如处理 `SIGTERM`、`SIGKILL` 等信号的方式。更多差异说明，参见
[Shell 与 exec 形式](/reference/dockerfile.md#shell-and-exec-form)。

## Building

要基于[上一节](#example)的 Dockerfile 构建容器镜像，请使用 `docker build` 命令：

```console
$ docker build -t test:latest .
```

选项 `-t test:latest` 用于指定镜像的名称与标签。

命令末尾的一个点（`.`）将[构建上下文](./context.md)指定为当前目录。这意味着构建会在执行命令的目录中查找 Dockerfile 与 `hello.py` 文件；如果它们不存在，构建会失败。

镜像构建完成后，你可以使用 `docker run` 并指定镜像名称，以容器的形式运行该应用：

```console
$ docker run -p 127.0.0.1:8000:8000 test:latest
```

这会将容器的 8000 端口映射到 Docker 主机上的 `http://localhost:8000`。

> [!TIP]
>
> 如果你在 Visual Studio Code 中编写 Dockerfile，想改进语法检查、代码导航与漏洞扫描能力，参见 [Docker VS Code 扩展](https://marketplace.visualstudio.com/items?itemName=docker.docker)。
