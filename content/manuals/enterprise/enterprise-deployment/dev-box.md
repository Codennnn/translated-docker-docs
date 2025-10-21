---
Title: Docker Desktop 在 Microsoft Dev Box 中
linkTitle: Microsoft Dev Box
description: 了解在 Microsoft Dev Box 中使用 Docker Desktop 的优势及设置方法
keywords: desktop, docker, windows, microsoft dev box
weight: 60
aliases:
 - /desktop/features/dev-box/
 - /desktop/setup/install/enterprise-deployment/dev-box/
---

Docker Desktop 作为预配置镜像在 Microsoft Azure Marketplace 中提供，可与 Microsoft Dev Box 配合使用，使开发人员能够快速在云中设置一致的开发环境。

Microsoft Dev Box 提供基于云的预配置开发工作站，让您无需配置本地开发环境即可编码、构建和测试应用程序。适用于 Microsoft Dev Box 的 Docker Desktop 镜像预装了 Docker Desktop 及其依赖项，为您提供了即用型容器化开发环境。

## 主要优势

- **预配置环境**：Docker Desktop、WSL2 和其他要求项已预装并配置完成
- **一致的开发体验**：确保所有团队成员使用相同的 Docker 环境
- **强大的资源**：访问比本地机器可能更多的计算能力和存储空间
- **状态持久性**：Dev Box 在会话之间保持您的状态，类似于本地计算机的休眠
- **无缝许可**：使用您现有的 Docker 订阅或直接通过 Azure Marketplace 购买新订阅

## 设置

### 先决条件 

- Azure 订阅
- Microsoft Dev Box 访问权限
- Docker 订阅（Pro、Team 或 Business）。您可以通过以下任一订阅选项在 Microsoft Dev Box 中使用 Docker Desktop：
   - 现有的或新的 Docker 订阅
   - 通过 Azure Marketplace 购买的新 Docker 订阅
   - 为您的组织配置了 SSO 的 Docker Business 订阅

### 在 Dev Box 中设置 Docker Desktop

1. 导航到 Azure Marketplace 中的 [Docker Desktop for Microsoft Dev Box](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/dockerinc1694120899427.devbox_azuremachine?tab=Overview) 页面。
2. 选择 **立即获取** 将虚拟机镜像添加到您的订阅。
3. 按照 Azure 工作流程完成设置。
4. 根据您组织的设置，使用该镜像创建虚拟机、分配给开发中心或创建 Dev Box 池。

### 激活 Docker Desktop

当您的 Dev Box 使用 Docker Desktop 镜像配置完成后：

1. 启动您的 Dev Box 实例。
2. 启动 Docker Desktop。
3. 使用您的 Docker ID 登录。

## 支持

如需获取以下相关问题的支持：

- Docker Desktop 配置、使用或许可：通过 [Docker 支持](https://hub.docker.com/support) 创建支持工单。
- Dev Box 创建、Azure 门户配置或网络：联系 Azure 支持。

## 限制

- Microsoft Dev Box 目前仅在 Windows 10 和 11 上可用（不支持 Linux 虚拟机）。
- 性能可能会根据您的 Dev Box 配置和网络条件而有所不同。
