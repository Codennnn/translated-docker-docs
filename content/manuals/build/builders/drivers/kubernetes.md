---
title: Kubernetes 驱动
description: |
  Kubernetes 驱动允许你在 Kubernetes 集群中运行 BuildKit。
  你可以通过 Buildx 连接到集群，并在集群中运行构建。
keywords: build, buildx, driver, builder, kubernetes
aliases:
  - /build/buildx/drivers/kubernetes/
  - /build/building/drivers/kubernetes/
  - /build/drivers/kubernetes/
---

Kubernetes 驱动可以把你的本地开发或 CI 环境连接到 Kubernetes 集群中的构建器，
从而利用更强的计算资源，并可选择使用多种原生架构的节点。

## 概要

运行以下命令以创建一个名为 `kube` 的新构建器，并使用 Kubernetes 驱动：

```console
$ docker buildx create \
  --bootstrap \
  --name=kube \
  --driver=kubernetes \
  --driver-opt=[key=value,...]
```

下表列出了可通过 `--driver-opt` 传入的驱动特定选项：

| 参数                          | 类型         | 默认值                                   | 说明                                                                                                                               |
| ----------------------------- | ------------ | ---------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| `image`                       | String       |                                          | 设置运行 BuildKit 所用的镜像。                                                                                                     |
| `namespace`                   | String       | 当前 K8s 上下文中的命名空间             | 设置 Kubernetes 命名空间。                                                                                                         |
| `default-load`                | Boolean      | `false`                                  | 自动将镜像加载到 Docker Engine 的镜像存储。                                                                                        |
| `replicas`                    | Integer      | 1                                        | 设置要创建的 Pod 副本数。参见[扩缩容 BuildKit][1]                                                                                 |
| `requests.cpu`                | CPU units    |                                          | 以 Kubernetes CPU 单位设置 CPU 请求值。如 `requests.cpu=100m` 或 `requests.cpu=2`                                                  |
| `requests.memory`             | Memory size  |                                          | 以字节或带合法后缀的形式设置内存请求值。如 `requests.memory=500Mi` 或 `requests.memory=4G`                                         |
| `requests.ephemeral-storage`  | Storage size |                                          | 以字节或带合法后缀的形式设置临时存储请求值。如 `requests.ephemeral-storage=2Gi`                                                   |
| `limits.cpu`                  | CPU units    |                                          | 以 Kubernetes CPU 单位设置 CPU 限制值。如 `requests.cpu=100m` 或 `requests.cpu=2`                                                  |
| `limits.memory`               | Memory size  |                                          | 以字节或带合法后缀的形式设置内存限制值。如 `requests.memory=500Mi` 或 `requests.memory=4G`                                         |
| `limits.ephemeral-storage`    | Storage size |                                          | 以字节或带合法后缀的形式设置临时存储限制值。如 `requests.ephemeral-storage=100M`                                                  |
| `buildkit-root-volume-memory` | Memory size  | 使用常规文件系统                         | 将 `/var/lib/buildkit` 挂载到内存支持的 `emptyDir` 卷上，`SizeLimit` 为容量限制。例如：`buildkit-root-folder-memory=6G`            |
| `nodeselector`                | CSV string   |                                          | 设置 Pod 的 `nodeSelector` 标签。参见[节点分配][2]。                                                                               |
| `annotations`                 | CSV string   |                                          | 为 Deployment 与 Pod 设置额外注解。                                                                                                |
| `labels`                      | CSV string   |                                          | 为 Deployment 与 Pod 设置额外标签。                                                                                                |
| `tolerations`                 | CSV string   |                                          | 配置 Pod 的污点容忍（taint toleration）。参见[节点分配][2]。                                                                      |
| `serviceaccount`              | String       |                                          | 设置 Pod 的 `serviceAccountName`。                                                                                                 |
| `schedulername`               | String       |                                          | 设置负责调度该 Pod 的调度器名称。                                                                                                  |
| `timeout`                     | Time         | `120s`                                   | 设置在构建前等待 Pod 准备完成的超时时间。                                                                                          |
| `rootless`                    | Boolean      | `false`                                  | 以非 root 用户运行容器。参见[无特权模式][3]。                                                                                      |
| `loadbalance`                 | String       | `sticky`                                 | 负载均衡策略（`sticky` 或 `random`）。`sticky` 根据上下文路径哈希选择 Pod。                                                        |
| `qemu.install`                | Boolean      | `false`                                  | 安装 QEMU 模拟以支持多平台。参见 [QEMU][4]。                                                                                        |
| `qemu.image`                  | String       | `tonistiigi/binfmt:latest`               | 设置 QEMU 模拟所用镜像。参见 [QEMU][4]。                                                                                            |

[1]: #scaling-buildkit
[2]: #node-assignment
[3]: #rootless-mode
[4]: #qemu

## 扩缩容 BuildKit

Kubernetes 驱动的一大优势是可以按需扩缩构建器副本数，以应对增长的构建负载。
可通过以下驱动选项进行配置：

- `replicas=N`

  将 BuildKit Pod 的副本数扩容到期望规模。默认仅创建 1 个 Pod。增加副本数可以利用集群中的多台节点。

- `requests.cpu`, `requests.memory`, `requests.ephemeral-storage`, `limits.cpu`, `limits.memory`, `limits.ephemeral-storage`

  这些选项允许按 Kubernetes 官方文档为每个 BuildKit Pod 设置资源请求与限制：
  [链接](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)。

例如，创建 4 个副本的 BuildKit Pod：

```console
$ docker buildx create \
  --bootstrap \
  --name=kube \
  --driver=kubernetes \
  --driver-opt=namespace=buildkit,replicas=4
```

列出相关对象，输出类似：

```console
$ kubectl -n buildkit get deployments
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
kube0   4/4     4            4           8s

$ kubectl -n buildkit get pods
NAME                     READY   STATUS    RESTARTS   AGE
kube0-6977cdcb75-48ld2   1/1     Running   0          8s
kube0-6977cdcb75-rkc6b   1/1     Running   0          8s
kube0-6977cdcb75-vb4ks   1/1     Running   0          8s
kube0-6977cdcb75-z4fzs   1/1     Running   0          8s
```

此外，当存在多个副本时，可通过 `loadbalance=(sticky|random)` 控制负载均衡行为。
`random` 从节点池中随机选择节点，使负载更均匀；`sticky`（默认）则尽量将同一构建多次连接到同一节点，以更好利用本地缓存。

更多可扩展性相关说明，参见 [`docker buildx create`](/reference/cli/docker/buildx/create.md#driver-opt) 的可选项。

## 节点分配（Node assignment）

Kubernetes 驱动支持通过 `nodeSelector` 与 `tolerations` 驱动选项控制 BuildKit Pod 的调度。
如需使用自定义调度器，可设置 `schedulername` 选项。

你可以使用 `annotations` 与 `labels` 为承载构建器的 Deployment 与 Pod 附加元数据。

`nodeSelector` 参数值为以逗号分隔的键值对字符串，键为节点标签名、值为标签内容。
例如：`"nodeselector=kubernetes.io/arch=arm64"`

`tolerations` 参数是以分号分隔的污点（taint）列表，取值与 Kubernetes 清单一致。
每一项可指定污点键以及对应的值、运算符或效果。例如：
`"tolerations=key=foo,value=bar;key=foo2,operator=exists;key=foo3,effect=NoSchedule"`

这些选项接收以逗号分隔的字符串。受 Shell 引号规则影响，建议使用单引号包裹取值。
甚至可以将整个 `--driver-opt` 用单引号包裹，例如：

```console
$ docker buildx create \
  --bootstrap \
  --name=kube \
  --driver=kubernetes \
  '--driver-opt="nodeselector=label1=value1,label2=value2","tolerations=key=key1,value=value1"'
```

## 多平台构建

Kubernetes 驱动支持创建[多平台镜像](/manuals/build/building/multi-platform.md)，
既可以借助 QEMU，也可以利用节点的原生架构。

### QEMU

与 `docker-container` 驱动类似，Kubernetes 驱动也支持使用
[QEMU](https://www.qemu.org/)（用户态）为非原生平台构建镜像。
在命令中添加 `--platform` 指定要输出的平台。

例如，构建同时支持 `amd64` 与 `arm64` 的 Linux 镜像：

```console
$ docker buildx build \
  --builder=kube \
  --platform=linux/amd64,linux/arm64 \
  -t <user>/<image> \
  --push .
```

> [!WARNING]
>
> QEMU 对非原生平台进行完整 CPU 模拟，速度远慢于原生构建。
> 编译、压缩/解压等计算密集型任务的性能会明显下降。

在构建中使用自定义 BuildKit 镜像或调用非原生二进制时，可能需要在创建构建器时通过 `qemu.install` 显式开启 QEMU：

```console
$ docker buildx create \
  --bootstrap \
  --name=kube \
  --driver=kubernetes \
  --driver-opt=namespace=buildkit,qemu.install=true
```

### 原生（Native）

如果集群中存在不同架构的节点，Kubernetes 驱动可以利用这些节点进行原生构建。
可通过 `docker buildx create` 的 `--append` 实现。

首先，创建仅支持单一架构（如 `amd64`）的构建器：

```console
$ docker buildx create \
  --bootstrap \
  --name=kube \
  --driver=kubernetes \
  --platform=linux/amd64 \
  --node=builder-amd64 \
  --driver-opt=namespace=buildkit,nodeselector="kubernetes.io/arch=amd64"
```

上述命令会创建名为 `kube` 的 Buildx 构建器，并包含名为 `builder-amd64` 的单个节点。
通过 `--node` 指定节点名是可选的；若不提供，Buildx 会随机生成。

注意：Buildx 中的“节点”与 Kubernetes 的“节点”概念不同。
此处的 Buildx 节点可以连接多台相同架构的 Kubernetes 节点。

创建好 `kube` 构建器后，可以使用 `--append` 引入其他架构。例如添加 `arm64`：

```console
$ docker buildx create \
  --append \
  --bootstrap \
  --name=kube \
  --driver=kubernetes \
  --platform=linux/arm64 \
  --node=builder-arm64 \
  --driver-opt=namespace=buildkit,nodeselector="kubernetes.io/arch=arm64"
```

列出构建器，可见 `kube` 构建器包含两个节点：

```console
$ docker buildx ls
NAME/NODE       DRIVER/ENDPOINT                                         STATUS   PLATFORMS
kube            kubernetes
  builder-amd64 kubernetes:///kube?deployment=builder-amd64&kubeconfig= running  linux/amd64*, linux/amd64/v2, linux/amd64/v3, linux/386
  builder-arm64 kubernetes:///kube?deployment=builder-arm64&kubeconfig= running  linux/arm64*
```

现在即可在构建命令中同时指定上述两个平台，构建多架构镜像：

```console
$ docker buildx build --builder=kube --platform=linux/amd64,linux/arm64 -t <user>/<image> --push .
```

你可以针对想要支持的任意多种架构，多次执行 `buildx create --append`。

## 无特权模式（Rootless mode）

Kubernetes 驱动支持无特权模式。其工作机制与前置条件，参见
[文档](https://github.com/moby/buildkit/blob/master/docs/rootless.md)。

在集群中启用它，可使用 `rootless=true` 驱动选项：

```console
$ docker buildx create \
  --name=kube \
  --driver=kubernetes \
  --driver-opt=namespace=buildkit,rootless=true
```

这会在不设置 `securityContext.privileged` 的情况下创建 Pod。

需要 Kubernetes 1.19 或更高版本。推荐使用 Ubuntu 作为宿主内核。

## 示例：在 Kubernetes 中创建 Buildx 构建器

本指南将演示如何：

- 为 Buildx 资源创建命名空间
- 创建一个 Kubernetes 构建器
- 列出可用的构建器
- 使用 Kubernetes 构建器构建镜像

前置条件：

- 你已有一个 Kubernetes 集群；若没有，可先安装 [minikube](https://minikube.sigs.k8s.io/docs/)。
- 目标集群可通过 `kubectl` 访问，必要时正确[设置 `KUBECONFIG` 环境变量](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/#set-the-kubeconfig-environment-variable)。

1. 创建 `buildkit` 命名空间。

   单独的命名空间有助于将 Buildx 资源与集群中的其他资源隔离。

   ```console
   $ kubectl create namespace buildkit
   namespace/buildkit created
   ```

2. 使用 Kubernetes 驱动创建新的构建器：

   ```console
   $ docker buildx create \
     --bootstrap \
     --name=kube \
     --driver=kubernetes \
     --driver-opt=namespace=buildkit
   ```

   > [!NOTE]
   > 记得在驱动选项中指定命名空间。

3. 使用 `docker buildx ls` 列出可用构建器。

   ```console
   $ docker buildx ls
   NAME/NODE                DRIVER/ENDPOINT STATUS  PLATFORMS
   kube                     kubernetes
     kube0-6977cdcb75-k9h9m                 running linux/amd64, linux/amd64/v2, linux/amd64/v3, linux/386
   default *                docker
     default                default         running linux/amd64, linux/386
   ```

4. 使用 `kubectl` 检查由构建驱动创建并运行中的 Pod。

   ```console
   $ kubectl -n buildkit get deployments
   NAME    READY   UP-TO-DATE   AVAILABLE   AGE
   kube0   1/1     1            1           32s

   $ kubectl -n buildkit get pods
   NAME                     READY   STATUS    RESTARTS   AGE
   kube0-6977cdcb75-k9h9m   1/1     Running   0          32s
   ```

   构建驱动会在集群中指定命名空间（此处为 `buildkit`）创建所需资源，同时将驱动配置保存在本地。

5. 在运行 buildx 命令时通过 `--builder` 指定并使用新构建器。例如：

   ```console
   # 将 <registry> 替换为你的 Docker 用户名
   # 将 <image> 替换为你要构建的镜像名称
   docker buildx build \
     --builder=kube \
     -t <registry>/<image> \
     --push .
   ```

完成。你已经使用 Buildx 在 Kubernetes Pod 中构建了一个镜像。

## 延伸阅读

关于 Kubernetes 驱动的更多信息，参见
[buildx 参考](/reference/cli/docker/buildx/create.md#driver)。
