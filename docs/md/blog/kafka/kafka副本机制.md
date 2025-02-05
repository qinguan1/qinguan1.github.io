## Kafka副本机制

作者：qinguan
<br/>博客：[https://bugstack.cn](https://bugstack.cn)

> 故不积跬步，无以至千里；不积小流，无以成江海！🌻

### 一、副本机制

#### 1.1 概述

副本（ Replica ）是分布式系统中常见的概念之一，指的是分布式系统对数据和服务提供的一种冗余方式，是解决分布式系统数据丢失问题最有效的手段。

Kafka 从 0.8 版本开始为分区引入了多副本机制，可以通过增加副本数量来提升数据容灾能力。

所谓的副本机制（Replication），也可以称之为**备份机制**，通常是指分布式系统在多台网络互联的机器上保存有相同的数据拷贝。
副本机制有什么好处呢？
>- **提供数据冗余**。即使系统部分组件失效，系统依然能够继续运转，因而增加了整体可用性以及数据持久性。
>- **提供高伸缩性**。支持横向扩展，能够通过增加机器的方式来提升读性能，进而提高读操作吞吐量。
>- **改善数据局部性**。允许将数据放入与用户地理位置相近的地方，从而降低系统延时。

对于 Apache Kafka 而言，目前只能享受到副本机制带来的第一个好处，也就是**提供数据冗余实现高可用性和高持久性**。
Kafka 所谓副本（Replica），本质就是**一个只能追加写消息的提交日志**。根据 Kafka 副本机制的定义，同一个分区下的所有副本保存有相同的消息序列，这些副本分散保存在不同的 Broker 实例上，从而能够容忍部分 Broker 实例宕机带来的数据不可用。

#### 1.2 副本分类

在 Kafka 中，副本分成两类：领导者副本（Leader Replica）和追随者副本（Follower Replica）。每个分区在创建时都要选举一个副本，称为领导者副本，其余的副本则为追随者副本（副本数量为1的情况下没有追随者副本）。

**只有 Leader 副本负责对外提供读写服务，Follower 副本只负责从 Leader 副本备份数据**。
如果作为 Leader 副本的 Broker 实例宕机，Kafka 集群的controller实例可以感知到对应的 Broker 实例宕机，并立即开启新一轮的 Leader 选举，会从 ISR 列表中选择优先副本（通常是第一个）作为新的 Leader 副本提供服务。

需要注意的是：**对于一个topic而言，它的 Leader 副本和 Follower 副本不会在一个 Broker 实例上**，也就意味着主题的副本数量**不会超过** Kafka 集群中的 Broker 实例数量。

在实际生产环境中，每台 Broker 实例都可能保存有各个主题下不同分区的不同副本，因此，单个 Broker 实例上存有成百上千个副本的现象是非常正常的。

#### 1.3 副本的重要概念

##### 1.3.1 概述
对于副本而言，还有两个概念：本地副本（ Local Replica ）和远程副本（ Remote Replica) , 本地副本是指对应的 Log 分配在当前的 broker 节点上，远程副本是指对应的 Log 分配在其他的 broker 节点上。

##### 1.3.2 LEO，HW(高水位)

LEO: 标识每个分区中最后一条消息的下一个位置，分区的每一个副本都有自己的LEO

HW: ISR中最小的LEO即为HW，俗称高水位，消费者只能拉取HW之前的消息

HW定义了消息的可见性，即标识Partition中的哪些消息是可以被Consumer消费的，只有小于HW值的消息才被认为是已备份或已提交的（committed）


#### 1.4 副本集合

##### 1.4.1 AR(Assigned Replicas)
分区中的所有副本统称为 AR (Assigned Replicas)
##### 1.4.1 ISR(in-sync-replica set)
与 Leader 副本保持一定程度同步的副本（包括 Leader 副本在内）组成 ISR (In Sync Replicas)。
至于同步的程度是由 replica.lag.time.max.ms 配置决定的，超过这个时间都没有跟leader同步过的一次的副本会被**移出** ISR 列表，移入 OSR 列表)

##### 1.4.2 OSR(Out-of-Sync Replicas)
数据同步严重滞后的副本组成OSR（网络原因造成的等等），如果副本数为1或者副本同步进度差不多，那么这个可以为空。
如果该 Follower 副本的 LEO 后续追赶上 Leader 副本的HW，则其有资格进入 ISR 列表，会被定时任务移入 ISR 列表。

集合之间的关系：**AR = ISR + OSR**

### 二、Leader Epoch 机制

### 2.1 概述

在 0.11.0.0 版本之前， Kafka使用的是基于HW的同步机制，但这样有可能出现数据丢失或 Leader 副本和 Follower 副本数据不一致的问题。

Kakfa引入Leader Epoch后，Follower就不再参考HW，而是根据Leader Epoch信息来截断Leader中不存在的消息。这种机制可以弥补基于HW的副本同步机制的不足，Leader Epoch由两部分组成：

- Epoch：一个单调增加的版本号。每当Leader副本发生变更时，都会增加该版本号。Epoch值较小的Leader被认为是过期Leader，不能再行使Leader的权力；
- 起始位移（Start Offset）：Leader副本在该Epoch值上写入首条消息的Offset。















































