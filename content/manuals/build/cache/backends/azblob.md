---
title: Azure Blob 存储缓存
description: 使用 Azure Blob 存储管理构建缓存
keywords: build, buildx, cache, backend, azblob, azure
aliases:
  - /build/building/cache/backends/azblob/
---

{{< summary-bar feature_name="Azure blob" >}}

`azblob` 缓存存储会将你的构建缓存上传至
[Azure 的 Blob 存储服务](https://azure.microsoft.com/en-us/services/storage/blobs/)。

默认的 `docker` 驱动不支持该缓存存储后端。
如需使用此功能，请使用其他驱动创建新的构建器；
详情参见[构建驱动](/manuals/build/builders/drivers/_index.md)。

## 概要

```console
$ docker buildx build --push -t <registry>/<image> \
  --cache-to type=azblob,name=<cache-image>[,parameters...] \
  --cache-from type=azblob,name=<cache-image>[,parameters...] .
```

下表描述了可通过 `--cache-to` 与 `--cache-from` 传入的逗号分隔参数（CSV）：

| 名称                | 选项                     | 类型        | 默认值  | 说明                                               |
| ------------------- | ------------------------ | ----------- | ------- | -------------------------------------------------- |
| `name`              | `cache-to`,`cache-from`  | String      |         | 必填。缓存镜像的名称。                             |
| `account_url`       | `cache-to`,`cache-from`  | String      |         | 存储账户的基础 URL。                               |
| `secret_access_key` | `cache-to`,`cache-from`  | String      |         | Blob 存储账号密钥，见[认证][1]。                   |
| `mode`              | `cache-to`               | `min`,`max` | `min`   | 导出的缓存层范围，见[缓存模式][2]。               |
| `ignore-error`      | `cache-to`               | Boolean     | `false` | 忽略因缓存导出失败引发的错误。                     |

[1]: #authentication
[2]: _index.md#cache-mode

## 认证 {#authentication}

若未显式指定 `secret_access_key`，BuildKit 服务器会遵循
[Azure Go SDK 的认证方案](https://docs.microsoft.com/en-us/azure/developer/go/azure-sdk-authentication)
从其环境变量中读取该值。注意，这些环境变量读取自服务器端，而非 Buildx 客户端。

## 延伸阅读

缓存入门请参阅《[Docker 构建缓存](../_index.md)》。

关于 `azblob` 缓存后端的更多信息，请参阅
[BuildKit README](https://github.com/moby/buildkit#azure-blob-storage-cache-experimental)。
