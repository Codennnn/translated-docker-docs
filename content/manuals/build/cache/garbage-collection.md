---
title: 构建垃圾回收（GC）
description: 了解 BuildKit 守护进程中的垃圾回收机制
keywords: build, buildx, buildkit, garbage collection, prune, gc
aliases:
  - /build/building/cache/garbage-collection/
---

与一次性执行的 [`docker builder prune`](/reference/cli/docker/builder/prune.md)
或 [`docker buildx prune`](/reference/cli/docker/buildx/prune.md) 不同，
垃圾回收（GC）会周期性运行，并遵循一组有序的清理策略。当前者达到阈值（体积过大或超过存活时间）时，
BuildKit 守护进程会清理构建缓存。

对大多数用户而言，默认 GC 行为已经足够，无需手动干预。
而对大型构建、自建构建器或存储受限的场景，高级用户可以根据自身流程进行定制配置。
下文将介绍 GC 的工作方式，并说明如何通过自定义配置调整其行为。

## 垃圾回收策略

GC 策略定义了如何管理与清理构建缓存的一组规则，包括：何时移除（基于缓存年龄）、
占用空间阈值、需要清理的缓存记录类型等。

每条 GC 策略会按顺序评估。先评估更具体的规则，若未能释放足够空间，再逐步应用更宽泛的规则。
如此可优先保留更有价值的缓存，同时保证系统性能与可用性。

例如，假设你有如下 GC 策略：

1. 查找过去 48 小时未被使用的“过期”缓存记录，并删除，直至“过期”缓存最多仅剩 5GB。
2. 若构建缓存总量超过 10GB，则删除记录直至总量不超过 10GB。

第一条规则更具体，优先清理“过期”缓存，并对这类价值较低的缓存设定了较低上限。
第二条规则给所有类型的缓存施加了更高的硬性上限。
在这些策略下，若你的构建缓存为 11GB，其中：

- 7GB 为“过期”缓存
- 4GB 为其他更有价值的缓存

一次 GC 将按第一条策略删除 5GB 过期缓存，剩余 6GB，因此无需再触发第二条策略。

默认 GC 策略（近似）如下：

1. 清理 48 小时未使用、且易于再生的缓存，例如来自本地目录或远程 Git 仓库的构建上下文，及缓存挂载。
2. 清理 60 天未用于构建的缓存。
3. 清理超过构建缓存大小上限的“非共享”缓存（指未被其他资源使用的层 Blob，通常为镜像层之外的缓存）。
4. 清理任何超过缓存大小上限的构建缓存。

具体算法与配置方式会随构建器类型略有差异，详见[配置](#configuration)。

## 配置

> [!NOTE]
> 如果你对默认 GC 行为感到满意且无需微调，可以跳过本节。默认配置适用于大多数场景，无需额外设置。

根据你使用的[构建驱动](../builders/drivers/_index.md)类型不同，需要通过不同的配置文件调整 GC 设置：

- 若使用 Docker Engine 的默认构建器（`docker` 驱动），请修改[Docker 守护进程配置文件](#docker-daemon-configuration-file)。
- 若使用自定义构建器，请使用 [BuildKit 配置文件](#buildkit-configuration-file)。

### Docker 守护进程配置文件

如果你使用默认的 [`docker` 驱动](../builders/drivers/docker.md)，
可以在 [`daemon.json` 配置文件](/reference/cli/dockerd.md#daemon-configuration-file) 中配置 GC；
若使用 Docker Desktop，则可在 [Settings > Docker Engine](/manuals/desktop/settings-and-maintenance/settings.md) 中配置。

以下片段展示了 Docker Desktop 用户在 `docker` 驱动下的默认构建器配置：

```json
{
  "builder": {
    "gc": {
      "defaultKeepStorage": "20GB",
      "enabled": true
    }
  }
}
```

`defaultKeepStorage` 用于设置构建缓存的体积上限，会影响 GC 策略。
`docker` 驱动的默认策略如下：

1. 若短期、未使用的构建缓存超过 `defaultKeepStorage` 的 13.8%（或至少 512MB），且已超过 48 小时，则清理。
2. 清理 60 天未使用的构建缓存。
3. 清理超过 `defaultKeepStorage` 上限的非共享构建缓存。
4. 清理任何超过 `defaultKeepStorage` 上限的构建缓存。

以 Docker Desktop 默认的 `defaultKeepStorage`=20GB 为例，默认 GC 策略可具体化为：

```json
{
  "builder": {
    "gc": {
      "enabled": true,
      "policy": [
        {
          "keepStorage": "2.764GB",
          "filter": [
            "unused-for=48h",
            "type==source.local,type==exec.cachemount,type==source.git.checkout"
          ]
        },
        { "keepStorage": "20GB", "filter": ["unused-for=1440h"] },
        { "keepStorage": "20GB" },
        { "keepStorage": "20GB", "all": true }
      ]
    }
  }
}
```

针对 `docker` 驱动，最简便的调优方式是调整 `defaultKeepStorage`：

- 如果 GC 过于激进，可适当增大该值。
- 如需节省空间，可适当减小该值。

若需更细粒度控制，可以直接自定义 GC 策略。
下面示例定义了更保守的 GC 配置：

1. 当构建缓存超过 50GB 时，删除超过 1440 小时（60 天）未使用的缓存项。
2. 当构建缓存超过 50GB 时，删除非共享缓存项。
3. 当构建缓存超过 100GB 时，删除任何缓存项。

```json
{
  "builder": {
    "gc": {
      "enabled": true,
      "defaultKeepStorage": "50GB",
      "policy": [
        { "keepStorage": "0", "filter": ["unused-for=1440h"] },
        { "keepStorage": "0" },
        { "keepStorage": "100GB", "all": true }
      ]
    }
  }
}
```

上述策略 1 与 2 的 `keepStorage` 均为 `0`，表示会回退到 `defaultKeepStorage` 指定的默认上限 50GB。

### BuildKit 配置文件

对于非 `docker` 驱动，需通过 [`buildkitd.toml`](../buildkit/toml-configuration.md) 配置 GC。
该文件提供以下高阶选项，用于调节 BuildKit 缓存可使用的磁盘阈值：

| Option          | Description                                                                                                                                             | Default value                                         |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------- |
| `reservedSpace` | BuildKit 可用于缓存的最小磁盘空间。低于此阈值时，GC 不会回收。                                                                 | 总磁盘的 10% 或 10GB（取较小者）                      |
| `maxUsedSpace`  | BuildKit 可使用的最大磁盘空间。超过此阈值会触发 GC 回收。                                                                                               | 总磁盘的 60% 或 100GB（取较小者）                     |
| `minFreeSpace`  | 必须保持空闲的磁盘空间。                                                                                                                                | 20GB                                                  |

这些选项可以使用字节数、带单位的字符串（如 `512MB`），或占总磁盘的百分比来设置。
阈值变化会影响 BuildKit worker 的默认 GC 策略。以默认阈值为例，策略具体化如下：

```toml
# Global defaults
[worker.oci]
  gc = true
  reservedSpace = "10GB"
  maxUsedSpace = "100GB"
  minFreeSpace = "20%"

# Policy 1
[[worker.oci.gcpolicy]]
  filters = [ "type==source.local", "type==exec.cachemount", "type==source.git.checkout" ]
  keepDuration = "48h"
  maxUsedSpace = "512MB"

# Policy 2
[[worker.oci.gcpolicy]]
  keepDuration = "1440h" # 60 days
  reservedSpace = "10GB"
  maxUsedSpace = "100GB"

# Policy 3
[[worker.oci.gcpolicy]]
  reservedSpace = "10GB"
  maxUsedSpace = "100GB"

# Policy 4
[[worker.oci.gcpolicy]]
  all = true
  reservedSpace = "10GB"
  maxUsedSpace = "100GB"
```

实际含义如下：

- Policy 1: If the build cache exceeds 512MB, BuildKit removes cache records
  for local build contexts, remote Git contexts, and cache mounts that haven’t
  been used in the last 48 hours.
- Policy 2: If disk usage exceeds 100GB, unshared build cache older than 60
  days is removed, ensuring at least 10GB of disk space is reserved for cache.
- Policy 3: If disk usage exceeds 100GB, any unshared cache is removed,
  ensuring at least 10GB of disk space is reserved for cache.
- Policy 4: If disk usage exceeds 100GB, all cache—including shared and
  internal records—is removed, ensuring at least 10GB of disk space is reserved
  for cache.

在定义缓存下限时，`reservedSpace` 优先级最高。
即便 `maxUsedSpace` 或 `minFreeSpace` 定义了更低的值，缓存下限也不会低于 `reservedSpace`。

当同时设置了 `reservedSpace` 与 `maxUsedSpace`，一次 GC 结束后缓存体积会处于两者之间。
例如，若 `reservedSpace`=10GB、`maxUsedSpace`=20GB，则 GC 后缓存体积小于 20GB、但不少于 10GB。

你也可以完全自定义 GC 策略，并借助过滤器精确指定可清理的缓存记录类型。

#### 在 BuildKit 中自定义 GC 策略

自定义策略可以细化 BuildKit 的缓存管理，让你基于缓存类型、保留时长或磁盘阈值等维度实现全面控制。
如果需要完全掌控阈值以及缓存优先级，自定义 GC 策略是最佳途径。

要定义自定义 GC 策略，请在 `buildkitd.toml` 中使用 `[[worker.oci.gcpolicy]]` 配置块。
每条策略定义其自身的阈值；一旦启用自定义策略，全局的 `reservedSpace`、`maxUsedSpace`、`minFreeSpace` 将不再生效。

示例配置如下：

```toml
# Custom GC Policy 1: Remove unused local contexts older than 24 hours
[[worker.oci.gcpolicy]]
  filters = ["type==source.local"]
  keepDuration = "24h"
  reservedSpace = "5GB"
  maxUsedSpace = "50GB"

# Custom GC Policy 2: Remove remote Git contexts older than 30 days
[[worker.oci.gcpolicy]]
  filters = ["type==source.git.checkout"]
  keepDuration = "720h"
  reservedSpace = "5GB"
  maxUsedSpace = "30GB"

# Custom GC Policy 3: Aggressively clean all cache if disk usage exceeds 90GB
[[worker.oci.gcpolicy]]
  all = true
  reservedSpace = "5GB"
  maxUsedSpace = "90GB"
```

除了 `reservedSpace`、`maxUsedSpace`、`minFreeSpace` 外，定义 GC 策略时还有两个额外选项：

- `all`：默认情况下，BuildKit 会在 GC 时排除部分缓存记录。将其设为 `true` 表示允许清理任何缓存记录。
- `filters`：用于指定某条 GC 策略允许清理的缓存记录类型。
