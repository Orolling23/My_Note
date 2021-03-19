# RocketMQ

## 架构组成
###  NameServer

负责管理元数据，充当路由消息的提供者，包括对**Topic和路由信息**的管理。其角色类似Dubbo中的Zookeeper，但更轻量，NameServer平时主要开销是**维持心跳和提供Topic-Broker的关系数据**。集群相互之间无通信，伪集群

### Broker
消息中转角色，负责**存储消息，转发消息**。  
* Broker是具体提供业务的服务器，**单个Broker节点与所有的NameServer节点保持长连接及心跳，并会定时将Topic信息注册到NameServer**，顺带一提底层的通信和连接都是**基于Netty实现**的。  
* Broker负责**消息存储**，以Topic为纬度支持轻量级的队列，单机可以支撑**上万队列**规模，支持**消息推拉模型**。  
* 具有上亿级消息堆积能力，同时可严格保证消息的有序性。  

### Producer
消息生产者，负责产生消息，一般由业务系统负责。  
* Producer由用户进行分布式部署，消息由Producer通过多种负载均衡模式发送到Broker集群，发送低延时，支持快速失败。  
* RocketMQ 提供了三种方式发送消息：同步、异步和单向
	* 同步发送：同步发送指消息发送方发出数据后会在收到接收方发回响应之后才发下一个数据包。一般用于重要通知消息，例如重要通知邮件、营销短信。
	* 异步发送指发送方发出数据后，不等接收方发回响应，接着发送下个数据包，一般用于可能链路耗时较长而对响应时间敏感的业务场景，例如用户视频上传后通知启动转码服务。
	* 单向发送是指只负责发送消息而不等待服务器回应且没有回调函数触发，适用于某些耗时非常短但对可靠性要求并不高的场景，例如日志收集。

### Consumer
消息消费者，负责消费消息，一般是后台系统负责异步消费。  
* Consumer也由用户部署，支持**PUSH和PULL两种消费模式**，支持集群消费和广播消息，提供实时的消息订阅机制。
* **Pull**：拉取型消费者（Pull Consumer）**主动从消息服务器拉取信息**，只要批量拉取到消息，用户应用就会启动消费过程，所以 Pull 称为主动消费型。  
* **Push**：推送型消费者（Push Consumer）封装了消息的拉取、消费进度和其他的内部维护工作，将消息到达时执行的回调接口留给用户应用程序来实现。所以 Push 称为被动消费类型，但**从实现上看还是从消息服务器中拉取消息**，不同于 Pull 的是 **Push 首先要注册消费监听器，当监听器处触发后才开始消费消息。**  

## 领域模型
### Message
要传输的信息。一条Message必须有一个Topic，也可以拥有一个可选的Tag和额外的键值对，它们可以用于设置一个业务Key并在Broker上查找此消息以便在开发期间查找问题。  

### Topic
Topic可以看作消息的归类，它是消息的第一级类型。比如一个电商系统可以分为：交易消息、物流消息等，一条消息必须有一个 Topic 。一个Topic下面有多个Message Queue，默认是4个。

### Tag
Tag可以看作子主题，消息的第二级类型，用于为用户提供额外的灵活性。*使用标签，同一业务模块不同目的的消息就可以用相同 Topic 而不同的 Tag 来标识。*  

标签有助于保持您的代码干净和连贯，并且还可以为 RocketMQ 提供的查询系统提供帮助。

### Group
分组，一个组可以订阅多个Topic。  

一般有生产者组（Producer Group）和消费者组（Consumer Group）。  

* 生产者组（Producer Group）：同一类Producer的集合，这类Producer发送同一类消息且发送逻辑一致。如果发送的是事务消息且原始生产者在发送之后崩溃，则Broker服务器会联系同一生产者组的其他生产者实例以提交或回溯消费。
* 消费者组（Consumer Group）：同一类Consumer的集合，这类Consumer通常消费同一类消息且消费逻辑一致。消费者组使得在消息消费方面，实现负载均衡和容错的目标变得非常容易。要注意的是，消费者组的消费者实例必须订阅完全相同的Topic。RocketMQ 支持两种消息模式：集群消费（Clustering）和广播消费（Broadcasting）。

### Queue
每个Queue内部有序，在RocketMQ中分为读和写两种队列，一般来说读写队列数量一致。

### Message Queue
消息队列，Topic被划分为一个或多个子主题，即消息队列。  
* 一个 Topic 下可以设置多个消息队列，发送消息时执行该消息的 Topic ，RocketMQ 默认会轮询该 Topic 下的所有队列将消息发出去。  
* 消息的物理管理单位。一个Topic下可以有多个Queue，Queue的引入使得消息的存储可以分布式集群化，具有了水平扩展能力。  

### Offset
在RocketMQ中，所有的消息队列都是持久化，长度无限的数据结构。  

队列中每个存储单元定长，访问其存储单元使用Offset，Offset为java long类型，64位，理论上100年不会溢出，所以认为长度无限。  

也可以认为Message Queue是一个长度无限的数组，Offset就是下标。  

## 消息发送模式
同步消息和异步消息需要Broker返回发送结果；而单向消息不关心结果，不需要Broker回复  

1. 发送同步消息：producer.send方法返回一个SendResult类型，其中包含是否成功送达
2. 发送异步消息：向producer.send中注册一个回调，覆写OnSuccess和onException。
3. 发送单向消息：producer.sendOneWay方法。

## 消息消费模式
### Clustering 集群消费
**默认**，相同Consumer Group的每个Consumer实例平均分摊消息。  

一个消费者集群共同消费一个主题的多个队列，**一个队列只会被每个Consumer Group中的一个消费者消费**，如果某个消费者挂掉，分组内的其他消费者会接替挂掉的消费者继续消费。  

### Broadcasting 广播消费
相同Consumer Group的每个Consumer实例都接收全量的消息。

## 顺序消息

RocketMQ的顺序消息可以实现**全局有序**和**分区有序**。是利用Queue的FIFO来实现的

### 顺序发送
**RocketMQ提供了MessageQueueSelector队列选择机制，用来实现Topic下的队列选择**  

* 全局有序：一个Topic下所有的消息是严格有序的。需要保证一个Topic下只向一个Queue中发送消息
* 分区有序：业务上将消息进行分区，比如同一个订单ID的消息要保证有序，利用hash将需要保证顺序的消息发送到同一个Queue，保证局部有序

### 顺序消费
> 来自官方文档https://github.com/apache/rocketmq/blob/master/docs/cn/RocketMQ_Example.md#22-%E9%A1%BA%E5%BA%8F%E6%B6%88%E8%B4%B9%E6%B6%88%E6%81%AF  
> 
> **在默认的情况下消息发送会采取Round Robin轮询方式把消息发送到不同的queue(分区队列)**；而消费消息的时候从多个queue上拉取消息，这种情况发送和消费是不能保证顺序。**但是如果控制发送的顺序消息只依次发送到同一个queue中，消费的时候只从这个queue上依次拉取，则就保证了顺序**。当发送和消费参与的queue只有一个，则是全局有序；如果多个queue参与，则为分区有序，即相对每个queue，消息都是有序的。  

消费者有两种：MessageListenerConcurrently并发消费， MessageListenerOrderly顺序消费  

* 并发消费允许一个Message Queue由Consumer的多个线程并发消费，不保证消息消费的顺序。是默认，也是最常用的消费方式，消费的性能高。  

* 顺序消费时，一个Message Queue只能由Consumer的一个线程消费，保证消费顺序。  

### Orderly 顺序消费
又分为普通顺序消息和严格顺序消息。  
* 普通顺序消息：消费者通过**同一个消费队列**收到的消息是有顺序的，不同消息队列收到的消息则可能是无顺序的。
* 严格顺序消息：消费者收到的所有消息均是有顺序的，所以如果正在处理全局顺序是强制性的场景，需要确保使用的**主题只有一个消息队列**。


## 延时消息
定时消息是指消息发到Broker后，不能立刻被Consumer消费，要到特定的时间点或者等待特定的时间后才能被消费。  

RocketMQ支持定时消息，但是**不支持任意时间精度**，支持特定的level，例如定时5s，10s，1m等。  
```
  message.setDelayTimeLevel(3);   //level 3表示10秒。详细的看delayTimeLevel
  // private String messageDelayLevel = "1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h";

```
### 延时消息的使用场景
比如电商场景中，需要客户1小时内付款。可以在用户下单时发送一个延时1小时的消息，待一小时到达时，消费消息，检查用户是否付款。  

## 过滤消息
利用TAG来过滤消息，即Topic + TAG 组合确定consumer要消费什么样的消息  

### 基本语法
* 数值比较，比如：>，>=，<，<=，BETWEEN，=；
* 字符比较，比如：=，<>，IN；
* IS NULL 或者 IS NOT NULL；
* 逻辑符号 AND，OR，NOT；

常量支持类型：
* 数值，比如：123，3.1415；
* 字符，比如：'abc'，必须用单引号包裹起来；
* NULL，特殊的常量
* 布尔值，TRUE 或 FALSE

## 事务消息
RocketMQ中，事务消息定义了三种状态：提交状态、回滚状态、中间状态

* TransactionStatus.CommitTransaction: 提交事务，它允许消费者消费此消息。
* TransactionStatus.RollbackTransaction: 回滚事务，它代表该消息将被删除，不允许被消费。
* TransactionStatus.Unknown: 中间状态，它代表需要检查消息队列来确定状态。



#################################################### 

## 一次完整的通信流程是怎样的
* Producer与NameServer集群中的其中一个（随机选择）建立长连接，定期从NameServer获取Topic路由信息，并向提供Topic服务的Broker Master建立长连接，且定期向Broker发送心跳。Producer的消息只能发送给Master。  
* Consumer与NameServer集群中的其中一个节点（随机选择）建立长连接，定期从NameServer获取Topic路由信息；Consumer可以同时和Master / Slave建立长连接，且定时向Master、Slave发送心跳，从二者订阅消息都可以



1. 启动NameServer，NameServer起来后监听端口，等待Broker、Producer、Consumer连上来，相当于一个**路由控制中心**。
2. Broker启动，跟**所有**的NameServer保持长连接，定时发送心跳包。**心跳包中包含当前Broker信息(IP+端口等)以及存储所有Topic信息**。注册成功后，**NameServer集群中就有Topic跟Broker的映射关系**。
3. 收发消息前，先创建Topic，创建Topic时需要指定该Topic要存储在哪些Broker上，也可以在发送消息时自动创建Topic。
4. Producer发送消息，启动时先跟NameServer集群中的其中一台建立长连接，并从NameServer中获取当前发送的Topic存在哪些Broker上，**轮询**从队列列表中选择一个队列，然后与队列所在的Broker建立长连接从而向Broker发消息。
5. Consumer跟Producer类似，跟其中一台NameServer建立长连接，获取当前订阅Topic存在哪些Broker上，然后直接跟Broker建立连接通道，开始消费消息。

## RocketMQ的优缺点
优点：  

* 单机吞吐量：十万级
* 可用性：分布式架构，可用性很高
* 消息可靠性：经过参数优化配置，可以达到消息0丢失
* 功能支持：MQ功能较为完善，扩展性好
* 支持10亿级别的消息堆积，不会因为堆积导致性能下降
* 稳定性好，经历过双11的考验

缺点：

* 支持的客户端语言少
* 社区活跃度一般
* 没有在MQ核心中，实现JMS（Java消息服务应用程序接口，是一个Java平台中关于面向消息中间件（MOM）的API），在某些系统迁移的时候可能比较麻烦

## 消息重复
消息领域有一个对消息投递的QoS（服务质量）定义，分为：
* 最多一次
* 至少一次
* 仅一次

RocketMQ实现了至少一次的质量，即如果消息发送失败，则重复发送直至成功。成功的确认消息是返回一个CONSUME_SUCCESS的成功标志。  

但如果发生**网络原因闪断，ACK返回失败**等故障，确认消息没有回到消息队列，导致消息队列不知道消息已经被消费过，消息再次被分发给其他消费者，导致**消息重复**。为了幂等性，就要有消息去重的实现。  

## 消息去重
去重的原则和目的：使业务端逻辑保持**幂等性**。只要保证幂等性，不管来多少条重复消息，最后的处理结果都一样，需要业务端来实现。  

**去重策略**：1. 保证每条消息都有唯一编号，2. 且保证消息处理成功与去重表的日志同时出现。  

建立一个消息表，拿到这个消息做数据库的insert操作。给这个消息做一个唯一主键（primary key）或者唯一约束，那么就算出现重复消费的情况，就会导致主键冲突，那么就不再处理这条消息。  

* 注：幂等性：就是用户对于同一操作发起的一次请求或者多次请求的结果是一致的，不会因为多次点击而产生了副作用，数据库的结果都是唯一的，不可变的。  

## 消息可用性
在集群模式下，有同步刷盘和异步刷盘的策略。如果开启同步刷盘，可以最大程度保证可用性，刷盘超时会返回FLUSH_DISK_TIMEOUT。  

另外主从同步也有同步和异步两种模式，同步同样可以提升可用性，但消息发送的RT会下降10%左右。  

## RocketMQ 刷盘实现
Broker在消息存取时直接操作内存（内存映射文件），这可以提高系统吞吐量，但无法避免机器掉电时数据丢失，所以需要持久化到磁盘。刷盘使用NIO中的 MappedByteBuffer.force() 将映射区的数据写入到磁盘。  

同步刷盘时，在Broker把消息写到CommitLog映射区后，就等待写入完成  

异步刷盘时，唤醒FlushCommitLogService线程，不保证执行时机。  

## 分布式事务
* Half Message 半消息：是指暂不能被Consumer消费的消息。**Producer 已经把消息成功发送到了 Broker 端，但此消息被标记为暂不能投递状态**，处于该种状态下的消息称为半消息。需要 Producer对消息的**二次确认**后，Consumer才能去消费它。
* 消息回查：由于网络闪段，生产者应用重启等原因，导致 Producer 端一直没有对 Half Message(半消息) 进行 二次确认。这时Brock服务器会**定时扫描长期处于半消息的消息**，会**主动询问 Producer端 该消息的最终状态(Commit或者Rollback)**,该消息即为 消息回查。

## 消息过滤
* Broker端：按照Consumer的要求过滤，减少了对Consumer无用消息的传输，但增加了Broker的负担。  
* Consumer端：可由应用完全自定义实现，缺点是需要传输很多无用消息给Consumer

## Broker的Buffer问题
Broker的Buffer通常指的是Broker中一个队列的内存Buffer大小，这类Buffer通常大小有限。  

另外，RocketMQ没有内存Buffer概念，**RocketMQ的队列都是持久化磁盘，数据定期清除**。  

RocketMQ同其他MQ有非常显著的区别，RocketMQ的内存Buffer抽象成一个无限长度的队列，不管有多少数据进来都能装得下，这个无限是有前提的，Broker会定期删除过期的数据。  

例如Broker只保存3天的消息，那么这个Buffer虽然长度无限，但是3天前的数据会被从队尾删除。

## 回溯消费
指Consumer已经消费成功的消息，业务上需求需要重新消费。  

要支持此功能，需要Broker向Consumer成功投递消息后，消息仍需要保留在Broker中。  

RocketMQ支持按照时间回溯消费，时间维度精确到毫秒，可以向前回溯也可以向后回溯。  

## 消息堆积
消息中间件的主要功能是异步解耦，还有个重要功能是**挡住前端的数据洪峰**，保证后端系统的稳定性，这就要求消息中间件具有**一定的消息堆积能力**。  

首先找到消息堆积的原因是Producer太多了，Consumer太少了导致的还是说其他情况？然后看一下消息消费的速度是否正常，如果正常，可以上线更多Consumer，临时解决堆积。  

### 如果Consumer和Queue不对等，上线了多台也在短时间内无法消费完堆积的消息怎么办？
准备一个临时topic，queue分配到多个Broker，上线一台Consumer做消息的搬运工，**把原本Topic中的消息搬运到新的Topic中**，上线N台Consumer同时消费临时Topic中的数据。改完消费者bug后再恢复原来的Consumer和Topic。  
### 堆积的消息怎么办？
消息消费失败后会进入**重试队列（%RETRY%+ConsumerGroup）**，重试失败16次之后进入**死信队列（%DLQ%+ConsumerGroup）**。  


## 为什么消息消费底层都是pull而不用事件监听机制？
事件驱动方式是建立好长连接，由事件（发送数据）的方式来实时推送。  

如果broker主动推送消息的话有可能push速度快，消费速度慢的情况，那么就会造成**消息在consumer端堆积过多**，同时又不能被其他consumer消费的情况。而pull的方式可以根据当前**Consumer自身情况来pull**，不会造成过多的压力而造成瓶颈。所以采取了pull的方式。

## 负载均衡
### Producer的负载均衡
**采用随机递增取模的负载均衡方式**，模数是Queue的数量。  

除此之外，还要通过`latencyFaultTolerance`方式过滤掉not available的Broker，对之前是白的消息，做一定时间的退避。例如，如果上次请求的latency超过550Lms，就退避3000Lms；超过1000L，就退避60000L；如果关闭，采用随机递增取模的方式选择一个队列（MessageQueue）来发送消息，latencyFaultTolerance机制是实现消息发送高可用的核心关键所在。

### Consumer的负载均衡
平均分配的负载均衡策略

### 当消费负载均衡consumer和queue不对等的时候会发生什么？
Consumer和queue会优先平均分配  
* 如果Consumer少于Queue的个数，则存在部分Consumer消费多个Queue的情况
* 如果Consumer等于Queue的个数，则一个Consumer消费一个Queue
* 如果Consumer大于Queue的个数，那么会有部分Consumer空余出来，浪费了