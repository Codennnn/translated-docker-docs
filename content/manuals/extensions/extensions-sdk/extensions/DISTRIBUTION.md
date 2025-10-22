---
title: 打包和发布扩展
description: Docker 扩展分发
keywords: Docker, extensions, sdk, distribution
aliases: 
 - /desktop/extensions-sdk/extensions/DISTRIBUTION/
weight: 30
---

本页面包含有关如何打包和分发扩展的额外信息。

## 打包扩展

Docker 扩展被打包为 Docker 镜像。整个扩展运行时，包括 UI、后端服务（主机或虚拟机）以及任何必要的二进制文件，都必须包含在扩展镜像中。
每个扩展镜像必须在其文件系统根目录中包含一个 `metadata.json` 文件，该文件定义了[扩展的内容](../architecture/metadata.md)。

Docker 镜像必须具有多个[镜像标签](labels.md)，提供有关扩展的信息。了解如何使用[扩展标签](labels.md)提供扩展概述信息。

要打包和发布扩展，您必须构建一个 Docker 镜像（`docker build`），并将镜像推送到 [Docker Hub](https://hub.docker.com/)（`docker push`），使用特定标签来管理扩展的版本。

## 发布扩展

Docker 镜像标签必须遵循 semver 约定，以便能够获取扩展的最新版本，并了解是否有可用的更新。请参阅 [semver.org](https://semver.org/) 了解有关语义版本控制的更多信息。

扩展镜像必须是多架构镜像，以便用户可以在 ARM/AMD 硬件上安装扩展。这些多架构镜像可以包含特定于 ARM/AMD 的二进制文件。Mac 用户将根据其架构自动使用正确的镜像。
在主机上安装二进制文件的扩展还必须在同一扩展镜像中提供 Windows 二进制文件。了解如何为您的扩展[构建多架构镜像](multi-arch.md)。

您可以在没有任何代码仓库约束的情况下实现扩展。Docker 不需要访问代码仓库即可使用扩展。此外，您可以管理扩展的新版本，而无需依赖 Docker Desktop 的发布。

## 新版本和更新

您可以通过向 Docker Hub 推送带有新标签的新镜像来发布 Docker 扩展的新版本。

推送到与扩展对应的镜像仓库的任何新镜像都会定义该扩展的新版本。镜像标签用于标识版本号。扩展版本必须遵循 semver，以便于理解和比较版本。

Docker Desktop 会扫描市场中发布的扩展列表以查找新版本，并在用户可以升级特定扩展时提供通知。目前，不属于市场的扩展没有自动更新通知。

用户可以在不更新 Docker Desktop 本身的情况下下载和安装任何扩展的较新版本。

## 扩展 API 依赖

扩展必须指定它们依赖的扩展 API 版本。Docker Desktop 会检查扩展所需的版本，并且只建议安装与当前安装的 Docker Desktop 版本兼容的扩展。用户可能需要更新 Docker Desktop 才能安装最新的可用扩展。

扩展镜像标签必须指定扩展所依赖的 API 版本。这允许 Docker Desktop 在无需预先下载完整扩展镜像的情况下检查扩展镜像的较新版本。

## 扩展和扩展 SDK 的许可证

[Docker 扩展 SDK](https://www.npmjs.com/package/@docker/extension-api-client) 采用 Apache 2.0 许可证授权，可免费使用。

对于每个扩展应如何获得许可没有约束，这取决于您在创建新扩展时的决定。
