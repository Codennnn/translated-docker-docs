---
description: 了解不可变标签及其在 Docker Hub 上保持镜像版本一致性的作用。
keywords: Docker Hub, Hub, 存储库内容, 标签, 不可变标签, 版本控制
title: Docker Hub 上的不可变标签
linkTitle: 不可变标签
weight: 11
---
{{< summary-bar feature_name="Immutable tags" >}}

不可变标签用于确保特定镜像版本一经发布至 Docker Hub 后即保持不变。
通过防止关键镜像版本被意外覆盖，该功能有助于在容器部署中维持一致性与可靠性。

## 什么是不可变标签？

不可变标签是指一旦推送到 Docker Hub 后，就不能被覆盖或删除的镜像标签。
这确保了镜像的特定版本在其生命周期内始终保持一致，带来以下好处：

- 版本一致性
- 可复现的构建
- 防止意外覆盖
- 更好的安全性与合规性

## 启用不可变标签

为存储库启用不可变标签：

1. 登录 [Docker Hub](https://hub.docker.com)。
2. 选择 **My Hub** > **Repositories**。
3. 选择你要启用不可变标签的存储库。
4. 进入 **Settings** > **General**。
5. 在 **Tag mutability settings** 下，选择以下其一：
   - **All tags are mutable (Default)**：
     标签可被更改为指向其他镜像，从而在无需新建标签的情况下重定向标签。
   - **All tags are immutable**：
     标签创建后不可再更新为指向其他镜像，以确保一致性并防止意外改动；`latest` 也包含在内。
   - **Specific tags are immutable**：
     使用正则表达式指定创建后不可更新的特定标签。
6. 点击 **Save**。

启用后，所有标签都会与其对应镜像“锁定”，确保每个标签始终指向同一镜像版本且不可修改。

> [!NOTE]
> 这里的正则表达式实现遵循 [Go regexp 包](https://pkg.go.dev/regexp)，其基于 RE2 引擎。
> 详见 [RE2 正则表达式语法](https://github.com/google/re2/wiki/Syntax)。

## 使用不可变标签

启用不可变标签后：

- 你不能再推送使用相同标签名的新镜像；
- 每个新的镜像版本都必须使用新的标签名。

要推送镜像，请为更新后的镜像创建一个新标签并推送到存储库。










