---
title: 容器设备接口（CDI）
weight: 60
description: 在构建中使用 CDI 访问 GPU 与其他设备
keywords: build, buildkit, buildx, guide, tutorial, cdi, device, gpu, nvidia, cuda, amd, rocm
---

<!-- vale Docker.We = NO -->

[容器设备接口（CDI）](https://github.com/cncf-tags/container-device-interface/blob/main/SPEC.md)
是一份规范，旨在标准化容器对各类设备（如 GPU、FPGA 及其他硬件加速器）的暴露与使用方式。
它为在容器化环境中使用硬件设备提供了一种更一致、更安全的机制，用以解决不同设备在安装与配置上的差异带来的问题。

除了允许容器与设备节点交互之外，CDI 还支持为设备指定额外配置，例如环境变量、宿主机挂载（如共享对象），以及可执行的 Hook。

## 快速开始

开始使用 CDI 之前，你需要准备好兼容的环境。
这包括安装 Docker v27+ 并完成[CDI 配置](/reference/cli/dockerd.md#configure-cdi-devices)，以及安装 Buildx v0.22+。

你还需要在以下任一目录中，创建[设备规范（JSON 或 YAML）](https://github.com/cncf-tags/container-device-interface/blob/main/SPEC.md#cdi-json-specification)：

* `/etc/cdi`
* `/var/run/cdi`
* `/etc/buildkit/cdi`

> [!NOTE]
> 如果你直接使用 BuildKit，可在 [`buildkitd.toml` 配置文件](../buildkit/configure.md) 的 `cdi` 段落中设置 `specDirs` 来调整该位置。
> 如果你使用 `docker` 驱动通过 Docker 守护进程进行构建，请参阅[配置 CDI 设备](/reference/cli/dockerd.md#configure-cdi-devices)。

> [!NOTE]
> 如果你在 WSL 上创建容器构建器，需要确保已安装 [Docker Desktop](../../desktop/_index.md)，并启用
> [WSL 2 GPU 半虚拟化](../../desktop/features/gpu.md#prerequisites)。同时还需要 Buildx v0.27+ 以便在容器中挂载 WSL 库。

## 使用简单的 CDI 规范进行构建

从一个简单的 CDI 规范开始：它向构建环境注入一个环境变量，并将该规范写入 `/etc/cdi/foo.yaml`：

```yaml {title="/etc/cdi/foo.yaml"}
cdiVersion: "0.6.0"
kind: "vendor1.com/device"
devices:
- name: foo
  containerEdits:
    env:
    - FOO=injected
```

检查 `default` 构建器，确认 `vendor1.com/device` 已被识别为设备：

```console
$ docker buildx inspect
Name:   default
Driver: docker

Nodes:
Name:             default
Endpoint:         default
Status:           running
BuildKit version: v0.23.2
Platforms:        linux/amd64, linux/amd64/v2, linux/amd64/v3, linux/amd64/v4, linux/386
Labels:
 org.mobyproject.buildkit.worker.moby.host-gateway-ip: 172.17.0.1
Devices:
 Name:                  vendor1.com/device=foo
 Automatically allowed: false
GC Policy rule#0:
 All:            false
 Filters:        type==source.local,type==exec.cachemount,type==source.git.checkout
 Keep Duration:  48h0m0s
 Max Used Space: 658.9MiB
GC Policy rule#1:
 All:            false
 Keep Duration:  1440h0m0s
 Reserved Space: 4.657GiB
 Max Used Space: 953.7MiB
 Min Free Space: 2.794GiB
GC Policy rule#2:
 All:            false
 Reserved Space: 4.657GiB
 Max Used Space: 953.7MiB
 Min Free Space: 2.794GiB
GC Policy rule#3:
 All:            true
 Reserved Space: 4.657GiB
 Max Used Space: 953.7MiB
 Min Free Space: 2.794GiB
```

接下来，创建一个 Dockerfile 来使用该设备：

```dockerfile
# syntax=docker/dockerfile:1-labs
FROM busybox
RUN --device=vendor1.com/device \
  env | grep ^FOO=
```

这里我们使用 [`RUN --device` 指令](/reference/dockerfile.md#run---device)，并设置 `vendor1.com/device`，
表示请求该规范中的第一个可用设备。本例中即为 `foo`，它是 `/etc/cdi/foo.yaml` 中的第一个设备。

> [!NOTE]
> [`RUN --device` 指令](/reference/dockerfile.md#run---device) 自 [Dockerfile 前端 v1.14.0-labs](../buildkit/dockerfile-release-notes.md#1140-labs)
> 起仅在[`labs` 渠道](../buildkit/frontend.md#labs-channel)中提供，稳定语法暂不支持。

现在构建该 Dockerfile：

```console
$ docker buildx build .
[+] Building 0.4s (5/5) FINISHED                                                                                                        docker:default
 => [internal] load build definition from Dockerfile                                                                                    0.0s 
 => => transferring dockerfile: 155B                                                                                                    0.0s
 => resolve image config for docker-image://docker/dockerfile:1-labs                                                                    0.1s 
 => CACHED docker-image://docker/dockerfile:1-labs@sha256:9187104f31e3a002a8a6a3209ea1f937fb7486c093cbbde1e14b0fa0d7e4f1b5              0.0s
 => [internal] load metadata for docker.io/library/busybox:latest                                                                       0.1s 
 => [internal] load .dockerignore                                                                                                       0.0s
 => => transferring context: 2B                                                                                                         0.0s 
ERROR: failed to build: failed to solve: failed to load LLB: device vendor1.com/device=foo is requested by the build but not allowed
```

构建失败的原因是：如上方 `buildx inspect` 输出所示，设备 `vendor1.com/device=foo` 并不会被构建自动允许：

```text
Devices:
 Name:                  vendor1.com/device=foo
 Automatically allowed: false
```

要允许该设备，可以在 `docker buildx build` 命令中使用 [`--allow` 标志](/reference/cli/docker/buildx/build.md#allow)：

```console
$ docker buildx build --allow device .
```

或者，你也可以在 CDI 规范中设置 `org.mobyproject.buildkit.device.autoallow` 注解，
从而对所有构建自动允许该设备：

```yaml {title="/etc/cdi/foo.yaml"}
cdiVersion: "0.6.0"
kind: "vendor1.com/device"
devices:
- name: foo
  containerEdits:
    env:
    - FOO=injected
annotations:
  org.mobyproject.buildkit.device.autoallow: true
```

现在使用 `--allow device` 标志再次运行构建：

```console
$ docker buildx build --progress=plain --allow device .
#0 building with "default" instance using docker driver

#1 [internal] load build definition from Dockerfile
#1 transferring dockerfile: 159B done
#1 DONE 0.0s

#2 resolve image config for docker-image://docker/dockerfile:1-labs
#2 DONE 0.1s

#3 docker-image://docker/dockerfile:1-labs@sha256:9187104f31e3a002a8a6a3209ea1f937fb7486c093cbbde1e14b0fa0d7e4f1b5
#3 CACHED

#4 [internal] load metadata for docker.io/library/busybox:latest
#4 DONE 0.1s

#5 [internal] load .dockerignore
#5 transferring context: 2B done
#5 DONE 0.0s

#6 [1/2] FROM docker.io/library/busybox:latest@sha256:f85340bf132ae937d2c2a763b8335c9bab35d6e8293f70f606b9c6178d84f42b
#6 CACHED

#7 [2/2] RUN --device=vendor1.com/device   env | grep ^FOO=
#7 0.155 FOO=injected
#7 DONE 0.2s
```

这次构建成功，输出表明 `FOO` 环境变量已按 CDI 规范注入到构建环境中。

## 配置支持 GPU 的容器构建器

本节演示如何使用 NVIDIA GPU 配置一个[容器构建器](../builders/drivers/docker-container.md)。
自 Buildx v0.22 起，如果宿主机内核已安装 GPU 驱动，在创建新容器构建器时会自动为其添加 GPU 请求。
这与在 `docker run` 命令中使用 [`--gpus=all`](/reference/cli/docker/container/run.md#gpus) 类似。

> [!NOTE]
> 我们提供了一个专门定制的 BuildKit 镜像，因为当前 BuildKit 的官方镜像基于 Alpine，无法支持 NVIDIA 驱动。
> 下方的镜像基于 Ubuntu，会安装 NVIDIA 客户端库，并在构建请求设备时为容器构建器生成你的 GPU 的 CDI 规范。
> 该镜像暂时托管在 Docker Hub：`crazymax/buildkit:v0.23.2-ubuntu-nvidia`。

现在使用 Buildx 创建一个名为 `gpubuilder` 的容器构建器：

```console
$ docker buildx create --name gpubuilder --driver-opt "image=crazymax/buildkit:v0.23.2-ubuntu-nvidia" --bootstrap
#1 [internal] booting buildkit
#1 pulling image crazymax/buildkit:v0.23.2-ubuntu-nvidia
#1 pulling image crazymax/buildkit:v0.23.2-ubuntu-nvidia 1.0s done
#1 creating container buildx_buildkit_gpubuilder0
#1 creating container buildx_buildkit_gpubuilder0 8.8s done
#1 DONE 9.8s
gpubuilder
```

检查该构建器：

```console
$ docker buildx inspect gpubuilder
Name:          gpubuilder
Driver:        docker-container
Last Activity: 2025-07-10 08:18:09 +0000 UTC

Nodes:
Name:                  gpubuilder0
Endpoint:              unix:///var/run/docker.sock
Driver Options:        image="crazymax/buildkit:v0.23.2-ubuntu-nvidia"
Status:                running
BuildKit daemon flags: --allow-insecure-entitlement=network.host
BuildKit version:      v0.23.2
Platforms:             linux/amd64, linux/amd64/v2, linux/amd64/v3, linux/arm64, linux/riscv64, linux/ppc64le, linux/s390x, linux/386, linux/arm/v7, linux/arm/v6
Labels:
 org.mobyproject.buildkit.worker.executor:         oci
 org.mobyproject.buildkit.worker.hostname:         d6aa9cbe8462
 org.mobyproject.buildkit.worker.network:          host
 org.mobyproject.buildkit.worker.oci.process-mode: sandbox
 org.mobyproject.buildkit.worker.selinux.enabled:  false
 org.mobyproject.buildkit.worker.snapshotter:      overlayfs
Devices:
 Name:      nvidia.com/gpu
 On-Demand: true
GC Policy rule#0:
 All:            false
 Filters:        type==source.local,type==exec.cachemount,type==source.git.checkout
 Keep Duration:  48h0m0s
 Max Used Space: 488.3MiB
GC Policy rule#1:
 All:            false
 Keep Duration:  1440h0m0s
 Reserved Space: 9.313GiB
 Max Used Space: 93.13GiB
 Min Free Space: 188.1GiB
GC Policy rule#2:
 All:            false
 Reserved Space: 9.313GiB
 Max Used Space: 93.13GiB
 Min Free Space: 188.1GiB
GC Policy rule#3:
 All:            true
 Reserved Space: 9.313GiB
 Max Used Space: 93.13GiB
 Min Free Space: 188.1GiB
```

可以看到，构建器中已将 `nvidia.com/gpu` 厂商识别为一个设备，这意味着已检测到相关驱动。

你也可以通过 `nvidia-smi` 检查容器中是否可用 NVIDIA GPU 设备：

```console
$ docker exec -it buildx_buildkit_gpubuilder0 nvidia-smi -L
GPU 0: Tesla T4 (UUID: GPU-6cf00fa7-59ac-16f2-3e83-d24ccdc56f84)
```

## 使用 GPU 支持进行构建

创建一个简单的 Dockerfile 来使用 GPU 设备：

```dockerfile
# syntax=docker/dockerfile:1-labs
FROM ubuntu
RUN --device=nvidia.com/gpu nvidia-smi -L
```

使用刚才创建的 `gpubuilder` 构建器运行构建：

```console
$ docker buildx --builder gpubuilder build --progress=plain .
#0 building with "gpubuilder" instance using docker-container driver
...

#7 preparing device nvidia.com/gpu
#7 0.000 > apt-get update
...
#7 4.872 > apt-get install -y gpg
...
#7 10.16 Downloading NVIDIA GPG key
#7 10.21 > apt-get update
...
#7 12.15 > apt-get install -y --no-install-recommends nvidia-container-toolkit-base
...
#7 17.80 time="2025-04-15T08:58:16Z" level=info msg="Generated CDI spec with version 0.8.0"
#7 DONE 17.8s

#8 [2/2] RUN --device=nvidia.com/gpu nvidia-smi -L
#8 0.527 GPU 0: Tesla T4 (UUID: GPU-6cf00fa7-59ac-16f2-3e83-d24ccdc56f84)
#8 DONE 1.6s
```

如你所见，步骤 `#7` 正在为 `nvidia.com/gpu` 设备做准备：
它会安装客户端库与相关工具，以便为该 GPU 生成 CDI 规范。

随后在容器中使用该 GPU 设备执行 `nvidia-smi -L` 命令，输出展示了 GPU 的 UUID。

你可以通过以下命令，在容器构建器内部查看生成的 CDI 规范：

```console
$ docker exec -it buildx_buildkit_gpubuilder0 cat /etc/cdi/nvidia.yaml
```

以下是在本示例中所用 EC2 实例 [`g4dn.xlarge`](https://aws.amazon.com/ec2/instance-types/g4/) 上的示例输出：

```yaml {collapse=true}
cdiVersion: 0.6.0
containerEdits:
  deviceNodes:
  - path: /dev/nvidia-modeset
  - path: /dev/nvidia-uvm
  - path: /dev/nvidia-uvm-tools
  - path: /dev/nvidiactl
  env:
  - NVIDIA_VISIBLE_DEVICES=void
  hooks:
  - args:
    - nvidia-cdi-hook
    - create-symlinks
    - --link
    - ../libnvidia-allocator.so.1::/usr/lib/x86_64-linux-gnu/gbm/nvidia-drm_gbm.so
    hookName: createContainer
    path: /usr/bin/nvidia-cdi-hook
  - args:
    - nvidia-cdi-hook
    - create-symlinks
    - --link
    - libcuda.so.1::/usr/lib/x86_64-linux-gnu/libcuda.so
    hookName: createContainer
    path: /usr/bin/nvidia-cdi-hook
  - args:
    - nvidia-cdi-hook
    - enable-cuda-compat
    - --host-driver-version=570.133.20
    hookName: createContainer
    path: /usr/bin/nvidia-cdi-hook
  - args:
    - nvidia-cdi-hook
    - update-ldcache
    - --folder
    - /usr/lib/x86_64-linux-gnu
    hookName: createContainer
    path: /usr/bin/nvidia-cdi-hook
  mounts:
  - containerPath: /run/nvidia-persistenced/socket
    hostPath: /run/nvidia-persistenced/socket
    options:
    - ro
    - nosuid
    - nodev
    - bind
    - noexec
  - containerPath: /usr/bin/nvidia-cuda-mps-control
    hostPath: /usr/bin/nvidia-cuda-mps-control
    options:
    - ro
    - nosuid
    - nodev
    - bind
  - containerPath: /usr/bin/nvidia-cuda-mps-server
    hostPath: /usr/bin/nvidia-cuda-mps-server
    options:
    - ro
    - nosuid
    - nodev
    - bind
  - containerPath: /usr/bin/nvidia-debugdump
    hostPath: /usr/bin/nvidia-debugdump
    options:
    - ro
    - nosuid
    - nodev
    - bind
  - containerPath: /usr/bin/nvidia-persistenced
    hostPath: /usr/bin/nvidia-persistenced
    options:
    - ro
    - nosuid
    - nodev
    - bind
  - containerPath: /usr/bin/nvidia-smi
    hostPath: /usr/bin/nvidia-smi
    options:
    - ro
    - nosuid
    - nodev
    - bind
  - containerPath: /usr/lib/x86_64-linux-gnu/libcuda.so.570.133.20
    hostPath: /usr/lib/x86_64-linux-gnu/libcuda.so.570.133.20
    options:
    - ro
    - nosuid
    - nodev
    - bind
  - containerPath: /usr/lib/x86_64-linux-gnu/libcudadebugger.so.570.133.20
    hostPath: /usr/lib/x86_64-linux-gnu/libcudadebugger.so.570.133.20
    options:
    - ro
    - nosuid
    - nodev
    - bind
  - containerPath: /usr/lib/x86_64-linux-gnu/libnvidia-allocator.so.570.133.20
    hostPath: /usr/lib/x86_64-linux-gnu/libnvidia-allocator.so.570.133.20
    options:
    - ro
    - nosuid
    - nodev
    - bind
  - containerPath: /usr/lib/x86_64-linux-gnu/libnvidia-cfg.so.570.133.20
    hostPath: /usr/lib/x86_64-linux-gnu/libnvidia-cfg.so.570.133.20
    options:
    - ro
    - nosuid
    - nodev
    - bind
  - containerPath: /usr/lib/x86_64-linux-gnu/libnvidia-gpucomp.so.570.133.20
    hostPath: /usr/lib/x86_64-linux-gnu/libnvidia-gpucomp.so.570.133.20
    options:
    - ro
    - nosuid
    - nodev
    - bind
  - containerPath: /usr/lib/x86_64-linux-gnu/libnvidia-ml.so.570.133.20
    hostPath: /usr/lib/x86_64-linux-gnu/libnvidia-ml.so.570.133.20
    options:
    - ro
    - nosuid
    - nodev
    - bind
  - containerPath: /usr/lib/x86_64-linux-gnu/libnvidia-nscq.so.570.133.20
    hostPath: /usr/lib/x86_64-linux-gnu/libnvidia-nscq.so.570.133.20
    options:
    - ro
    - nosuid
    - nodev
    - bind
  - containerPath: /usr/lib/x86_64-linux-gnu/libnvidia-nvvm.so.570.133.20
    hostPath: /usr/lib/x86_64-linux-gnu/libnvidia-nvvm.so.570.133.20
    options:
    - ro
    - nosuid
    - nodev
    - bind
  - containerPath: /usr/lib/x86_64-linux-gnu/libnvidia-opencl.so.570.133.20
    hostPath: /usr/lib/x86_64-linux-gnu/libnvidia-opencl.so.570.133.20
    options:
    - ro
    - nosuid
    - nodev
    - bind
  - containerPath: /usr/lib/x86_64-linux-gnu/libnvidia-pkcs11-openssl3.so.570.133.20
    hostPath: /usr/lib/x86_64-linux-gnu/libnvidia-pkcs11-openssl3.so.570.133.20
    options:
    - ro
    - nosuid
    - nodev
    - bind
  - containerPath: /usr/lib/x86_64-linux-gnu/libnvidia-pkcs11.so.570.133.20
    hostPath: /usr/lib/x86_64-linux-gnu/libnvidia-pkcs11.so.570.133.20
    options:
    - ro
    - nosuid
    - nodev
    - bind
  - containerPath: /usr/lib/x86_64-linux-gnu/libnvidia-ptxjitcompiler.so.570.133.20
    hostPath: /usr/lib/x86_64-linux-gnu/libnvidia-ptxjitcompiler.so.570.133.20
    options:
    - ro
    - nosuid
    - nodev
    - bind
  - containerPath: /lib/firmware/nvidia/570.133.20/gsp_ga10x.bin
    hostPath: /lib/firmware/nvidia/570.133.20/gsp_ga10x.bin
    options:
    - ro
    - nosuid
    - nodev
    - bind
  - containerPath: /lib/firmware/nvidia/570.133.20/gsp_tu10x.bin
    hostPath: /lib/firmware/nvidia/570.133.20/gsp_tu10x.bin
    options:
    - ro
    - nosuid
    - nodev
    - bind
devices:
- containerEdits:
    deviceNodes:
    - path: /dev/nvidia0
  name: "0"
- containerEdits:
    deviceNodes:
    - path: /dev/nvidia0
  name: GPU-6cf00fa7-59ac-16f2-3e83-d24ccdc56f84
- containerEdits:
    deviceNodes:
    - path: /dev/nvidia0
  name: all
kind: nvidia.com/gpu
```

恭喜你已使用 BuildKit 与 CDI 完成了首个启用 GPU 设备的构建。
