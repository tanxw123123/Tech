## 1.消息队列两种模式

- 点对点：Queue，不可重复消费
- 发布订阅：Topic，可以重复消费

1.点对点

```
    消息生产者生产消息发送到queue中，然后消息消费者从queue中取出并且消费消息。消息被消费以后，queue中不再有存储，所以消息消费者不可能消费到已经被消费的消息。Queue支持存在多个消费者，但是对一个消息而言，只会有一个消费者可以消费。
```

2.发布订阅

```
    消息生产者（发布）将消息发布到topic中，同时有多个消息消费者（订阅）消费该消息。和点对点方式不同，发布到topic的消息会被所有订阅者消费。
```

---



## 2.关键术语

（1）生产者和消费者（producer和consumer）：消息的发送者叫 Producer，消息的使用者和接受者是 Consumer，生产者将数据保存到 Kafka 集群中，消费者从中获取消息进行业务的处理。

（2）broker：Kafka 集群中有很多台 Server，其中每一台 Server 都可以存储消息，将每一台 Server 称为一个 kafka 实例，也叫做 broker。

（3）主题（topic）：一个 topic 里保存的是同一类消息，相当于对消息的分类，每个 producer 将消息发送到 kafka 中，都需要指明要存的 topic 是哪个，也就是指明这个消息属于哪一类。

（4）分区（partition）：每个 topic 都可以分成多个 partition，每个 partition 在存储层面是 append log 文件。任何发布到此 partition 的消息都会被直接追加到 log 文件的尾部。为什么要进行分区呢？最根本的原因就是：kafka基于文件进行存储，当文件内容大到一定程度时，很容易达到单个磁盘的上限，因此，采用分区的办法，一个分区对应一个文件，这样就可以将数据分别存储到不同的server上去，另外这样做也可以负载均衡，容纳更多的消费者。

（5）偏移量（Offset）：一个分区对应一个磁盘上的文件，而消息在文件中的位置就称为 offset（偏移量），offset 为一个 long 型数字，它可以唯一标记一条消息。由于kafka 并没有提供其他额外的索引机制来存储 offset，文件只能顺序的读写，所以在kafka中几乎不允许对消息进行“随机读写”。

**综上，我们总结一下 Kafka 的几个要点:**

- kafka 是一个基于发布-订阅的分布式消息系统（消息队列）
- Kafka 面向大数据，消息保存在主题中，而每个 topic 有分为多个分区
- kafka 的消息数据保存在磁盘，每个 partition 对应磁盘上的一个文件，消息写入就是简单的文件追加，文件可以在集群内复制备份以防丢失
- 即使消息被消费，kafka 也不会立即删除该消息，可以通过配置使得过一段时间后自动删除以释放磁盘空间
- kafka依赖分布式协调服务Zookeeper，适合离线/在线信息的消费，与 storm 和 spark 等实时流式数据分析常常结合使用



**副本（replicated）**

```
kafka 还可以配置 partitions 需要备份的个数(replicas),每个 partition 将会被备份到多台机器上,以提高可用性，备份的数量可以通过配置文件指定。

  这种冗余备份的方式在分布式系统中是很常见的，那么既然有副本，就涉及到对同一个文件的多个备份如何进行管理和调度。kafka 采取的方案是：每个 partition 选举一个 server 作为“leader”，由 leader 负责所有对该分区的读写，其他 server 作为 follower 只需要简单的与 leader 同步，保持跟进即可。如果原来的 leader 失效，会重新选举由其他的 follower 来成为新的 leader。

  至于如何选取 leader，实际上如果我们了解 ZooKeeper，就会发现其实这正是 Zookeeper 所擅长的，Kafka 使用 ZK 在 Broker 中选出一个 Controller，用于 Partition 分配和 Leader 选举。

  另外，这里我们可以看到，实际上作为 leader 的 server 承担了该分区所有的读写请求，因此其压力是比较大的，从整体考虑，有多少个 partition 就意味着会有多少个leader，kafka 会将 leader 分散到不同的 broker 上，确保整体的负载均衡。
```

---



## 3.安装zookeeper

https://www.w3cschool.cn/apache_kafka/apache_kafka_installation_steps.html

```shell
$ wget https://archive.apache.org/dist/zookeeper/zookeeper-3.4.6/zookeeper-3.4.6.tar.gz
$ tar -xf zookeeper-3.4.6.tar.gz
$ cd zookeeper-3.4.6
$ mkdir data

$ vi conf/zoo.cfg
tickTime=2000
dataDir=/path/to/zookeeper/data
clientPort=2181
initLimit=5
syncLimit=2
```

启动zookeeper：

```shell
$ bin/zkServer.sh start  # 启动
$ bin/zkServer.sh stop   # 停止
```

## 4.安装Kafka



```shell
$ wget https://archive.apache.org/dist/kafka/0.9.0.0/kafka_2.11-0.9.0.0.tgz
$ tar -xf kafka_2.11-0.9.0.0.tgz
$ cd kafka_2.11-0.9.0.0
$ vim config/server.properties
listeners=PLAINTEXT://192.168.254.51:9092
```

启动Kafka：

```shell
$ bin/kafka-server-start.sh -daemon config/server.properties      # 启动kafka命令加上–daemon，那么kafka会以守护进程的方式启动
$ bin/kafka-server-stop.sh config/server.properties   # 停止

```

## 5.配置

1.创建队列

- --zookeeper指定zookeeper的地址和端口
- --partitions指定partition的数量
- --replication-factor指定数据副本的数量

```shell
$ kafka-topics.sh --create --zookeeper 192.168.254.51:2181 --topic kb09demo --partitions 1 --replication-factor 1

//创建Topic  5个分区
$ kafka-topics.sh --create --zookeeper 192.168.254.51:2181 --topic kb09demo2 --partitions 5 --replication-factor 1
```

2.查看队列列表

```shell
$ kafka-topics.sh --zookeeper 192.168.254.51:2181 --list
```

3.查看指定topic明细

```shell
$ kafka-topics.sh --zookeeper 192.168.254.51:2181 --topic kb09demo --describe
$ kafka-topics.sh --zookeeper 192.168.254.51:2181 --topic kb09demo2 --describe
```

4.修改列对

```shell
# 下面命令，增加partion数量，从5个partition增加到6个
$ kafka-topics.sh --zookeeper 192.168.254.51:2181 --alter --topic kb09demo2 --partitions 6

# 但是减少partition是不允许的。如果执行配置的partition变少，会抛出一个错误，显示partition数量只能增加
```

5.删除队列

```shell
# 删除之前，需要先将server.properties文件中的配置delete.topic.enable=true更改一下，否则执行删除命令不会生效。
$ kafka-topics.sh --zookeeper 192.168.254.51:2181 --delete --topic kb09demo
```

6.Producer（消息的生成者）

创建生产消息
运行Producer并将Terminal输入的消息发送到服务端

```shell
$ kafka-console-producer.sh --topic kb09demo --broker-list 192.168.254.51:9092
```

7.Consumer（消息的消费者）

启动Consumer读取消息并输出到标准输出

```shell
$ kafka-console-consumer.sh --zookeeper 192.168.254.51:2181 --topic kb09demo2 --bootstrap-server 192.168.254.51:9092 --from-beginning

# 如果加上from-beginning指定从第一条数据开始消费
# --bootstrap-server 192.168.254.51:9092 此参数为可选项
```

---

## 6.kafka集群迁移

**1.先迁生产者，再迁消费者**

​        指先将生产消息的业务迁移到新的Kafka，原Kafka不会有新的消息生产。待原有Kafka实例的消息全部消费完成后，再将消费消息业务迁移到新的Kafka，开始消费新Kafka实例的消息。

1. 将生产客户端的Kafka连接地址修改为新Kafka实例的连接地址。
2. 重启生产业务，使得生产者将新的消息发送到新Kafka实例中。
3. 观察各消费组在原Kafka的消费进度，直到原Kafka中数据都已经被消费完毕。
4. 将消费客户端的Kafka连接地址修改为新Kafka实例的连接地址。
5. 重启消费业务，使得消费者从新Kafka实例中消费消息。
6. 观察消费者是否能正常从新Kafka实例中获取数据。
7. 迁移结束。

​         本方案为业界通用的迁移方案，操作步骤简单，迁移过程由业务侧自主控制，整个过程中消息不会存在乱序问题，**适用于对消息顺序有要求的场景**。但是该方案中需要等待消费者业务直至消费完毕，存在一个时间差的问题，部分数据可能存在较大的端到端时延。

```shell
#如何查看kafka消息消费进度以及是否有未消费的消息
# 查看消费组列表
$ kafka-consumer-groups.sh --bootstrap-server <kafka-ip>:9092 --list
# 查看 kafka 中某一个消费者组的消费情况
$ kafka-consumer-groups.sh --bootstrap-server <kafka-ip>:9092 --group test-17 --describe
GROUP           TOPIC                    PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID                                     HOST            CLIENT-ID
logstash        wsdy-prod-nginx-info     0          20117           20117           0               logstash-0-6dd159ce-f977-4efc-b99c-20d2f5b0f848 /192.168.254.60 logstash-0
logstash        wsdy-prod-ubuntu01-info  0          12111           12111           0               logstash-0-6dd159ce-f977-4efc-b99c-20d2f5b0f848 /192.168.254.60 logstash-0
logstash        wsdy-prod-ubuntu01-error 0          3               3               0               logstash-0-6dd159ce-f977-4efc-b99c-20d2f5b0f848 /192.168.254.60 logstash-0

---
在上面这张图中，我们可以看到该消费者组消费的 topic、partition、当前消费到的 offset 、最新 offset 、LAG(消费进度) 等等。如果消费者的 offset 很长时间没有提交导致 LAG 越来越大，则证明消费 Kafka 的服务异常。

消费者组消费 topic 的元数据信息，在旧版本里面是存储在 zookeeper 中，但由于 zookeeper 并不适合大批量的频繁写入操作，新版 kafka 已将消费者组的元数据信息保存在 kafka 内部的 topic 中，即 __consumer_offsets ，并提供了 kafka-console-consumer.sh 脚本供用户查看消费者组的元数据信息。
```

