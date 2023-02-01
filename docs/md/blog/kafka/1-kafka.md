## Kafka

作者：qinguan
<br/>博客：[https://bugstack.cn](https://bugstack.cn)

> 故不积跬步，无以至千里；不积小流，无以成江海！🌻

### 一、概述
kafka的重要概念：
1. **Producer**：Producer 即生产者，消息的产生者，是消息的入口。
2. **Broker**：Broker是 Kafka 实例，每个服务器上有一个或多个 Kafka 的实例，我们姑且认为每个 broker 对应一台服务器。每个kafka集群内的 broker 都有一个不重复的编号，如图中的 broker-0、broker-1 等……
3. **Topic**：消息的主题，可以理解为消息的分类，kafka的数据就保存在 Topic。在每个broker上都可以创建多个 Topic。
4. **Partition**：Topic 的分区，每个 Topic 可以有多个分区，分区的作用是做负载，提高kafka的吞吐量。同一个 Topic 在不同的分区的数据是不重复的，Partition 的表现形式就是一个一个的文件夹！
5. **Replication**:每一个分区都有多个副本，副本的作用是做备胎。当主分区（Leader）故障的时候会选择一个备胎（Follower）上位，成为 Leader。在 Kafka 中默认副本的最大数量是10个，且副本的数量不能大于 Broker 的数量，Follower 和 Leader 绝对是在不同的机器，同一机器对同一个分区也只可能存放一个副本（包括自己）。
6. **Message**：每一条发送的消息主体。
7. **Consumer**：消费者，即消息的消费方，是消息的出口。
8. **Consumer Group**：我们可以将多个消费组组成一个消费者组，在 Kafka 的设计中同一个分区的数据只能被消费者组中的某一个消费者消费。同一个消费者组的消费者可以消费同一个 Topic 的不同分区的数据，这也是为了提高 kafka 的吞吐量！


### 为什么kafka要逐步放弃zk？
1. kafka自身侧面：逐步实现自我管理，不必再依赖zk进行元数据的存储，这样也就较少了交互复杂度
2. 运维层面：不论是部署还是日常维护，都可以大大减轻运维的压力，不需要单独为kafka维护一套zk集群
3. 协议层面：Raft比ZK的ZAB协议更加易懂，也更加高效










