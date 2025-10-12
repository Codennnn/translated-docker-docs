---
title: 使用默认的 Compose Bridge 转换
linkTitle: 用法
weight: 10
description: 了解如何使用默认的 Compose Bridge 转换，将 Compose 文件转换为 Kubernetes 清单
keywords: docker compose bridge, compose kubernetes transform, kubernetes from compose, compose bridge convert, compose.yaml to kubernetes
---

{{< summary-bar feature_name="Compose bridge" >}}

Compose Bridge 为你的 Compose 配置文件提供了开箱即用的转换能力。基于任意 `compose.yaml`，Compose Bridge 会生成：

- 一个 [Namespace](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)，用于隔离你的资源，避免与其他部署的资源冲突。
- 一个 [ConfigMap](https://kubernetes.io/docs/concepts/configuration/configmap/)，为 Compose 应用中的每个 [config](/reference/compose-file/configs.md) 资源生成对应条目。
- 为应用服务生成 [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)，确保在 Kubernetes 集群中维持指定数量的实例。
- 为服务暴露的端口生成 [Services](https://kubernetes.io/docs/concepts/services-networking/service/)，用于服务间通信。
- 为服务发布的端口生成 [Services](https://kubernetes.io/docs/concepts/services-networking/service/)，类型为 `LoadBalancer`，以便 Docker Desktop 在主机上映射相同端口。
- 根据 `compose.yaml` 中定义的网络拓扑生成 [Network policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)。 
- 为你的卷生成 [PersistentVolumeClaims](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)，使用 `hostpath` 存储类，由 Docker Desktop 管理卷创建。
- 生成包含机密信息的 [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)，用于本地测试环境。

同时还会生成一个专用于 Docker Desktop 的 Kustomize overlay，其中包含：
 - 为需要在主机上暴露端口的服务提供 `LoadBalancer`。
 - 一个 `PersistentVolumeClaim`，使用 Docker Desktop 的存储供应器 `desktop-storage-provisioner` 更高效地完成卷的供应。
 - 一个 Kustomize 配置文件，用于将所有资源关联在一起。

## 使用默认的 Compose Bridge 转换

运行以下命令以使用默认转换：

```console
$ docker compose bridge convert
```

Compose 会在当前目录查找 `compose.yaml` 文件并执行转换。

成功后，Compose Bridge 会生成 Kubernetes 清单，并输出类似如下的日志：

```console
$ docker compose bridge convert -f compose.yaml 
Kubernetes resource api-deployment.yaml created
Kubernetes resource db-deployment.yaml created
Kubernetes resource web-deployment.yaml created
Kubernetes resource api-expose.yaml created
Kubernetes resource db-expose.yaml created
Kubernetes resource web-expose.yaml created
Kubernetes resource 0-avatars-namespace.yaml created
Kubernetes resource default-network-policy.yaml created
Kubernetes resource private-network-policy.yaml created
Kubernetes resource public-network-policy.yaml created
Kubernetes resource db-db_data-persistentVolumeClaim.yaml created
Kubernetes resource api-service.yaml created
Kubernetes resource web-service.yaml created
Kubernetes resource kustomization.yaml created
Kubernetes resource db-db_data-persistentVolumeClaim.yaml created
Kubernetes resource api-service.yaml created
Kubernetes resource web-service.yaml created
Kubernetes resource kustomization.yaml created
```

生成的文件会保存在项目的 `/out` 目录下。 

随后，可以使用标准部署命令 `kubectl apply -k out/overlays/desktop/` 在 Kubernetes 上运行应用。

> [!IMPORTANT]
>
> 在部署 Compose Bridge 转换结果之前，请确保你已在 Docker Desktop 中启用 Kubernetes。

如果要转换位于其他目录下的 `compose.yaml` 文件，可以运行：

```console
$ docker compose bridge convert -f <path-to-file>/compose.yaml 
```

要查看所有可用参数，运行：

```console
$ docker compose bridge convert --help
```

> [!TIP]
>
> 你也可以在 Docker Desktop 的 Compose 文件查看器中，将 Compose 项目转换并部署到 Kubernetes 集群。
> 
> 确保已登录 Docker 账户，进入 **Containers** 视图找到你的容器，点击右上角 **View configurations**，然后选择 **Convert and Deploy to Kubernetes**。

## 下一步

- [了解如何自定义 Compose Bridge](customize.md)