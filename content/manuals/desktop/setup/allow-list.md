---
description: 在组织环境中，为确保 Docker Desktop 正常工作所需放行的域名列表。
keywords: Docker Desktop, allowlist, 允许列表, 防火墙, 鉴权 URL, 分析
title: Docker Desktop 允许列表（Allowlist）
tags: [admin]
linkTitle: Allowlist
weight: 100
aliases:
  - /desktop/allow-list/
---

{{< summary-bar feature_name="Allow list" >}}

本页列出了需要加入防火墙允许列表（allowlist）的域名 URL，以确保 Docker Desktop 在你所在组织内可以正常工作。

## 需要放行的域名 URL

| Domains                                                                              | 描述                                         |
| ------------------------------------------------------------------------------------ | -------------------------------------------- |
| https://api.segment.io                                                               | 分析                                         |
| https://cdn.segment.com                                                              | 分析                                         |
| https://notify.bugsnag.com                                                           | 错误上报                                     |
| https://sessions.bugsnag.com                                                         | 错误上报                                     |
| https://auth.docker.io                                                               | 身份验证                                     |
| https://cdn.auth0.com                                                                | 身份验证                                     |
| https://login.docker.com                                                             | 身份验证                                     |
| https://auth.docker.com                                                              | 身份验证                                     |
| https://desktop.docker.com                                                           | 更新                                         |
| https://hub.docker.com                                                               | Docker Hub                                   |
| https://registry-1.docker.io                                                         | 镜像拉取/推送                                |
| https://production.cloudflare.docker.com                                             | 镜像拉取/推送（付费方案）                    |
| https://docker-images-prod.6aa30f8b08e16409b46e0173d6de2f56.r2.cloudflarestorage.com | 镜像拉取/推送（个人/匿名）                   |
| https://docker-pinata-support.s3.amazonaws.com                                       | 故障排查                                     |
| https://api.dso.docker.com                                                           | Docker Scout 服务                            |
| https://api.docker.com                                                               | 新版 API                                     |
