# Robot Service MQ 接入文档

### 概要

为解决当前机器人服务事件的分发于消费的强关联性以及在极端情况下某一机器人所有实例crash掉造成关注事件永久丢失。

**生产端：**

- 由gitee-access 服务接受webhook事件并生产Kafka消息

- 根据事件所属组织创建topic,如 openEuler:webhook。

  > 华为云分布式消息服务Kafka文档：Kafka实例本身对Topic数量没有做限制，但是Topic的分区数之和有上限，当达到上限之后，用户无法继续创建Topic。
  >
  > 所以，Topic数量和实例分区数上限、每个Topic的分区数有关，其中，每个Topic分区数可在创建Topic时设置，如[图1](https://support.huaweicloud.com/intl/zh-cn/productdesc-kafka/Kafka-specification.html#Kafka-specification__fig145861331153112)，实例分区数上限参考[表1](https://support.huaweicloud.com/intl/zh-cn/productdesc-kafka/Kafka-specification.html#Kafka-specification__table78751014154818)。

- 为了保证消息消费的时序性，使用org/repo/[pr|issue]number作为消息的Key，并根据hash计算均匀的分布在topic的不同分区中，并保证同一PR或issue的事件在同一分区中。

- kafka能保证每条消息只被传递一次：

  > 从 0.11.0.0 版本开始，Kafka producer新增了幂等性的传递选项，该选项保证重传不会在 log 中产生重复条目。 为实现这个目的, broker 给每个 producer 都分配了一个 ID ，并且 producer 给每条被发送的消息分配了一个序列号来避免产生重复的消息。 同样也是从 0.11.0.0 版本开始, producer 新增了使用类似事务性的语义将消息发送到多个 topic partition 的功能： 也就是说，要么所有的消息都被成功的写入到了 log，要么一个都没写进去。这种语义的主要应用场景就是 Kafka topic 之间的 exactly-once 的数据传递。

  > 发送方确认机制：ack=0，不管消息是否成功写入分区；ack=1，消息成功写入leader分区后，返回成功；ack=-1，消息成功写入所有分区后，返回成功。

- 注意事项：

  > 1. 同步复制客户端需要配合使用：acks=all
  > 2. 配置发送失败重试：retries=3
  > 3. 发送优化：linger.ms=0
  > 4. 生产端的内存要足够，避免内存不足导致发送阻塞

**消费端：**

- 同一器人服务不同实例归属于同一个消费组，并topic中的每个分区只由不同分组中唯一实例消费。引入kafka对机器人服务实例数有限制，服务实例数超过topic的分区数会造成实例无法消费消息

- 怎么保证消息只被消费一次？

  - offset 自动提交手动提交

    > 自动提交是每间隔多少时间提交一次offset，当间隔期内服务crash 会造成offset提交丢失造成消息重复消费。

    > 鉴于Kafka自动提交offset的不灵活性和不精确性(只能是按指定频率的提交)，Kafka提供了手动提交offset策略。手动提交能对偏移量更加灵活精准地控制，以保证消息不被重复消费以及消息不被丢失。

  - 消息消费的控制粒度

    > 消费者拉取到消息并成功透传到业务代码

    > 消费者正常完成整个业务逻辑

- 注意事项：

  > 1. consumer的owner线程需确保不会异常退出，避免客户端无法发起消费请求，阻塞消费。
  > 2. 确保处理完消息后再做消息commit，避免业务消息处理失败，无法重新拉取处理失败的消息。
  > 3. consumer不能频繁加入和退出group，频繁加入和退出，会导致consumer频繁做rebalance，阻塞消费。
  > 4. consumer数量不能超过topic分区数，否则会有consumer拉取不到消息。
  > 5. consumer需周期poll，维持和server的心跳，避免心跳超时，导致consumer频繁加入和退出，阻塞消费。
  > 6. consumer拉取的消息本地缓存应有大小限制，避免OOM（Out of Memory）。
  > 7. consumer session设置为30秒，session.timeout.ms=30000。
  > 8. Kafka不能保证消费重复的消息，业务侧需保证消息处理的幂等性。
  > 9. 消费线程退出要调用consumer的close方法，避免同一个组的其他消费者阻塞sesstion.timeout.ms的时间。

### 方案

#### 方案一：机器人服务即为MQ消费者

![](C:\otherProject\prowdoc\mq\images\robot-mq.png)

优势：

- 机器人服务直接依赖MQ，于其它（gitee access）服务无配置项以及网络请求依赖；
- 无需其它容灾方案即可防止服务宕机到重新运行过程中丢失webhook事件；
- 各服务实例负责自己所在分区的消息，无需k8s service 做负载均衡；
- 上线新的机器人只需关注机器人本身所关注的topic以及自身功能配置项，无需关联其它服务配置；

劣势：

- 服务实例与kafka消息分区强关联，因kafka点对点消息消费策略(即同一分区中消息同一时间只能被消费组中唯一一个消费者消费)会限制消费者实例数量(max = topic_num * partition_num)，当消费者数量超过分区数，会造成多出的服务实例饿死；
- 消费组中的服务实例不能频繁退出和加入消费组，因kafka的rebalance比较耗时且会阻塞消费者组中所有的消费者。
- 当某个机器人服务消费组整体达到性能瓶颈，需增加topic分区数，或拆分topic 才能提增加服务数量，且kafka 要缩小分区数需先删除topic在创建topic。

#### 方案二：在机器人服务之上加分发层

 ![](C:\otherProject\prowdoc\mq\images\robot-mq2.png)

优势：

-  解耦机器人服务与topic分区的强关联；将方案1的劣势下沉到分发层；
- 可以快速实现服务实例的扩缩容；单一服务实例宕掉不会阻塞消息的消费；

劣势：

- 机器人服务与分发层强关联，当发布新的机器人需告知分发层；
- 增加了问题跟踪的深度；
- 因分发层消费者不是唯一分发给某一个服务实例集，当某个服务所有实例宕掉，其关注的topic的消息将丢失无法再次消费；
- 方案一的劣势将在分发层体现

**kafka go 开源库**

1. [sarama](https://github.com/Shopify/sarama) : 具有完整协议支持的纯 Go 实现。包括消费者和生产者实现。
2. [confluent-kafka-go](https://github.com/confluentinc/confluent-kafka-go) : 客户端封装了 librdkafka C 库，提供完整的 Kafka 协议支持，具有出色的性能和可靠性。

### 机器人接入MQ的设想

如下图所示：MQ作为整个机器人服务核心域中的重要组件，需做到隔离变化，依赖抽象。应在接入之初就兼容对不同MQ的切换，每种MQ的不同特性由具体MQ实现去支持。而所有的机器人实现在MQ中的角色应是发布者或订阅者且都依赖Broker抽象层，当切换MQ对应用业务应做到无感知。

![](images/broker.png)

### 讨论FAQ

**一个topic分区的上限是多少?**

> Kafka以分区为粒度管理消息，分区多导致生产、存储、消费都碎片化，影响性能稳定性。在使用过程中，当Topic的总分区数达到上限后，用户就无法继续创建Topic。不同规格配置的Topic总分区数不同，具体请参考[Kafka产品规格说明](https://support.huaweicloud.com/productdesc-kafka/Kafka-specification.html)。

> [Apache Kafka支持单集群20万分区](https://www.cnblogs.com/huxi2b/p/9984523.html)]

**保证消息的顺序有无必要?**

在多个分区的情况下是有必要的，随机写入多个分区将会放大消息乱序的出现几率从而影响功能的正确性。

> 假定有两个分区管理一个社区的所有事件，有一个PR 发生两个事件，顺序为添加LGTM 标签，删除LGTM标签；此时A分区有10个消息等待被消费，B分区有1个消息等待被消费，添加标签事件落在A分区，删除标签事件落在B分区，则消息在下层被消费就有极大的概率先消费删除标签事件，从而造成功能的正确性，且跟踪问题和修复问题困难。

**消费者组中消费者个数与分区强耦合的情况下怎么支持消费者动态扩缩容？**

依照kafka架构特性，消费关注的topic的总分区数将成为服务实例扩容的阈值，且频繁的扩缩容将影响消息消费的性能；

在方案一的场景下：

> 在超出topic的总分区数的情况下，扩容需先增加topic的分区数，在进行服务实例的扩容；且kafka对分区数的缩小需对topic进行重建。

在方案二的场景下：

> 业务服务实例的扩缩将变得简单，但是如果是分发层出现性能瓶颈也不能优雅的处理；同时，在该场景下如果出现消息堆积极大的可能也是分发层的性能瓶颈，也就是说消息堆积的监控指标对监控业务服务效果不明显；同时也增加了服务的部署复杂度；

考虑动态扩缩容对整体服务的质量保证：

可以在系统设计之初，经过测算，压测，以保证各机器整体服务支持十万开发者；此时在分区总数的阈值内对单个服务实例的扩缩容也能很好的支持，只是需考虑kafka rebalance对性能的影响，但kafka中短时间的消息堆积对于机器人相对来说时效性不强的业务影响是可以容忍的。从经济情况来说只是扩缩pod，是不会影响k8s集群的整体费用的。

**自动提交是怎么做的？能实现期望拿到消息就移动offset，不用关注消息的业务逻辑处理结果吗？**

可以，开启手动提交可使用编码实现移动offset自主可控。

