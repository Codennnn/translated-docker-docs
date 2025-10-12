---
title: 指定项目名
weight: 10
description: 了解如何在 Compose 中自定义项目名，以及各类设置方式的优先级。
keywords: name, compose, project, -p flag, name top-level element
aliases:
- /compose/project-name/
---

默认情况下，Compose 会根据包含 Compose 文件的目录名来分配项目名。你可以通过多种方式覆盖该默认值。

本文通过示例说明在哪些场景下自定义项目名更有帮助，梳理设置项目名的可选方法，并给出各方法的优先级顺序。

> [!NOTE]
>
> 默认的项目目录是 Compose 文件所在的基目录。你也可以使用 [`--project-directory` 命令行选项](/reference/cli/docker/compose.md#options)为其设置自定义路径。

## 示例用例

Compose 使用项目名来隔离彼此独立的环境。以下场景中，项目名尤为有用：

- 在开发主机上：为同一环境创建多个副本，便于为项目的每个功能分支运行稳定的环境实例。
- 在 CI 服务器上：将项目名设置为唯一的构建号，防止不同构建之间相互干扰。
- 在共享或开发主机上：当不同项目可能使用相同的服务名时，通过项目名避免彼此影响。

## 设置项目名

项目名只能包含小写字母、十进制数字、连字符和下划线，并且必须以小写字母或十进制数字开头。如果项目目录或当前目录的基本名称不满足上述约束，可以采用其他方式进行设置。

各种设置方式的优先级（从高到低）如下：

1. `-p` 命令行标志。
2. `COMPOSE_PROJECT_NAME` 环境变量（见 [环境变量](environment-variables/envvars.md)）。
3. Compose 文件顶层的 `name:` 属性；或当你在命令行使用 `-f` 指定了多个 Compose 文件时，以最后一个文件中的 `name:` 为准（见 [`version` 与 `name`](/reference/compose-file/version-and-name.md) 以及 [合并多个 Compose 文件](multiple-compose-files/merge.md)）。
4. 包含 Compose 文件的项目目录的基本名称；或当你在命令行使用 `-f` 指定了多个 Compose 文件时，以第一个 Compose 文件的基本名称为准（见 [合并多个 Compose 文件](multiple-compose-files/merge.md)）。
5. 如果未指定 Compose 文件，则为当前目录的基本名称。

## 下一步

- 继续阅读：[使用多个 Compose 文件](multiple-compose-files/_index.md)。
- 查看一些[示例应用](samples-for-compose.md)。
