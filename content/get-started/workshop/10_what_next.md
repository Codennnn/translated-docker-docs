---
title: Docker 入门工作坊结束后可以做什么
weight: 100
linkTitle: "第 9 部分：接下来做什么"
keywords: 入门, 安装, 导览, 快速开始, 介绍, 概念, 容器,
  Docker Desktop
description: 帮你发掘应用接下来可以尝试与扩展的方向
aliases:
 - /get-started/11_what_next/
 - /guides/workshop/10_what_next/
---

尽管你已经完成了本次工作坊，关于容器还有很多值得继续学习的内容。

下面是一些推荐的下一步方向。

## 容器编排

在生产环境运行容器并不轻松。你不会想登录到某台机器上仅仅执行
`docker run` 或 `docker compose up`。为什么？容器挂了怎么办？如何在多台机器上扩缩容？
容器编排正是为了解决这些问题。Kubernetes、Swarm、Nomad、ECS 等工具都能提供帮助，但实现方式各有不同。

基本思路是：集群中有一组管理者接收「期望状态」。例如：「我要运行两个我的 Web 应用实例，并开放 80 端口。」
管理者随后会查看集群中的所有机器，把任务分配给工作节点。管理者会持续监控变化（例如某个容器退出），并努力让「实际状态」与「期望状态」保持一致。

## 云原生计算基金会（CNCF）项目

CNCF 是一个中立的开源项目之家，涵盖 Kubernetes、Prometheus、Envoy、Linkerd、NATS 等。
你可以查看已毕业与孵化中的[项目列表](https://www.cncf.io/projects/)，以及完整的
[CNCF Landscape](https://landscape.cncf.io/)。这些项目覆盖监控、日志、安全、镜像仓库、消息等多个领域，帮助你解决各类问题。

## 入门视频工作坊

Docker 推荐观看 DockerCon 2022 的视频工作坊。你可以完整观看，或通过以下链接直接跳转到对应章节：

* [Docker 概览与安装](https://youtu.be/gAGEar5HQoU)
* [拉取、运行并探索容器](https://youtu.be/gAGEar5HQoU?t=1400)
* [构建镜像](https://youtu.be/gAGEar5HQoU?t=3185)
* [将应用容器化](https://youtu.be/gAGEar5HQoU?t=4683)
* [连接数据库并设置绑定挂载](https://youtu.be/gAGEar5HQoU?t=6305)
* [将容器部署到云端](https://youtu.be/gAGEar5HQoU?t=8280)

<iframe src="https://www.youtube-nocookie.com/embed/gAGEar5HQoU" style="max-width: 100%; aspect-ratio: 16 / 9;" width="560" height="auto" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## 从零实现一个容器

如果你想了解容器是如何从零构建的，Aqua Security 的 Liz Rice 有一个精彩的演讲，她使用 Go 语言从头实现了一个容器。尽管该演讲没有深入涉及网络、使用镜像作为文件系统等更高级的话题，但它能帮助你更深入地理解容器的工作原理。

<iframe src="https://www.youtube-nocookie.com/embed/8fi7uSYlOdc" style="max-width: 100%; aspect-ratio: 16 / 9;" width="560" height="auto" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
