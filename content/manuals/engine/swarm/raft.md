---
description: Swarm 模式下的 Raft 共识算法
keywords: docker, container, cluster, swarm, raft
title: Swarm 模式中的 Raft 共识
---

当 Docker Engine 以 Swarm 模式运行时，管理节点会实现并使用
[Raft 共识算法](http://thesecretlivesofdata.com/raft/) 来维护全局集群状态。

Swarm 模式采用共识算法的原因，是为了确保负责集群管理与任务调度的所有管理节点
始终持有一致的状态副本。

当整个集群保持一致状态时，即便发生故障，任意管理节点都能接管工作并将服务恢复到稳定状态。
例如，如果负责调度的 Leader 管理节点意外宕机，其他任意管理节点都可以接手调度，
并对任务进行再平衡，以满足期望状态。

在分布式系统中，通过共识算法复制日志的架构需要特别关注。此类系统通过要求多数节点就提议达成一致，
来确保在出现故障时，集群状态仍然保持一致。

Raft 最多可容忍 `(N-1)/2` 个节点故障，并要求多数（法定人数）即 `(N/2)+1` 个成员
对提交给集群的提议达成一致。这意味着：在一个包含 5 个管理节点并运行 Raft 的集群中，
如果有 3 个节点不可用，系统将无法继续处理新的调度请求。现有任务虽会继续运行，
但若管理节点集合不健康，调度器将无法对任务进行再平衡以应对故障。

Swarm 模式下对共识算法的实现，具备分布式系统应有的特性：

- 在容错系统中对提议达成一致。（参见 [FLP 不可能性定理](https://www.the-paper-trail.org/post/2008-08-13-a-brief-tour-of-flp-impossibility/)
  与 [Raft 共识算法论文](https://www.usenix.org/system/files/conference/atc14/atc14-paper-ongaro.pdf)）
- 通过领导者选举实现互斥
- 集群成员关系管理
- 全局一致的对象序列与 CAS（Compare-And-Swap，比较并交换）原语
