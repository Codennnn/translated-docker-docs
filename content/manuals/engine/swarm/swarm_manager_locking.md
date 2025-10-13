---
description: 自动锁定 Swarm 管理节点以保护加密密钥
keywords: swarm, manager, lock, unlock, autolock, encryption
title: 锁定 Swarm 以保护其加密密钥
---

Swarm 管理节点使用的 Raft 日志默认以加密形式落盘（静态加密）。这可以在攻击者获取到加密的 Raft 日志时，仍然保护你的服务配置与数据安全。引入该特性的原因之一，是为了支持 [Docker Secrets](secrets.md)。

当 Docker 重启时，用于加密 Swarm 节点间通信的 TLS 密钥，以及用于对磁盘上 Raft 日志进行加/解密的密钥，都会被加载到每个管理节点的内存中。Docker 提供了一种机制来保护这些密钥：你可以接管密钥的“所有权”，并要求在管理节点重启后进行手动解锁。这一特性称为“自动锁定”（autolock）。

当 Docker 重启时，你必须先使用 Swarm 被锁定时 Docker 生成的“密钥加密密钥”（key encryption key），来[解锁 Swarm](#unlock-a-swarm)。你可以随时轮换这一密钥。

> [!NOTE]
>
> 当有新节点加入 Swarm 时，无需解锁 Swarm；密钥会通过双向 TLS 传播至该节点。

## 初始化 Swarm 并启用自动锁定

初始化新 Swarm 时，可使用 `--autolock` 以在 Docker 重启后自动锁定各管理节点。

```console
$ docker swarm init --autolock

Swarm initialized: current node (k1q27tfyx9rncpixhk69sa61v) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join \
    --token SWMTKN-1-0j52ln6hxjpxk2wgk917abcnxywj3xed0y8vi1e5m9t3uttrtu-7bnxvvlz2mrcpfonjuztmtts9 \
    172.31.46.109:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.

To unlock a swarm manager after it restarts, run the `docker swarm unlock`
command and provide the following key:

    SWMKEY-1-WuYH/IX284+lRcXuoVf38viIDK3HJEKY13MIHX+tTt8
```

请妥善保管该密钥（例如存入密码管理器）。

当 Docker 重启时，你需要先[解锁 Swarm](#unlock-a-swarm)。若 Swarm 处于锁定状态，你在启动或重启服务时会看到如下错误：

```console
$ sudo service docker restart

$ docker service ls

Error response from daemon: Swarm is encrypted and needs to be unlocked before it can be used. Use "docker swarm unlock" to unlock it.
```

## 在现有 Swarm 上启用或关闭自动锁定

要在现有 Swarm 上启用自动锁定，将 `autolock` 设为 `true`：

```console
$ docker swarm update --autolock=true

Swarm updated.
To unlock a swarm manager after it restarts, run the `docker swarm unlock`
command and provide the following key:

    SWMKEY-1-+MrE8NgAyKj5r3NcR4FiQMdgu+7W72urH0EZeSmP/0Y

请务必将该密钥保存到密码管理器；没有它将无法重启管理节点。
```

若要关闭自动锁定，将 `--autolock` 设为 `false`。此时用于双向 TLS 的密钥以及读写 Raft 日志的加密密钥会以明文形式存储在磁盘上。需要在“磁盘明文存储的风险”与“无需逐个解锁即可重启 Swarm 的便利”之间权衡。

```console
$ docker swarm update --autolock=false
```

关闭自动锁定后，请在短时间内保留旧的解锁密钥，以防某个仍使用旧密钥配置的管理节点宕机后需要恢复。

## 解锁 Swarm

要解锁处于锁定状态的 Swarm，执行 `docker swarm unlock`：

```console
$ docker swarm unlock

Please enter unlock key:
```

根据提示输入在锁定或轮换密钥时生成并输出的解锁密钥，即可完成解锁。

## 查看正在运行的 Swarm 的当前解锁密钥

设想一种情况：Swarm 正常运行时某个管理节点变得不可用。你排查后让该物理节点恢复上线，但需要提供解锁密钥以读取加密的凭据与 Raft 日志，才能解锁该管理节点。

如果自该节点离开集群后密钥未被轮换，且当前有法定数量的管理节点处于正常状态，你可以直接执行 `docker swarm unlock-key`（无参数）查看当前解锁密钥。

```console
$ docker swarm unlock-key

To unlock a swarm manager after it restarts, run the `docker swarm unlock`
command and provide the following key:

    SWMKEY-1-8jDgbUNlJtUe5P/lcr9IXGVxqZpZUXPzd+qzcGp4ZYA

Please remember to store this key in a password manager, since without it you
will not be able to restart the manager.
```

如果在该节点不可用期间进行了密钥轮换，而你又没有旧密钥的记录，可能需要强制让该管理节点退出 Swarm，并以新管理节点的身份重新加入。

## 轮换解锁密钥

建议定期轮换已锁定 Swarm 的解锁密钥：

```console
$ docker swarm unlock-key --rotate

Successfully rotated manager unlock key.

To unlock a swarm manager after it restarts, run the `docker swarm unlock`
command and provide the following key:

    SWMKEY-1-8jDgbUNlJtUe5P/lcr9IXGVxqZpZUXPzd+qzcGp4ZYA

Please remember to store this key in a password manager, since without it you
will not be able to restart the manager.
```

> [!WARNING]
>
> 轮换解锁密钥时，请在几分钟内保留旧密钥记录。若某个管理节点在获取新密钥前宕机，仍可用旧密钥对其解锁。
