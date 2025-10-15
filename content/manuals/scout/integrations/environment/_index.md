---
description:
  Docker Scout 可以与运行时环境集成，为您的软件供应链提供实时洞察。
keywords: supply chain, security, streams, environments, workloads, deployments
title: 将 Docker Scout 集成到环境中
---

您可以将 Docker Scout 与运行时环境集成，获取正在运行的工作负载的洞察，从而实时掌握已部署制品的安全状态。

Docker Scout 允许您定义多个环境，并将镜像分配到不同环境。这样可以全局审视软件供应链，并对比不同环境（例如 staging 与 production）之间的差异。

如何定义与命名环境完全由您决定，只需符合团队的语义与交付方式即可。

## 分配到环境

每个环境都会包含若干镜像引用，这些引用代表该环境中当前正在运行的容器。

例如：如果生产环境正在运行 `myorg/webapp:3.1`，您可以把该标签分配给 `production` 环境；而在 staging 环境中可能运行的是该镜像的另一个版本，此时就将对应版本分配到 `staging` 环境。

向 Docker Scout 添加环境的方式：

- 使用 `docker scout env <environment> <image>` CLI 命令，手动将镜像记录到环境
- 启用运行时集成，自动发现环境中的镜像

Docker Scout 支持以下运行时集成：

- [Docker Scout GitHub Action](https://github.com/marketplace/actions/docker-scout#record-an-image-deployed-to-an-environment)
- [CLI 客户端](./cli.md)
- [Sysdig 集成](./sysdig.md)

> [!NOTE]
>
> 只有组织所有者可以创建新环境并配置集成。
> 另外，仅当镜像已被[分析](/manuals/scout/explore/analysis.md)（手动或通过[仓库集成](/manuals/scout/integrations/_index.md#container-registries)）时，Docker Scout 才会将其分配到环境。

## 列出环境

要查看某个组织的全部可用环境，可使用 `docker scout env` 命令：

```console
$ docker scout env
```

默认情况下，该命令会列出您个人 Docker 组织下的所有环境。若需查看您参与的其他组织的环境，请使用 `--org` 选项：

```console
$ docker scout env --org <org>
```

您可以通过 `docker scout config` 命令更改默认组织。该设置会影响所有 `docker scout` 命令（不仅限于 `env`）。

```console
$ docker scout config organization <org>
```

## 跨环境对比

将镜像分配到环境后，您可以进行环境内与跨环境的对比。例如在 GitHub Pull Request 中，将 PR 构建出的镜像与 staging 或 production 中的对应镜像进行比较。

您也可以在 [`docker scout compare`](/reference/cli/docker/scout/compare.md) 命令中使用 `--to-env` 选项与某个环境进行对比：

```console
$ docker scout compare --to-env production myorg/webapp:latest
```

## 查看某个环境的镜像

查看环境中的镜像：

1. 在 Docker Scout 控制台打开 [Images 页面](https://scout.docker.com/)。
2. 打开 **Environments** 下拉菜单。
3. 选择要查看的环境。

列表会显示已分配到所选环境的所有镜像。如果同一镜像在该环境中部署了多个版本，列表中会显示所有这些版本。

或者，您也可以在终端使用 `docker scout env` 命令查看：

```console
$ docker scout env production
docker/scout-demo-service:main@sha256:ef08dca54c4f371e7ea090914f503982e890ec81d22fd29aa3b012351a44e1bc
```

### 标签与摘要不匹配（Mismatching image tags）

当您在 **Images** 选项卡中选择了某个环境时，列表中的标签代表用于部署该镜像的标签。标签是可变的，即同一个标签可指向不同的镜像摘要（digest）。如果 Docker Scout 发现标签指向了过期的摘要，会在镜像名称旁显示警告图标。
