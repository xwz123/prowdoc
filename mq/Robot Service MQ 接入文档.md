# Robot Service MQ 接入文档

### 概要

为解决当前机器人服务事件的分发于消费的强关联性以及在极端情况下某一机器人所有实例crash掉造成关注事件永久丢失。接入Kafka后事件整体处理流程如下图所示：

![process](images/robot-mq.png)

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

**kafka go 开源库**

1. [sarama](https://github.com/Shopify/sarama) : 具有完整协议支持的纯 Go 实现。包括消费者和生产者实现。
2. [confluent-kafka-go](https://github.com/confluentinc/confluent-kafka-go) : 客户端封装了 librdkafka C 库，提供完整的 Kafka 协议支持，具有出色的性能和可靠性。

### 机器人接入MQ的设想

如下图所示：MQ作为整个机器人服务核心域中的重要组件，需做到隔离变化，依赖抽象。应在接入之初就兼容对不同MQ的切换，每种MQ的不同特性由具体MQ实现去支持。而所有的机器人实现在MQ中的角色应是发布者或订阅者且都依赖Broker抽象层，当切换MQ对应用业务应做到无感知。

![](images/broker.png)



