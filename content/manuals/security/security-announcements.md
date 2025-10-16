---
description: Docker 安全公告
keywords: Docker, CVEs, security, notice, Log4J 2, Log4Shell, Text4Shell, announcements, 安全, 公告
title: Docker 安全公告
linkTitle: 安全公告
outputs: ["HTML", "markdown", "RSS"]
type: "security-announcements"
weight: 80
toc_min: 1
toc_max: 2
---

{{< rss-button feed="/security/security-announcements/index.xml" text="订阅安全 RSS 提要" >}}

## Docker Desktop 4.47.0 安全更新：CVE-2025-10657

Docker Desktop 中的一个漏洞已在 9 月 25 日发布的 [4.47.0](/manuals/desktop/release-notes.md#4470) 版本中修复：

- 修复了 [CVE-2025-10657](https://www.cve.org/CVERecord?id=CVE-2025-10657)：在仅限 Docker Desktop 4.46.0 的情况下，增强型容器隔离（ECI）的 [Docker Socket 命令限制](../enterprise/security/hardened-desktop/enhanced-container-isolation/config.md#command-restrictions) 功能未正常生效（其配置被忽略）。

## Docker Desktop 4.44.3 安全更新：CVE-2025-9074

_最近更新：2025 年 8 月 20 日_

Docker Desktop 中的一个漏洞已在 8 月 20 日发布的 [4.44.3](/manuals/desktop/release-notes.md#4443) 版本中修复：

- 修复了 [CVE-2025-9074](https://www.cve.org/CVERecord?id=CVE-2025-9074)：在 Docker Desktop 上运行的恶意容器无需挂载 Docker socket 即可访问 Docker Engine 并启动额外容器，可能导致主机上的用户文件被未授权访问。增强型容器隔离（ECI）无法缓解此漏洞。


## Docker Desktop 4.44.0 安全更新：CVE-2025-23266

_最近更新：2025 年 7 月 31 日_

我们已注意到 [CVE-2025-23266](https://nvd.nist.gov/vuln/detail/CVE-2025-23266)，这是影响 NVIDIA Container Toolkit CDI 模式（至 1.17.7 版本）的严重漏洞。Docker Desktop 已内置 1.17.8 版本，不受影响。但较早版本的 Docker Desktop 若捆绑了旧版工具包，并且你曾手动启用 CDI 模式，则可能受影响。请升级至 Docker Desktop 4.44 或更高版本以确保使用修复版本。

## Docker Desktop 4.43.0 安全更新：CVE-2025-6587

_最近更新：2025 年 7 月 3 日_

Docker Desktop 中的一个漏洞已在 7 月 3 日发布的 [4.43.0](/manuals/desktop/release-notes.md#4430) 版本中修复：

- 修复了 [CVE-2025-6587](https://www.cve.org/CVERecord?id=CVE-2025-6587)：Docker Desktop 诊断日志中包含了敏感的系统环境变量，可能导致机密信息泄露。

## Docker Desktop 4.41.0 安全更新：CVE-2025-3224、CVE-2025-4095、CVE-2025-3911

_最近更新：2025 年 5 月 15 日_

Docker Desktop 中的三个漏洞已在 4 月 28 日发布的 [4.41.0](/manuals/desktop/release-notes.md#4410) 版本中修复。

- 修复了 [CVE-2025-3224](https://www.cve.org/CVERecord?id=CVE-2025-3224)：攻击者在拥有目标机器访问权限的情况下，可能在 Docker Desktop 更新时实现权限提升。
- 修复了 [CVE-2025-4095](https://www.cve.org/CVERecord?id=CVE-2025-4095)：当使用 macOS 配置描述文件时，镜像仓库访问管理（RAM）策略未被正确强制执行，导致用户可以从未批准的仓库拉取镜像。
- 修复了 [CVE-2025-3911](https://www.cve.org/CVERecord?id=CVE-2025-3911)：拥有目标机器读取权限的攻击者可能从 Docker Desktop 日志文件中获取敏感信息，包括为运行中的容器设置的环境变量。

我们强烈建议你升级到 Docker Desktop [4.41.0](/manuals/desktop/release-notes.md#4410)。

## Docker Desktop 4.34.2 安全更新：CVE-2024-8695 与 CVE-2024-8696

_最近更新：2024 年 9 月 13 日_

与 Docker 扩展相关的两项远程代码执行（RCE）漏洞由 [Cure53](https://cure53.de/) 报告，并已在 9 月 12 日发布的 [4.34.2](/manuals/desktop/release-notes.md#4342) 版本中修复。

- [CVE-2024-8695](https://www.cve.org/cverecord?id=CVE-2024-8695)：在 4.34.2 之前的 Docker Desktop 中，恶意扩展可以通过构造的扩展描述/变更日志实施远程代码执行（RCE）。【严重】
- [CVE-2024-8696](https://www.cve.org/cverecord?id=CVE-2024-8696)：在 4.34.2 之前的 Docker Desktop 中，恶意扩展可以通过构造的扩展发布者 URL/附加 URL 实施远程代码执行（RCE）。【高】

扩展市场中未发现利用这些漏洞的现有扩展。Docker 团队将持续密切监控并严格审核新的扩展发布请求。

我们强烈建议你升级到 Docker Desktop [4.34.2](/manuals/desktop/release-notes.md#4342)。如果暂时无法升级，可作为权宜之计先[禁用 Docker 扩展](/manuals/extensions/settings-feedback.md#turn-on-or-turn-off-extensions)。

## 当强制启用 SSO 时，弃用 CLI 中的密码登录

_最近更新：2024 年 7 月_

当首次引入[强制启用 SSO](/manuals/enterprise/security/single-sign-on/connect.md) 时，Docker 提供了一个过渡期，允许在 Docker CLI 认证到 Docker Hub 时继续使用密码，以便组织更容易过渡到 SSO 强制模式。建议配置 SSO 的管理员提前引导 CLI 用户[切换为使用个人访问令牌（PAT）](/manuals/enterprise/security/single-sign-on/_index.md#prerequisites)，为过渡期结束做准备。

自 2024 年 9 月 16 日起，过渡期结束；当 SSO 被强制启用时，将无法再通过 Docker CLI 使用密码登录 Docker Hub。受影响的用户需要改用 PAT 才能继续登录。

Docker 一直致力于为开发者与组织提供最安全的使用体验，此次弃用是实现该目标的重要一步。

## 通过 SOC 2 Type 2 鉴证与获得 ISO 27001 认证

_最近更新：2024 年 6 月_

Docker 很高兴地宣布，我们已顺利通过 SOC 2 Type 2 鉴证并获得 ISO 27001 认证，且无例外与重大不符合项。

安全性是 Docker 运营的基石，已深度融入我们的使命与公司战略。Docker 的产品是用户社区的核心，而 SOC 2 Type 2 鉴证与 ISO 27001 认证也再次表明了我们对安全的长期承诺。

了解更多，请参阅[博客公告](https://www.docker.com/blog/docker-announces-soc-2-type-2-attestation-iso-27001-certification/)。

## Docker 安全公告：runc、BuildKit 与 Moby 中的多个漏洞

_最近更新：2024 年 2 月 2 日_

Docker 高度重视软件的安全与完整性，以及用户对我们的信任。Snyk Labs 的安全研究人员在容器生态中识别并报告了四个安全漏洞。其中一个漏洞 [CVE-2024-21626](https://scout.docker.com/v/CVE-2024-21626) 涉及 runc 容器运行时，另外三个影响 BuildKit（[CVE-2024-23651](https://scout.docker.com/v/CVE-2024-23651)、[CVE-2024-23652](https://scout.docker.com/v/CVE-2024-23652)、[CVE-2024-23653](https://scout.docker.com/v/CVE-2024-23653)）。我们与报告方及开源维护者紧密协作，已积极协调并实施必要修复。

我们致力于保持最高的安全标准。1 月 31 日我们发布了包含修复的 runc、BuildKit 与 Moby 版本，并于 2 月 1 日发布了 Docker Desktop 更新以应对这些漏洞。此外，我们最新版本的 BuildKit 与 Moby 还包含了对 [CVE-2024-23650](https://scout.docker.com/v/CVE-2024-23650) 与 [CVE-2024-24557](https://scout.docker.com/v/CVE-2024-24557) 的修复，分别由独立研究者与 Docker 内部研究发现。

|                        | 受影响的版本               |
|:-----------------------|:--------------------------|
| `runc`                 | <= 1.1.11                 |
| `BuildKit`             | <= 0.12.4                 |
| `Moby (Docker Engine)` | <= 25.0.1 与 <= 24.0.8    |
| `Docker Desktop`       | <= 4.27.0                 |

### 如果我使用的是受影响版本，该怎么办？

如果你正在使用受影响版本的 runc、BuildKit、Moby 或 Docker Desktop，请尽快升级到下表所列的最新修复版本：

|                        | 修复版本                   |
|:-----------------------|:--------------------------|
| `runc`                 | >= [1.1.12](https://github.com/opencontainers/runc/releases/tag/v1.1.12)                 |
| `BuildKit`             | >= [0.12.5](https://github.com/moby/buildkit/releases/tag/v0.12.5)                 |
| `Moby (Docker Engine)` | >= [25.0.2](https://github.com/moby/moby/releases/tag/v25.0.2) 与 >= [24.0.9](https://github.com/moby/moby/releases/tag/v24.0.9)   |
| `Docker Desktop`       | >= [4.27.1](/manuals/desktop/release-notes.md#4271)                 |


如果暂时无法立即升级到不受影响的版本，请遵循以下最佳实践以降低风险：

* Only use trusted Docker images (such as [Docker Official Images](../docker-hub/image-library/trusted-content.md#docker-official-images)).
* Don’t build Docker images from untrusted sources or untrusted Dockerfiles.
* If you are a Docker Business customer using Docker Desktop and unable to update to v4.27.1, make sure to enable [Hardened Docker Desktop](/manuals/enterprise/security/hardened-desktop/_index.md) features such as:
  * [Enhanced Container Isolation](/manuals/enterprise/security/hardened-desktop/enhanced-container-isolation/_index.md), which mitigates the impact of CVE-2024-21626 in the case of running containers from malicious images.
  * [Image Access Management](/manuals/enterprise/security/hardened-desktop/image-access-management.md), and [Registry Access Management](/manuals/enterprise/security/hardened-desktop/registry-access-management.md), which give organizations control over which images and repositories their users can access.
* For CVE-2024-23650, CVE-2024-23651, CVE-2024-23652, and CVE-2024-23653, avoid using BuildKit frontend from an untrusted source. A frontend image is usually specified as the #syntax line on your Dockerfile, or with `--frontend` flag when using the `buildctl build` command.
* To mitigate CVE-2024-24557, make sure to either use BuildKit or disable caching when building images. From the CLI this can be done via the `DOCKER_BUILDKIT=1` environment variable (default for Moby >= v23.0 if the buildx plugin is installed) or the `--no-cache flag`. If you are using the HTTP API directly or through a client, the same can be done by setting `nocache` to `true` or `version` to `2` for the [/build API endpoint](https://docs.docker.com/reference/api/engine/version/v1.44/#tag/Image/operation/ImageBuild).

### 技术细节与影响

#### CVE-2024-21626（高）

In runc v1.1.11 and earlier, due to certain leaked file descriptors, an attacker can gain access to the host filesystem by causing a newly-spawned container process (from `runc exec`) to have a working directory in the host filesystem namespace, or by tricking a user to run a malicious image and allow a container process to gain access to the host filesystem through `runc run`. The attacks can also be adapted to overwrite semi-arbitrary host binaries, allowing for complete container escapes. Note that when using higher-level runtimes (such as Docker or Kubernetes), this vulnerability can be exploited by running a malicious container image without additional configuration or by passing specific workdir options when starting a container. The vulnerability can also be exploited from within Dockerfiles in the case of Docker.

_该问题已在 runc v1.1.12 中修复。_

#### CVE-2024-23651（高）

In BuildKit <= v0.12.4, two malicious build steps running in parallel sharing the same cache mounts with subpaths could cause a race condition, leading to files from the host system being accessible to the build container. This will only occur if a user is trying to build a Dockerfile of a malicious project.

_该问题已在 BuildKit v0.12.5 中修复。_

#### CVE-2024-23652（高）

In BuildKit <= v0.12.4, a malicious BuildKit frontend or Dockerfile using `RUN --mount` could trick the feature that removes empty files created for the mountpoints into removing a file outside the container from the host system. This will only occur if a user is using a malicious Dockerfile.

_该问题已在 BuildKit v0.12.5 中修复。_

#### CVE-2024-23653（高）

In addition to running containers as build steps, BuildKit also provides APIs for running interactive containers based on built images. In BuildKit <= v0.12.4, it is possible to use these APIs to ask BuildKit to run a container with elevated privileges. Normally, running such containers is only allowed if special `security.insecure` entitlement is enabled both by buildkitd configuration and allowed by the user initializing the build request.

_该问题已在 BuildKit v0.12.5 中修复。_

#### CVE-2024-23650（中）

In BuildKit <= v0.12.4, a malicious BuildKit client or frontend could craft a request that could lead to BuildKit daemon crashing with a panic.

_该问题已在 BuildKit v0.12.5 中修复。_

#### CVE-2024-24557（中）

In Moby <= v25.0.1 and <= v24.0.8, the classic builder cache system is prone to cache poisoning if the image is built FROM scratch. Also, changes to some instructions (most important being `HEALTHCHECK` and `ONBUILD`) would not cause a cache miss. An attacker with knowledge of the Dockerfile someone is using could poison their cache by making them pull a specially crafted image that would be considered a valid cache candidate for some build steps.

_该问题已在 Moby >= v25.0.2 与 >= v24.0.9 中修复。_

### Docker 产品受到怎样的影响？

#### Docker Desktop

Docker Desktop v4.27.0 and earlier are affected. Docker Desktop v4.27.1 was released on February 1 and includes runc, BuildKit, and dockerd binaries patches. In addition to updating to this new version, we encourage all Docker users to diligently use Docker images and Dockerfiles and ensure you only use trusted content in your builds.

As always, you should check Docker Desktop system requirements for your operating system ([Windows](/manuals/desktop/setup/install/windows-install.md#system-requirements), [Linux](/manuals/desktop/setup/install/linux/_index.md#general-system-requirements), [Mac](/manuals/desktop/setup/install/mac-install.md#system-requirements)) before updating to ensure full compatibility.

#### Docker Build Cloud

Any new Docker Build Cloud builder instances will be provisioned with the latest Docker Engine and BuildKit versions and will, therefore, be unaffected by these CVEs. Updates have also been rolled out to existing Docker Build Cloud builders.

_No other Docker products are affected by these vulnerabilities._

### 公告链接

* Runc
  * [CVE-2024-21626](https://github.com/opencontainers/runc/security/advisories/GHSA-xr7r-f8xq-vfvv)
* BuildKit
  * [CVE-2024-23650](https://github.com/moby/buildkit/security/advisories/GHSA-9p26-698r-w4hx)
  * [CVE-2024-23651](https://github.com/moby/buildkit/security/advisories/GHSA-m3r6-h7wv-7xxv)
  * [CVE-2024-23652](https://github.com/moby/buildkit/security/advisories/GHSA-4v98-7qmw-rqr8)
  * [CVE-2024-23653](https://github.com/moby/buildkit/security/advisories/GHSA-wr6v-9f75-vh2g)
* Moby
  * [CVE-2024-24557](https://github.com/moby/moby/security/advisories/GHSA-xw73-rw38-6vjc)

## Text4Shell CVE-2022-42889

_最近更新：2022 年 10 月_

[CVE-2022-42889](https://nvd.nist.gov/vuln/detail/CVE-2022-42889) has been discovered in the popular Apache Commons Text library. Versions of this library up to but not including 1.10.0 are affected by this vulnerability.

我们强烈建议你升级到最新版本的 [Apache Commons Text](https://commons.apache.org/proper/commons-text/download_text.cgi)。

### 在 Docker Hub 上扫描镜像

在 2021 年 10 月 21 日 12:00（UTC）之后触发的 Docker Hub 安全扫描，已能正确识别 Text4Shell CVE。此日期之前的扫描暂不能准确反映该漏洞状态。因此，我们建议通过推送新镜像触发扫描，以在漏洞报告中查看 Text4Shell CVE 的状态。详见[在 Docker Hub 上扫描镜像](../docker-hub/repos/manage/vulnerability-scanning.md)。

### 受 CVE-2022-42889 影响的 Docker 官方镜像

部分 [Docker 官方镜像](../docker-hub/image-library/trusted-content.md#docker-official-images)包含受影响版本的 Apache Commons Text。以下是可能包含受影响版本 Apache Commons Text 的 Docker 官方镜像：

- [bonita](https://hub.docker.com/_/bonita)
- [Couchbase](https://hub.docker.com/_/couchbase)
- [Geonetwork](https://hub.docker.com/_/geonetwork)
- [neo4j](https://hub.docker.com/_/neo4j)
- [sliverpeas](https://hub.docker.com/_/sliverpeas)
- [solr](https://hub.docker.com/_/solr)
- [xwiki](https://hub.docker.com/_/xwiki)

We have updated
Apache Commons Text in these images to the latest version. Some of these images may not be
vulnerable for other reasons. We recommend that you also review the guidelines published on the upstream websites.

## Log4j 2 CVE-2021-44228

_最近更新：2021 年 12 月_

[Log4j 2 CVE-2021-44228](https://nvd.nist.gov/vuln/detail/CVE-2021-44228) 是存在于常见 Java 日志库 Log4j 2 中的漏洞，可被远程执行代码，且往往发生在攻击者易于触达的上下文中。例如，它曾在 Minecraft 服务器中被发现：攻击者可在聊天内容中输入命令，被记录到日志后触发执行。由于该日志库的广泛使用与利用门槛低，此漏洞影响极其严重。许多开源维护者正努力为软件生态提供修复与更新。

受影响的 Log4j 2 版本为 2.0 至 2.14.1（含）。首个修复版本为 2.15.0。我们强烈建议你升级到[最新版本](https://logging.apache.org/log4j/2.x/download.html)。如果你使用的是 2.0 之前的版本，则不受影响。

即便使用受影响范围内的版本，你也可能因配置已做缓解或记录内容不包含用户输入而不受影响。但若不全面理解所有日志路径及其输入来源，很难进行准确验证。因此，我们仍建议升级所有使用受影响版本的代码。

> CVE-2021-45046
>
> 作为对 [CVE-2021-44228](https://nvd.nist.gov/vuln/detail/CVE-2021-44228) 的补充说明，2.15.0 的修复并不完整。又发现了其他问题，分别记录为 [CVE-2021-45046](https://nvd.nist.gov/vuln/detail/CVE-2021-45046) 与 [CVE-2021-45105](https://nvd.nist.gov/vuln/detail/CVE-2021-45105)。若想更全面地修复该漏洞，建议尽可能升级至 2.17.0。

### 在 Docker Hub 上扫描镜像

在 2021 年 12 月 13 日 17:00（UTC）之后触发的 Docker Hub 安全扫描，已能正确识别 Log4j 2 的相关 CVE。此日期之前的扫描暂不能准确反映该漏洞状态。因此，我们建议通过推送新镜像触发扫描，以在漏洞报告中查看 Log4j 2 CVE 的状态。详见[在 Docker Hub 上扫描镜像](../docker-hub/repos/manage/vulnerability-scanning.md)。

## 受 Log4j 2 CVE 影响的 Docker 官方镜像

_Last updated December 2021_

部分 [Docker 官方镜像](../docker-hub/image-library/trusted-content.md#docker-official-images)包含受影响版本的 Log4j 2（CVE-2021-44228）。下表列出了可能包含受影响版本的 Docker 官方镜像。我们已将这些镜像中的 Log4j 2 升级至最新版本。部分镜像可能因为其他原因并不受影响。建议同时参阅上游网站发布的指引。

| Repository                | Patched version         | Additional documentation       |
|:------------------------|:-----------------------|:-----------------------|
| [couchbase](https://hub.docker.com/_/couchbase)    | 7.0.3 | [Couchbase blog](https://blog.couchbase.com/what-to-know-about-the-log4j-vulnerability-cve-2021-44228/) |
| [Elasticsearch](https://hub.docker.com/_/elasticsearch)    | 6.8.22, 7.16.2 | [Elasticsearch announcement](https://www.elastic.co/blog/new-elasticsearch-and-logstash-releases-upgrade-apache-log4j2) |
| [Flink](https://hub.docker.com/_/flink)    | 1.11.6, 1.12.7, 1.13.5, 1.14.2  | [Flink advice on Log4j CVE](https://flink.apache.org/2021/12/10/log4j-cve.html) |
| [Geonetwork](https://hub.docker.com/_/geonetwork)    | 3.10.10 | [Geonetwork GitHub discussion](https://github.com/geonetwork/core-geonetwork/issues/6076) |
| [lightstreamer](https://hub.docker.com/_/lightstreamer)     | Awaiting info | Awaiting info  |
| [logstash](https://hub.docker.com/_/logstash)    | 6.8.22, 7.16.2 | [Elasticsearch announcement](https://www.elastic.co/blog/new-elasticsearch-and-logstash-releases-upgrade-apache-log4j2) |
| [neo4j](https://hub.docker.com/_/neo4j)     | 4.4.2 | [Neo4j announcement](https://community.neo4j.com/t/log4j-cve-mitigation-for-neo4j/48856) |
| [solr](https://hub.docker.com/_/solr)    | 8.11.1 | [Solr security news](https://solr.apache.org/security.html#apache-solr-affected-by-apache-log4j-cve-2021-44228) |
| [sonarqube](https://hub.docker.com/_/sonarqube)    | 8.9.5, 9.2.2 | [SonarQube announcement](https://community.sonarsource.com/t/sonarqube-sonarcloud-and-the-log4j-vulnerability/54721) |
| [storm](https://hub.docker.com/_/storm)    | Awaiting info | Awaiting info |

> [!NOTE]
>
> 尽管某些扫描器可能会将 [xwiki](https://hub.docker.com/_/xwiki) 镜像识别为受影响，但镜像作者认为其并不受 Log4j 2 CVE 影响，因为 API JAR 并不包含漏洞。
> [Nuxeo](https://hub.docker.com/_/nuxeo) 镜像已弃用且不再更新。
