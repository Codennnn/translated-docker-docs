---
title: 使用环境变量配置 Docker Scout
linkTitle: Docker Scout 环境变量
description: 使用以下环境变量配置 Docker Scout CLI 命令的行为
keywords: scout, supply chain, cli, environment, variables, env, vars, configure
aliases:
  - /scout/env-vars/
---

以下环境变量可用于配置 Docker Scout CLI 命令，以及对应的 `docker/scout-cli` 容器镜像：

| 名称                                    | 类型    | 说明                                                                                        |
| :-------------------------------------- | ------- | :------------------------------------------------------------------------------------------ |
| DOCKER_SCOUT_CACHE_FORMAT               | String  | 本地镜像缓存格式；可选 `oci` 或 `tar`（默认：`oci`）                                         |
| DOCKER_SCOUT_CACHE_DIR                  | String  | 本地 SBOM 缓存的存储目录（默认：`$HOME/.docker/scout`）                                     |
| DOCKER_SCOUT_NO_CACHE                   | Boolean | 设为 `true` 时禁用本地 SBOM 缓存                                                             |
| DOCKER_SCOUT_OFFLINE                    | Boolean | 在索引 SBOM 时启用[离线模式](#offline-mode)                                                 |
| DOCKER_SCOUT_REGISTRY_TOKEN             | String  | 从仓库拉取镜像时用于认证的令牌                                                              |
| DOCKER_SCOUT_REGISTRY_USER              | String  | 从仓库拉取镜像时用于认证的用户名                                                            |
| DOCKER_SCOUT_REGISTRY_PASSWORD          | String  | 从仓库拉取镜像时用于认证的密码或个人访问令牌                                                |
| DOCKER_SCOUT_HUB_USER                   | String  | 用于认证 Docker Scout 后端的 Docker Hub 用户名                                              |
| DOCKER_SCOUT_HUB_PASSWORD               | String  | 用于认证 Docker Scout 后端的 Docker Hub 密码或个人访问令牌                                  |
| DOCKER_SCOUT_NEW_VERSION_WARN           | Boolean | 提醒是否有可用的 Docker Scout CLI 新版本                                                    |
| DOCKER_SCOUT_EXPERIMENTAL_WARN          | Boolean | 针对实验性功能的提示                                                                        |
| DOCKER_SCOUT_EXPERIMENTAL_POLICY_OUTPUT | Boolean | 关闭策略评估的实验性输出                                                                    |

## 离线模式 {#offline-mode}

在正常工作模式下，Docker Scout 会交叉引用外部系统（如 npm、NuGet、proxy.golang.org），以获取镜像中软件包的补充信息。

当将 `DOCKER_SCOUT_OFFLINE` 设为 `true` 时，镜像分析将以离线模式运行。离线模式下，Docker Scout 不会向外部系统发起任何出站请求。

启用离线模式：

```console
$ export DOCKER_SCOUT_OFFLINE=true
```
