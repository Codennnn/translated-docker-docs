---
description: 了解如何配置 Docker Compose 以在基于 CUDA 的容器中使用 NVIDIA GPU
keywords: documentation, docs, docker, compose, GPU access, NVIDIA, samples
title: 使用 GPU 访问运行 Docker Compose 服务 
linkTitle: 启用 GPU 支持
weight: 90
aliases:
- /compose/gpu-support/
---

如果 Docker 主机具备 GPU 设备，并且 Docker 守护进程已正确配置，那么 Compose 中的服务就可以声明 GPU 设备的保留（reservation）。为此，请确保已安装所需的[先决条件](/manuals/engine/containers/resource_constraints.md#gpu)。

下文示例专注于通过 Docker Compose 为服务容器提供对 GPU 设备的访问能力。
你可以使用 `docker-compose` 或 `docker compose` 命令。更多信息参见[迁移到 Compose V2](/manuals/compose/releases/migrate.md)。

## 为服务容器启用 GPU 访问

在需要使用 GPU 的服务中，你可以在 `compose.yaml` 中使用 Compose Deploy 规范里的 [device](/reference/compose-file/deploy.md#devices) 属性来引用 GPU。

这使你能够对 GPU 预留进行更细粒度的控制，可为以下设备属性设置自定义值： 

- `capabilities`：以字符串列表的形式指定。例如 `capabilities: [gpu]`。你必须在 Compose 文件中设置该字段，否则服务部署会报错。
- `count`：可为整数或 `all`，表示需要预留的 GPU 数量（前提是主机上有足够数量的 GPU）。若设置为 `all` 或未指定，默认使用主机上的所有可用 GPU。
- `device_ids`：以字符串列表形式指定，表示主机上的 GPU 设备 ID。可通过主机上的 `nvidia-smi` 输出找到设备 ID。若未设置 `device_ids`，默认使用主机上的所有可用 GPU。
- `driver`：字符串，例如 `driver: 'nvidia'`。
- `options`：表示驱动程序特定选项的键值对。


> [!IMPORTANT]
>
> 你必须设置 `capabilities` 字段，否则服务部署时会报错。

> [!NOTE]
>
> `count` 与 `device_ids` 互斥。一次只能定义其中一个字段。

关于这些属性的更多说明，参见 [Compose Deploy 规范](/reference/compose-file/deploy.md#devices)。

### 示例：可访问 1 块 GPU 的 Compose 文件

```yaml
services:
  test:
    image: nvidia/cuda:12.9.0-base-ubuntu22.04
    command: nvidia-smi
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: 1
              capabilities: [gpu]
```

使用 Docker Compose 运行：

```console
$ docker compose up
Creating network "gpu_default" with the default driver
Creating gpu_test_1 ... done
Attaching to gpu_test_1    
test_1  | +-----------------------------------------------------------------------------+
test_1  | | NVIDIA-SMI 450.80.02    Driver Version: 450.80.02    CUDA Version: 11.1     |
test_1  | |-------------------------------+----------------------+----------------------+
test_1  | | GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
test_1  | | Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
test_1  | |                               |                      |               MIG M. |
test_1  | |===============================+======================+======================|
test_1  | |   0  Tesla T4            On   | 00000000:00:1E.0 Off |                    0 |
test_1  | | N/A   23C    P8     9W /  70W |      0MiB / 15109MiB |      0%      Default |
test_1  | |                               |                      |                  N/A |
test_1  | +-------------------------------+----------------------+----------------------+
test_1  |                                                                                
test_1  | +-----------------------------------------------------------------------------+
test_1  | | Processes:                                                                  |
test_1  | |  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
test_1  | |        ID   ID                                                   Usage      |
test_1  | |=============================================================================|
test_1  | |  No running processes found                                                 |
test_1  | +-----------------------------------------------------------------------------+
gpu_test_1 exited with code 0

```

在具有多块 GPU 的主机上，可以通过 `device_ids` 指定要使用的特定 GPU 设备，并使用 `count` 限制分配给服务容器的 GPU 数量。

在每个服务定义中可以选择使用 `count` 或 `device_ids`。如果同时设置两者、指定了无效的设备 ID，或 `count` 大于系统中的 GPU 数量，都会报错。

```console
$ nvidia-smi   
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 450.80.02    Driver Version: 450.80.02    CUDA Version: 11.0     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  Tesla T4            On   | 00000000:00:1B.0 Off |                    0 |
| N/A   72C    P8    12W /  70W |      0MiB / 15109MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+
|   1  Tesla T4            On   | 00000000:00:1C.0 Off |                    0 |
| N/A   67C    P8    11W /  70W |      0MiB / 15109MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+
|   2  Tesla T4            On   | 00000000:00:1D.0 Off |                    0 |
| N/A   74C    P8    12W /  70W |      0MiB / 15109MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+
|   3  Tesla T4            On   | 00000000:00:1E.0 Off |                    0 |
| N/A   62C    P8    11W /  70W |      0MiB / 15109MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+
```

## 访问特定设备

仅允许访问 GPU-0 与 GPU-3：

```yaml
services:
  test:
    image: tensorflow/tensorflow:latest-gpu
    command: python -c "import tensorflow as tf;tf.test.gpu_device_name()"
    deploy:
      resources:
        reservations:
          devices:
          - driver: nvidia
            device_ids: ['0', '3']
            capabilities: [gpu]

```
