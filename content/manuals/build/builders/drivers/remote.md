---
title: 远程驱动
description: |
  远程驱动允许你连接到手动搭建并配置的远程 BuildKit 实例。
keywords: build, buildx, driver, builder, remote
aliases:
  - /build/buildx/drivers/remote/
  - /build/building/drivers/remote/
  - /build/drivers/remote/
---

Buildx 的远程驱动适用于更复杂的自定义构建工作负载，
可连接由外部管理的 BuildKit 实例。对于需要手动管理 BuildKit 守护进程，
或由其他来源对外暴露 BuildKit 守护进程的场景，这种方式非常有用。

## 概要

```console
$ docker buildx create \
  --name remote \
  --driver remote \
  tcp://localhost:1234
```

下表说明了可通过 `--driver-opt` 传入的驱动特定选项：

| 参数           | 类型    | 默认值             | 描述                                                                     |
| -------------- | ------- | ------------------ | ------------------------------------------------------------------------ |
| `key`          | String  |                    | 设置 TLS 客户端私钥。                                                     |
| `cert`         | String  |                    | 指向提供给 `buildkitd` 的 TLS 客户端证书的绝对路径。                       |
| `cacert`       | String  |                    | 用于校验的 TLS 根证书（CA）绝对路径。                                     |
| `servername`   | String  | 端点主机名         | 在请求中使用的 TLS 服务器名称。                                           |
| `default-load` | Boolean | `false`            | 构建完成后自动将镜像加载到 Docker Engine 的镜像存储。                     |

## 示例：通过 Unix 套接字连接远程 BuildKit

本指南演示如何让 BuildKit 守护进程监听 Unix 套接字，并通过该套接字由 Buildx 进行连接。

1. 确认已安装 [BuildKit](https://github.com/moby/buildkit)。

   例如，你可以用如下命令启动一个 buildkitd 实例：

   ```console
   $ sudo ./buildkitd --group $(id -gn) --addr unix://$HOME/buildkitd.sock
   ```

   或者，参考[此处](https://github.com/moby/buildkit/blob/master/docs/rootless.md)以无特权（rootless）模式运行 buildkitd，
   或参考[此处](https://github.com/moby/buildkit/tree/master/examples/systemd) 了解作为 systemd 服务运行的示例。

2. 确认你已有可连接的 Unix 套接字：

   ```console
   $ ls -lh /home/user/buildkitd.sock
   srw-rw---- 1 root user 0 May  5 11:04 /home/user/buildkitd.sock
   ```

3. 使用远程驱动让 Buildx 连接到该套接字：

   ```console
   $ docker buildx create \
     --name remote-unix \
     --driver remote \
     unix://$HOME/buildkitd.sock
   ```

4. 通过 `docker buildx ls` 列出可用的构建器，你应该能看到 `remote-unix`：

   ```console
   $ docker buildx ls
   NAME/NODE           DRIVER/ENDPOINT                        STATUS  PLATFORMS
   remote-unix         remote
     remote-unix0      unix:///home/.../buildkitd.sock        running linux/amd64, linux/amd64/v2, linux/amd64/v3, linux/386
   default *           docker
     default           default                                running linux/amd64, linux/386
   ```

你可以使用 `docker buildx use remote-unix` 将其设为默认构建器，
或在每次构建时通过 `--builder` 指定：

```console
$ docker buildx build --builder=remote-unix -t test --load .
```

如果希望将构建结果加载到 Docker 守护进程，请记得使用 `--load` 标志。

## 示例：在 Docker 容器中使用远程 BuildKit

本指南演示一种类似 `docker-container` 驱动的方案：手动启动一个 BuildKit 的 Docker 容器，
并通过 Buildx 的远程驱动连接到它。该过程会手动创建容器，并通过其暴露的端口访问。
（实际场景中，通过 Docker 守护进程连接 BuildKit 的 `docker-container` 驱动通常更合适，
此处仅作示例说明。）

1.  为 BuildKit 生成证书。

    你可以使用该[ Bake 定义](https://github.com/moby/buildkit/blob/master/examples/create-certs)作为起点：

    ```console
    SAN="localhost 127.0.0.1" docker buildx bake "https://github.com/moby/buildkit.git#master:examples/create-certs"
    ```

    注意：虽然可以在不使用 TLS 的情况下通过 TCP 暴露 BuildKit，但并不推荐。
    这么做意味着任何人都可以在无凭据的情况下访问 BuildKit。

2.  在 `.certs/` 目录中生成证书后，启动容器：

    ```console
    $ docker run -d --rm \
      --name=remote-buildkitd \
      --privileged \
      -p 1234:1234 \
      -v $PWD/.certs:/etc/buildkit/certs \
      moby/buildkit:latest \
      --addr tcp://0.0.0.0:1234 \
      --tlscacert /etc/buildkit/certs/daemon/ca.pem \
      --tlscert /etc/buildkit/certs/daemon/cert.pem \
      --tlskey /etc/buildkit/certs/daemon/key.pem
    ```

    上述命令会启动一个 BuildKit 容器，并将守护进程的 1234 端口暴露到本地主机。

3.  使用 Buildx 连接到该运行中的容器：

    ```console
    $ docker buildx create \
      --name remote-container \
      --driver remote \
      --driver-opt cacert=${PWD}/.certs/client/ca.pem,cert=${PWD}/.certs/client/cert.pem,key=${PWD}/.certs/client/key.pem,servername=<TLS_SERVER_NAME> \
      tcp://localhost:1234
    ```

    或者，使用 `docker-container://` URL 方案在不指定端口的情况下连接到该 BuildKit 容器：

    ```console
    $ docker buildx create \
      --name remote-container \
      --driver remote \
      docker-container://remote-container
    ```

## 示例：在 Kubernetes 中使用远程 BuildKit

本指南演示一种类似 `kubernetes` 驱动的方案：手动创建 BuildKit 的 `Deployment`。
虽然 `kubernetes` 驱动会在幕后替你完成这些工作，但在某些情况下，你可能希望手动伸缩 BuildKit。
此外，如果从 Kubernetes Pod 内发起构建，则需要在每个 Pod 内重新创建 Buildx 构建器，或在 Pod 之间复制。

1. 按照[这里的说明](https://github.com/moby/buildkit/tree/master/examples/kubernetes)创建 `buildkitd` 的 Kubernetes Deployment。

   按指南操作，使用 [create-certs.sh](https://github.com/moby/buildkit/blob/master/examples/kubernetes/create-certs.sh)
   为 BuildKit 守护进程和客户端创建证书，并创建连接这些 Pod 的 Service。

2. 假设该 Service 名为 `buildkitd`，在 Buildx 中创建一个远程构建器，确保相关证书文件存在：

   ```console
   $ docker buildx create \
     --name remote-kubernetes \
     --driver remote \
     --driver-opt cacert=${PWD}/.certs/client/ca.pem,cert=${PWD}/.certs/client/cert.pem,key=${PWD}/.certs/client/key.pem \
     tcp://buildkitd.default.svc:1234
   ```

请注意，此方式仅在集群内部可用，因为前述 BuildKit 搭建指南只创建了 `ClusterIP` 类型的 Service。
如果要从集群外访问构建器，可以配置并使用 Ingress（不在本指南范围内）。

### 在 Kubernetes 中调试远程构建器

如果你在访问部署于 Kubernetes 的远程构建器时遇到问题，
可以使用 `kube-pod://` URL 方案，通过 Kubernetes API 直接连接到某个 BuildKit Pod。
注意，这种方式仅连接到该 Deployment 中的单个 Pod。

```console
$ kubectl get pods --selector=app=buildkitd -o json | jq -r '.items[].metadata.name'
buildkitd-XXXXXXXXXX-xxxxx
$ docker buildx create \
  --name remote-container \
  --driver remote \
  kube-pod://buildkitd-XXXXXXXXXX-xxxxx
```

或者，使用 `kubectl` 的端口转发机制：

```console
$ kubectl port-forward svc/buildkitd 1234:1234
```

随后即可将远程驱动指向 `tcp://localhost:1234`。
