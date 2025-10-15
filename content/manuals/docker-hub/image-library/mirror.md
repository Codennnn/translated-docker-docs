---
description: 配置 Docker Hub 镜像的本地镜像站
keywords: 注册表, 本地部署, 镜像, 标签, 仓库, 分发, 镜像站, Hub,
  配置, 高级
title: 镜像 Docker Hub 内容库
linkTitle: 镜像站
weight: 80
aliases:
- /engine/admin/registry_mirror/
- /registry/recipes/mirror/
- /docker-hub/mirror/
---

## 适用场景

如果你的环境中运行了多个 Docker 实例（例如多台物理机或虚拟机都在运行 Docker），每个守护进程在本地没有所需镜像时，都会从互联网上的 Docker 仓库拉取镜像。你可以部署一个本地注册表镜像站（registry mirror），并将所有守护进程指向该镜像站，从而避免额外的外网流量。

> [!NOTE]
>
> Docker 官方镜像是 Docker 的知识产权。

### 替代方案

如果你所使用的镜像集合范围明确，也可以选择手动拉取这些镜像，并推送到一个简单的本地私有注册表。

另外，如果你的镜像全部由内部构建，不依赖 Docker Hub，那么完全依赖本地注册表是最简单的方案。

### 注意事项

目前无法对其他私有注册表进行镜像；只能镜像中央 Hub。

> [!NOTE]
>
> Docker Hub 的镜像站同样受制于 Docker 的[公平使用政策](/manuals/docker-hub/usage/_index.md#fair-use)。

### 解决方案

Registry 可以配置为“拉取透传缓存”（pull-through cache）模式。在该模式下，Registry 会响应常规的 `docker pull` 请求，并将所有内容缓存在本地。

### 在镜像站中使用 Registry Access Management（RAM）

如果你的 Registry Access Management（RAM）配置限制了对 Docker Hub 的访问，即便这些镜像在你的镜像站中可用，也无法拉取源自 Docker Hub 的镜像。

你会遇到如下错误：
```console
Error response from daemon: Access to docker.io has been restricted by your administrators.
```

如果无法放通对 Docker Hub 的访问，你可以从镜像站手动拉取，并可选地为镜像重新打标签。例如：
```console
docker pull <your-registry-mirror>[:<port>]/library/busybox
docker tag <your-registry-mirror>[:<port>]/library/busybox:latest busybox:latest
```

## 工作原理

当你第一次从本地镜像站请求某个镜像时，镜像站会先从公共 Docker 注册表拉取该镜像并存储在本地，然后再返回给你。随后再次请求该镜像时，本地镜像站可直接从自身存储中提供服务。

### 如果 Hub 上的内容发生变化怎么办？

当以标签（tag）拉取时，Registry 会检查远端以确认当前是否为所请求内容的最新版本；若不是，则会获取并缓存最新内容。

### 磁盘占用怎么办？

在高变更率的环境中，缓存中可能会累积陈旧数据。以透传缓存模式运行时，Registry 会定期清理旧内容以节省磁盘空间。之后再次请求被清理的内容时，会触发远端拉取并重新缓存到本地。

为获得最佳性能并确保正确性，建议将 Registry 缓存配置为使用 `filesystem` 存储驱动。

## 将 Registry 以透传缓存方式运行

将 Registry 作为透传缓存运行的最简单方式，是直接运行官方的[Registry](https://hub.docker.com/_/registry) 镜像。至少需要在 `/etc/docker/registry/config.yml` 中指定 `proxy.remoteurl`，如下小节所述。

可在同一后端上部署多个注册表缓存。单个缓存可以确保并发请求不会重复拉取数据，但缓存集群不具备这一特性。

### 配置缓存

要将 Registry 配置为透传缓存，需要在配置文件中新增 `proxy` 段。

若需访问 Docker Hub 上的私有镜像，可以提供用户名与密码。

```yaml
proxy:
  remoteurl: https://registry-1.docker.io
  username: [username]
  password: [password]
```

> [!WARNING]
>
> 若你配置了用户名与密码，务必理解：该用户在 Docker Hub 有权访问的私有资源将可通过你的镜像站访问。若希望这些资源保持私密，必须为镜像站实现访问控制与鉴权！

> [!WARNING]
>
> 要让调度器清理旧条目，必须在注册表配置中启用 `delete`。

### 配置 Docker 守护进程

可以在手动启动 `dockerd` 时传入 `--registry-mirror` 选项，或编辑 [`/etc/docker/daemon.json`](/reference/cli/dockerd.md#daemon-configuration-file)，添加 `registry-mirrors` 键值以持久化该配置。

```json
{
  "registry-mirrors": ["https://<my-docker-mirror-host>"]
}
```

保存文件并重载 Docker 使变更生效。

> [!NOTE]
>
> 某些看起来像错误的日志实际上只是信息级消息。
>
> 可以通过日志中的 `level` 字段判断是错误/告警还是仅供参考的信息。例如，下面这条日志就是信息级：
>
> ```text
> time="2017-06-02T15:47:37Z" level=info msg="error statting local store, serving from upstream: unknown blob" go.version=go1.7.4
> ```
>
> 它表示该文件尚未存在于本地缓存，正在从上游拉取。
