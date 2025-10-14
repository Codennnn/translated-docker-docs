---
description: 如何启用嵌套虚拟化的指南
keywords: 嵌套虚拟化, Docker Desktop, Windows, 虚拟机, VDI 环境
title: 在 VM 或 VDI 环境中运行 Windows 版 Docker Desktop
linkTitle: VM 或 VDI 环境
aliases:
 - /desktop/nested-virtualization/
 - /desktop/vm-vdi/
weight: 30
---

Docker 建议在 Mac、Linux 或 Windows 上原生运行 Docker Desktop。不过，只要虚拟桌面配置正确，Windows 版 Docker Desktop 也可以在虚拟桌面中运行。

要在虚拟桌面环境中运行 Docker Desktop，取决于是否支持嵌套虚拟化，你有两种选择：

- 如果你的环境支持嵌套虚拟化，可以使用默认的本地 Linux 虚拟机运行 Docker Desktop。
- 如果不支持嵌套虚拟化，建议使用 [Docker Offload](/offload/)。

## 使用 Docker Offload

Docker Offload 允许你将容器工作负载卸载到高性能、全托管的云环境，从而获得无缝的混合体验。

在不支持嵌套虚拟化的虚拟桌面环境中，Docker Offload 非常有用。在这类环境里，Docker Desktop 会默认使用 Docker Offload，确保你无需依赖本地虚拟化也能构建和运行容器。

Docker Offload 将 Docker Desktop 客户端与 Docker 引擎解耦，使 Docker CLI 与 Docker Desktop 仪表板可以像访问本地资源那样访问云端资源。运行容器时，Docker 会通过 SSH 隧道为 Docker Desktop 连接一个安全、隔离且短暂的云端环境。尽管容器在远端运行，但绑定挂载与端口转发等功能依然可无缝使用，提供接近本地的体验。要使用 Docker Offload：

开始使用请参见 [Docker Offload 快速入门](/offload/quickstart/)。

## 使用嵌套虚拟化时的虚拟桌面支持

> [!NOTE]
>
> 在虚拟桌面上运行 Docker Desktop 的支持仅面向 Docker Business 客户，且仅限 VMware ESXi 或 Azure VM。

Docker 支持包含在 VM 内安装并运行 Docker Desktop，前提是正确启用嵌套虚拟化。目前仅验证支持的虚拟化管理程序为 VMware ESXi 与 Azure，其他 VM 不在支持范围内。更多支持信息，参见[获取支持](/manuals/desktop/troubleshoot-and-support/support.md)。

对于 Docker 无法控制的问题与间歇性故障，请联系你的虚拟化管理程序厂商。不同厂商提供的支持级别不同。例如，Microsoft 支持在本地与 Azure 上运行嵌套的 Hyper-V（存在版本限制），而 VMware ESXi 可能并非如此。

Docker 不支持在同一台机器上于 VM 或 VDI 环境中运行多个 Docker Desktop 实例。

> [!TIP]
>
> 如果你在 Citrix VDI 中运行 Docker Desktop，请注意 Citrix 可搭配多种底层虚拟化管理程序，例如 VMware、Hyper-V、Citrix Hypervisor/XenServer。而 Docker Desktop 需要嵌套虚拟化，Citrix Hypervisor/XenServer 并不支持。
>
> 请与你的 Citrix 管理员或 VDI 基础设施团队确认使用的虚拟化管理程序，以及是否已启用嵌套虚拟化。

## 启用嵌套虚拟化

在未使用 Docker Cloud 的虚拟机上安装 Docker Desktop 之前，你必须先启用嵌套虚拟化。

### 在 VMware ESXi 上启用嵌套虚拟化

在 vSphere VM 内嵌套其他虚拟化管理程序（如 Hyper-V）[并非受支持的场景](https://kb.vmware.com/s/article/2009916)。不过，在 VMware ESXi VM 中运行 Hyper-V VM 在技术上是可行的，并且根据不同版本，ESXi 将硬件辅助虚拟化作为受支持的功能。内部测试时使用了 1 个 CPU（4 核）与 12GB 内存的 VM。

有关如何向来宾操作系统开放硬件辅助虚拟化的步骤，请参见 [VMware 文档](https://docs.vmware.com/en/VMware-vSphere/7.0/com.vmware.vsphere.vm_admin.doc/GUID-2A98801C-68E8-47AF-99ED-00C63E4857F6.html)。

### 在 Azure 虚拟机上启用嵌套虚拟化

Microsoft 支持在 Azure VM 中运行嵌套的 Hyper-V。

对 Azure 虚拟机，请[检查所选 VM 规格是否支持嵌套虚拟化](https://docs.microsoft.com/en-us/azure/virtual-machines/sizes)。Microsoft 提供了[Azure VM 规格说明列表](https://docs.microsoft.com/en-us/azure/virtual-machines/acu)，并标注了当前支持嵌套虚拟化的规格。内部测试使用的是 D4s_v5 机型。为了获得更佳的 Docker Desktop 性能，建议采用该规格或更高配置。

## 在 Nutanix 驱动的 VDI 环境中的 Docker Desktop 支持

只要底层 Windows 环境支持 WSL 2 或 Windows 容器模式，Docker Desktop 就可以在由 Nutanix 驱动的 VDI 环境中使用。由于 Nutanix 官方支持 WSL 2，只要 WSL 2 在该 VDI 环境中运行正常，Docker Desktop 应能如预期工作。

如果使用 Windows 容器模式，请确认 Nutanix 环境支持 Hyper-V 或其他 Windows 容器后端。

### 支持的配置

Docker Desktop 遵循前文所述的 VDI 支持定义（见[上文](#virtual-desktop-support-when-using-nested-virtualization)）：

 - 持久化 VDI 环境（支持）：跨会话使用同一虚拟桌面实例，已安装软件与配置可以保留。

 - 非持久化 VDI 环境（不支持）：操作系统在会话之间会重置，需要每次重新安装或配置，Docker Desktop 不支持这类环境。

### 支持范围与职责

关于 WSL 2 相关问题，请联系 Nutanix 支持；关于 Docker Desktop 相关问题，请联系 Docker 支持。

## 其他资源

- [Microsoft Dev Box 上的 Docker Desktop](/manuals/enterprise/enterprise-deployment/dev-box.md)