---
title: Docker Desktop（Windows）中的 GPU 支持
linkTitle: GPU 支持
weight: 40
description: 如何在 Docker Desktop 中使用 GPU
keywords: gpu, gpu support, nvidia, wsl2, docker desktop, windows
toc_max: 3
aliases:
- /desktop/gpu/
---

> [!NOTE]
>
> 目前，Docker Desktop 的 GPU 支持仅在启用 WSL 2 后端的 Windows 上可用。

Windows 版 Docker Desktop 在 NVIDIA 显卡上支持 NVIDIA GPU Paravirtualization（GPU‑PV），允许容器访问 GPU 资源，用于 AI、机器学习或视频处理等计算密集型工作负载。

## 前提条件

要启用 WSL 2 的 GPU 虚拟化，你需要：

- 一台配备 NVIDIA GPU 的 Windows 设备
- 已更新到最新版本的 Windows 10 或 Windows 11
- 支持 WSL 2 GPU 虚拟化的 NVIDIA [最新驱动](https://developer.nvidia.com/cuda/wsl)
- 最新版 WSL 2 Linux 内核。请在命令行运行 `wsl --update`
- 确保在 Docker Desktop 中已[开启 WSL 2 后端](wsl/_index.md#turn-on-docker-desktop-wsl-2)

## 验证 GPU 支持

要确认 Docker 内能够正常访问 GPU，请运行：

```console
$ docker run --rm -it --gpus=all nvcr.io/nvidia/k8s/cuda-sample:nbody nbody -gpu -benchmark
```

该命令会在 GPU 上运行 n‑body 模拟的基准测试。输出类似于：

```console
Run "nbody -benchmark [-numbodies=<numBodies>]" to measure performance.
        -fullscreen       (run n-body simulation in fullscreen mode)
        -fp64             (use double precision floating point values for simulation)
        -hostmem          (stores simulation data in host memory)
        -benchmark        (run benchmark to measure performance)
        -numbodies=<N>    (number of bodies (>= 1) to run in simulation)
        -device=<d>       (where d=0,1,2.... for the CUDA device to use)
        -numdevices=<i>   (where i=(number of CUDA devices > 0) to use for simulation)
        -compare          (compares simulation results running once on the default GPU and once on the CPU)
        -cpu              (run n-body simulation on the CPU)
        -tipsy=<file.bin> (load a tipsy model file for simulation)

> NOTE: The CUDA Samples are not meant for performance measurements. Results may vary when GPU Boost is enabled.

> Windowed mode
> Simulation data stored in video memory
> Single precision floating point simulation
> 1 Devices used for simulation
MapSMtoCores for SM 7.5 is undefined.  Default to use 64 Cores/SM
GPU Device 0: "GeForce RTX 2060 with Max-Q Design" with compute capability 7.5

> Compute 7.5 CUDA device: [GeForce RTX 2060 with Max-Q Design]
30720 bodies, total time for 10 iterations: 69.280 ms
= 136.219 billion interactions per second
= 2724.379 single-precision GFLOP/s at 20 flops per interaction
```

## 运行真实模型：使用 Ollama 运行 Llama2

使用 [官方 Ollama 镜像](https://hub.docker.com/r/ollama/ollama) 在 GPU 加速下运行 Llama2 大语言模型：

```console
$ docker run --gpus=all -d -v ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama
```

然后启动模型：

```console
$ docker exec -it ollama ollama run llama2
```
