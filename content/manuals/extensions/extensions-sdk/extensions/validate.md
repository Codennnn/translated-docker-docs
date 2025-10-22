---
title: 验证扩展
linkTitle: 验证
description: 扩展创建流程的第三步
keywords: Docker, Extensions, sdk, validate, install
aliases:
 - /desktop/extensions-sdk/extensions/validation/
 - /desktop/extensions-sdk/build/build-install/
 - /desktop/extensions-sdk/dev/cli/build-test-install-extension/
 - /desktop/extensions-sdk/extensions/validate/
weight: 20
---

在共享或发布扩展之前，请先验证扩展。验证扩展可以确保：

- 扩展已构建包含在市场中正确显示所需的[镜像标签](labels.md)
- 扩展能够正常安装和运行

扩展 CLI 允许你在本地安装和运行扩展之前对其进行验证。

验证过程会检查扩展的 `Dockerfile` 是否指定了所有必需标签，以及元数据文件是否符合 JSON 模式文件的要求。

要执行验证，请运行：

```console
$ docker extension validate <你的扩展名称>
```

如果你的扩展有效，将显示以下消息：

```console
The extension image "name-of-your-extension" is valid
```

在构建镜像之前，也可以仅验证 `metadata.json` 文件：

```console
$ docker extension validate /path/to/metadata.json
```

用于验证 `metadata.json` 文件的 JSON 模式可以在[发布页面](https://github.com/docker/extensions-sdk/releases/latest)找到。
