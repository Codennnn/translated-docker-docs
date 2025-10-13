---
description: 了解如何使用日志驱动插件以扩展与自定义 Docker 的日志能力
title: 使用日志驱动插件
keywords: 日志, 驱动, 插件, 监控
aliases:
  - /engine/admin/logging/plugins/
  - /engine/reference/logging/plugins/
  - /config/containers/logging/plugins/
---

Docker 的日志插件可以让你在[内置日志驱动](configure.md)的基础上进一步扩展与自定义日志能力。
日志服务提供商可以[实现自有插件](/manuals/engine/extend/plugins_logging.md)，并将其发布到 Docker Hub 或私有仓库。本文介绍作为该日志服务的使用者，如何配置 Docker 来使用该插件。

## 安装日志驱动插件

根据插件开发者提供的信息，使用 `docker plugin install <org/image>` 安装日志驱动插件。

使用 `docker plugin ls` 可以列出已安装的所有插件；使用 `docker inspect` 可以查看指定插件的详细信息。

## 将插件配置为默认日志驱动

插件安装完成后，可在 `daemon.json` 中将插件名称设为 `log-driver` 的值，使其成为默认日志驱动。具体步骤参见[日志概览](configure.md#configure-the-default-logging-driver)。若该日志驱动支持额外选项，可在同一文件的 `log-opts` 中进行配置。

## 为容器配置使用该插件作为日志驱动

插件安装后，可在 `docker run` 中通过 `--log-driver` 指定容器使用该插件作为日志驱动，详见[日志概览](configure.md#configure-the-logging-driver-for-a-container)。若日志驱动支持额外选项，可使用一个或多个 `--log-opt` 标志指定键值对参数。
