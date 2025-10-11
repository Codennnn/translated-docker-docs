---
title: 基础镜像
weight: 80
description: 了解基础镜像及其创建方式
keywords: images, base image, examples
aliases:
- /articles/baseimages/
- /engine/articles/baseimages/
- /engine/userguide/eng-image/baseimages/
- /develop/develop-images/baseimages/
---

所有 Dockerfile 都从一个基础镜像开始。
所谓“基础”，指的是你的镜像所继承的那个镜像。
它对应 Dockerfile 中 `FROM` 指令所指定的内容。

```dockerfile
FROM debian
```

在大多数情况下，你不需要自己创建基础镜像。Docker Hub 提供了大量可直接作为基础镜像使用的镜像库。[Docker 官方镜像](../../docker-hub/image-library/trusted-content.md#docker-official-images) 文档清晰、倡导最佳实践，并且会定期更新。除此之外，还有由可信发布合作伙伴创建、并经 Docker 验证的 [Docker 认证发布者镜像](../../docker-hub/image-library/trusted-content.md#verified-publisher-images)。

## 创建基础镜像

如果你需要完全掌控镜像内容，可以基于你选择的某个 Linux 发行版创建自定义基础镜像，或使用特殊的 `FROM scratch` 作为基础：

```dockerfile
FROM scratch
```

`scratch` 镜像通常用于创建仅包含应用所需内容的最小化镜像。参见[使用 scratch 创建最小化基础镜像](#create-a-minimal-base-image-using-scratch)。

若要创建发行版基础镜像，你可以使用一个打包为 `tar` 文件的根文件系统，并通过 `docker import` 将其导入到 Docker 中。如何创建取决于你要打包的 Linux 发行版。参见[使用 tar 创建完整镜像](#create-a-full-image-using-tar)。

## 使用 scratch 创建最小化基础镜像 {#create-a-minimal-base-image-using-scratch}

保留字、最小化的 `scratch` 镜像可作为构建容器的起点。使用 `scratch` 表示你希望 Dockerfile 中的下一条命令成为镜像的首个文件系统层。

尽管 `scratch` 会出现在 Docker 的 [Docker Hub 仓库](https://hub.docker.com/_/scratch) 页面上，但你无法拉取、运行它，或用 `scratch` 为任何镜像打标签。你只能在 `Dockerfile` 中引用它。例如，使用 `scratch` 创建一个最小容器：

```dockerfile
# syntax=docker/dockerfile:1
FROM scratch
ADD hello /
CMD ["/hello"]
```

假设在[构建上下文](/manuals/build/concepts/context.md)的根目录存在一个名为 `hello` 的可执行二进制文件。你可以使用如下 `docker build` 命令构建该镜像：

```console
$ docker build --tag hello .
```

要运行新镜像，使用 `docker run` 命令：

```console
$ docker run --rm hello
```

只有当 `hello` 二进制在运行时不存在任何依赖时，这个示例镜像才能成功执行。通常程序会依赖运行时环境中的某些程序或资源，例如：

- 编程语言运行时
- 动态链接的 C 库
- CA 证书

在构建基础镜像（或任意镜像）时，这是一个需要重点考虑的方面。这也解释了为何除非是小而简单的程序，否则使用 `FROM scratch` 创建基础镜像会比较困难。另一方面，将镜像内容控制在必要范围内同样重要，这有助于减小镜像体积并降低攻击面。

## 使用 tar 创建完整镜像 {#create-a-full-image-using-tar}

一般而言，你可以从一台正在运行目标发行版的工作主机开始，将其打包为基础镜像。当然，也并非总是需要一台现成的系统，例如可以使用 Debian 的 [Debootstrap](https://wiki.debian.org/Debootstrap) 等工具来构建 Ubuntu 镜像。

例如，创建一个 Ubuntu 基础镜像：

```dockerfile
$ sudo debootstrap noble noble > /dev/null
$ sudo tar -C noble -c . | docker import - noble

sha256:81ec9a55a92a5618161f68ae691d092bf14d700129093158297b3d01593f4ee3

$ docker run noble cat /etc/lsb-release

DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=24.04
DISTRIB_CODENAME=noble
DISTRIB_DESCRIPTION="Ubuntu 24.04.2 LTS"
```

在 [Moby GitHub 仓库](https://github.com/moby/moby/blob/master/contrib) 中可以找到更多用于创建基础镜像的示例脚本。

## 更多资源

关于构建镜像与编写 Dockerfile 的更多信息，参见：

* [Dockerfile 参考](/reference/dockerfile.md)
* [Dockerfile 最佳实践](/manuals/build/building/best-practices.md)
* [Docker 官方镜像](../../docker-hub/image-library/trusted-content.md#docker-official-images)
