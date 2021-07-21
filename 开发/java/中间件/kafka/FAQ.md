## 副本同步延迟

kafka消费者也具备容灾的能力，当消费者使用pull从服务端拉去消息，并且保存了消费的具体offset，当消费者宕机恢复后，会根据之前保存的消费者位置重新进行消费，这样就不会造成消息丢失。
分区中的所有副本统称AR(Assigned Replicas),所有与leader副本保持一定程度同步的副本（包括leader副本）组成ISR(In-Sync Replicas),ISR是AR集合的一个子集。消息会先发送到leader副本，然后follower副本才能从leader副本中拉取消息进行同步，同步期间内follower副本相对于leader副本有一定程度的滞后，这里说的一定程度是指可以忍受的范围内，这个参数可以通过配置。与leader副本同步滞后过多的副本（不包含leader副本）组成OSR(Out-of-Snyc-Replicas),也就是AR=ISR+OSR，正常情况下，所有的follower副本都应该与该leader副本保持一定程度的同步，也就是AR=ISR。
leader副本负责维护和跟踪ISR集合中所有follower副本的滞后状态，当follower落后太多或者失效时，leader副本会把它从ISR中删除，如果OSR中的follower副本追上了leader副本，那么leader副本会把它从OSR移到ISR中。默认情况下，在leader副本发送故障，只有在ISR集合中副本才有机会被选中为leader，在OSR集合中的副本没有任何机会。
ISR和HW和LEO也有密切关系，HW是高水位，标识了一个特定的消息的偏移量，消费者只能拉取到这个offset之前的消息

![img](https://pic4.zhimg.com/80/v2-0899e9c3d61e7b601c8a61c00e727ca7_1440w.jpg)

它代表一个日志文件的一个分区，这个日志文件分区中有9条消息，第一条消息的offset为0，最后一条消息的offset为8，offset为9的消息用虚线表示，代表下一条待输入的消息的offset，日志文件的HW为6，表示消费者只能拉去到offset为0至5之间的消息，而offset为6的消息对消费者是不可见的。
LEO是log end offset，它标识当前日志文件中下一条待写入消息的offset，上图中的9为LEO位置。分区ISR集合中的每个副本都会维护自身的LEO，而ISR集合中最小的LEO为分区的HW，对消费者而言，只能消费HW之前的消息。
为了更好理解ISR集合，以及HW和LEO之间的关系，通过下图说明



![img](https://pic1.zhimg.com/80/v2-07c1753a42bf1a07be58f98bc1f9a56c_1440w.jpg)



假设某个分区的ISR集合中有3个副本，也就是一个leader副本和两个follower副本，此时分区的LEO和HW都是3，消息3和消息4从生产者发出之后先被存入leader副本，在消息写入leader之后，follower副本会发送拉去消息请求。

![img](https://pic3.zhimg.com/80/v2-7c4befbd58cd19b423d5a2f394d7443a_1440w.jpg)


在同步过程中，不同的follower副本同步的效率也不同，如下图

![img](https://pic1.zhimg.com/80/v2-e65da74d07741f435d6ea622b427a0bc_1440w.jpg)



在某一时刻follower1完全跟上了leader，但是follower2只同步了消息3，因此leader副本的LEO值为5，
follower1的LEO为5，follower2的LEO为4，那么当前的HW为4，所以消费者只能消费offset为0-3的消息。
所有的副本都成功的写入了消息3和消息4，这个分区的Hw和LEO都是5，因此消费者可以消费到offset为0-4之间的消息了。

## 消息丢失

## 重复消费

## 消息有序

前面提到，在同一个 partition 中，消息的顺序是能够得到保证的。因此对于一个小型的、对可靠性要求不高、但是对顺序性要求很高的系统而言，或许可以使用单 partition 的方案。

　　但是这个方案其实是非常危险的：

- 首先，单一 partition 就意味着 consumer 也只能有一个，否则会出现消息重复消费的问题。在一个生产项目中进行单点部署，这几乎是不可接受的
- 虽然在 Kafka 内部，单一 partition 内的消息顺序能够得到保证，但如果生产者未能得到保证的话，那么 kafka 内的消息顺序依然不是真实的。因此对于有强顺序要求的消息队列系统中，不建议使用时间顺序，而是采用逻辑顺序/逻辑时钟来区分消息的先后。

　　因此在实际生产环境中，我们应当适当地分配 partition 的数量。如果对顺序性有要求，那么不应该依赖 kafka 的顺序机制，而是使用额外的机制来保证。

## 保证消息发送成功

## 单条消息大小

 默认1M

### 解决方案

- 修改kafka的broker配置：message.max.bytes（默认:1000000B），这个参数表示单条消息的最大长度。在使用kafka的时候，应该预估单条消息的最大长度，不然导致发送失败。
- 修改kafka的broker配置：replica.fetch.max.bytes (默认: 1MB)，broker可复制的消息的最大字节数。这个值应该比message.max.bytes大，否则broker会接收此消息，但无法将此消息复制出去，从而造成数据丢失。
- 修改消费者程序端配置：fetch.message.max.bytes (默认 1MB) – 消费者能读取的最大消息。这个值应该大于或等于message.max.bytes。如果不调节这个参数，就会导致消费者无法消费到消息，并且不会爆出异常或者警告，导致消息在broker中累积，此处要注意。

### 带来的问题

- 从性能上考虑：通过性能测试，kafka在消息为10K时吞吐量达到最大，更大的消息会降低吞吐量，在设计集群的容量时，尤其要考虑这点。
- 可用的内存和分区数：Brokers会为每个分区分配replica.fetch.max.bytes参数指定的内存空间，假设replica.fetch.max.bytes=1M，且有1000个分区，则需要差不多1G的内存，确保 分区数*最大的消息不会超过服务器的内存，否则会报OOM错误。同样地，消费端的fetch.message.max.bytes指定了最大消息需要的内存空间，同样，分区数*最大需要内存空间 不能超过服务器的内存。所以，如果你有大的消息要传送，则在内存一定的情况下，只能使用较少的分区数或者使用更大内存的服务器。
- 垃圾回收：更大的消息会让GC的时间更长（因为broker需要分配更大的块），随时关注GC的日志和服务器的日志信息。如果长时间的GC导致kafka丢失了zookeeper的会话，则需要配置zookeeper.session.timeout.ms参数为更大的超时时间。

## 消息淤积

```text	
参考：
		https://zhuanlan.zhihu.com/p/88429983
		https://blog.csdn.net/hanjibing1990/article/details/51673540
```

