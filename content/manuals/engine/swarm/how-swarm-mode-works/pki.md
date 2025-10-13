---
description: 了解 Swarm 模式中的 PKI 工作方式
keywords: swarm, security, tls, pki
title: 使用公钥基础设施（PKI）管理 Swarm 安全
---

Docker 内置的 Swarm 模式公钥基础设施（PKI）系统，让你能够更简便、安全地部署容器编排系统。swarm 中的各节点通过双向 TLS（传输层安全）实现相互间的身份验证、授权与通信加密。

当你运行 `docker swarm init` 创建 swarm 时，Docker 会将自身设为管理节点。默认情况下，该管理节点会生成新的根 CA（证书颁发机构）及其密钥对，用于与加入集群的其他节点进行安全通信。你也可以选择自带根 CA：在 [`docker swarm init`](/reference/cli/docker/swarm/init.md) 中使用 `--external-ca`。

管理节点还会生成两类用于加入集群的令牌：工作节点令牌与管理节点令牌。每个令牌都包含根 CA 证书的摘要以及一个随机生成的密钥。新节点加入时，会使用该摘要验证来自远端管理节点的根 CA 证书；远端管理节点则使用该密钥确认该加入节点被允许加入。

每当有新节点加入，管理节点都会为其签发证书。证书中包含随机生成的节点 ID（作为证书通用名 CN）与节点角色（作为组织单位 OU）。在该节点的整个生命周期内，这个节点 ID 作为其加密安全身份标识。

下图展示了管理节点与工作节点如何使用至少 TLS 1.2 进行加密通信。

![TLS diagram](/engine/swarm/images/tls.webp?w=600)

下面示例展示了某个工作节点证书中的信息：

```text
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            3b:1c:06:91:73:fb:16:ff:69:c3:f7:a2:fe:96:c1:73:e2:80:97:3b
        Signature Algorithm: ecdsa-with-SHA256
        Issuer: CN=swarm-ca
        Validity
            Not Before: Aug 30 02:39:00 2016 GMT
            Not After : Nov 28 03:39:00 2016 GMT
        Subject: O=ec2adilxf4ngv7ev8fwsi61i7, OU=swarm-worker, CN=dw02poa4vqvzxi5c10gm4pq2g
...snip...
```

默认情况下，swarm 中的每个节点会每三个月续期一次证书。你可以运行 `docker swarm update --cert-expiry <TIME PERIOD>` 配置该间隔；最小轮换值为 1 小时。详见 [`docker swarm update`](/reference/cli/docker/swarm/update.md)。

## 轮换 CA 证书

> [!NOTE]
>
> Mirantis Kubernetes Engine（MKE，原 Docker UCP）为 swarm 提供外部证书管理服务。若你的 swarm 运行在 MKE 上，请不要手动轮换 CA 证书；如需轮换，请联系 Mirantis 支持。

当集群的 CA 私钥或某个管理节点遭到入侵时，你可以轮换 swarm 根 CA，使集群中的节点不再信任由旧根 CA 签发的证书。

运行 `docker swarm ca --rotate` 可生成新的 CA 证书与密钥。你也可以通过 `--ca-cert` 与 `--external-ca` 指定外部根证书；或者使用 `--ca-cert` 与 `--ca-key` 明确指定证书与密钥文件。

当你执行 `docker swarm ca --rotate` 时，按以下顺序发生：

1.  Docker 生成一个交叉签名证书：即用旧根 CA 对新根 CA 证书的某个版本进行签名。该交叉签名证书作为所有新节点证书的中间证书，确保仍信任旧根 CA 的节点也能验证由新根 CA 签发的证书。

2.  Docker 通知所有节点立即更新其 TLS 证书。该过程可能需要数分钟，具体取决于集群节点数量。

3.  当集群内所有节点均换发了由新 CA 签署的 TLS 证书后，Docker 会“遗忘”旧 CA 的证书与密钥材料，并通知所有节点仅信任新 CA 证书。与此同时，swarm 的加入令牌也会发生变化，旧令牌不再有效。

从此之后，所有新签发的节点证书都由新根 CA 直接签署，不再包含中间证书。

## 延伸阅读

* 了解[节点](nodes.md)的工作方式。
* 了解 Swarm 模式下[服务](services.md)的工作方式。
