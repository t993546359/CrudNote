# kafka学习笔记



# 1.  第一章 kafka的基本概念

### 1.1 什么是kafka, 它有什么特点

kafka扮演着多种角色，他既是一个基于zookKeeper协调(主流版本)的多分区，多副本的**分布式消息系统**，同时也是一个分布式的**流式处理平台**。

作为分布式的消息系统，他具有系统解耦，流量削峰填谷，异步通信，集群扩展灵活以及可靠性等特点。

**作为消息系统的特性**

系统解耦: 对重要的业务模块进行解耦，业务只需要遵守数据的规范进行消息传输即可

流量削峰填谷: 当业务上游模块流量zhe出现陡增时候，kafka可以起到一个缓冲区的左右。kafka会基于pageCache机制将消息快速写入磁盘做一个持久化存储. 业务的下游则可以按照自己的节奏稳定的从分区消费消息。不用担心流量波动导致服务崩溃。

集群扩展灵活: **<u>TODO:</u>**

异步通信

**作为kafka的特性**

发布订阅模型：支持点对多通信。有的业务上下游场景非常需要kafka的这个特点。

可靠性: kafka集群有着分区副本机制，同时也支持集群的横向扩展，一般来说，只要kafka集群中有节点还在工作，服务就能正常运行。同时kafka也允许服务恢复的节点重新加入集群。

作为流式处理平台，kafka拥有高吞吐，持久化存储，集群水平化扩展等特点，kafka的存储系统特性为流式处理框架提供了一个可靠的数据来源，同时也提供了一个完整的流式处理类库。

### 1.2 kafka集群体系架构

一个kafka分布式集群系统有若干个producer, 若干个consumer以及若干个broker进程以及一个zookeeper集群所组成。 producer负责将消息发送到集群系统。consumer负责从集群系统中拉取消息。broker进程负责将系统中的消息持久化存储到磁盘中。zookeeper集群负责集群元数据的管理以及控制器leader的选举。

### 1.3 kafka的一些基本术语

#### 1.3.1 producer 

发送消息方，一个producer实例负责创建消息并传递到kafka系统中。

#### 1.3.2 CONSUMER

接受消息方，一个consumer实例负责从kafka系统中接受消息。

#### 1.3.3 BROKER

broker可以看作一个kafka服务实例，若干个broker组成了一个kafka集群的服务端。

#### 1.3.4 TOPIC

topic是一个逻辑概念，基于发布订阅模型，生产者可以通过指定topic把消息发送到该topic上.消费者则通过订阅该主题的方式来接受消息。

#### 1.3.5 PARTITION

topic可以被细分成若干个分区。分区或者说是分区副本在物理层面上的概念可以说是一个磁盘上若干个.log日志文件。发送到topic上的消息会经过某种方式计算，最终被存储到某个分区下。分区内的每条消息会被分配一个**offset**作为唯一标识。

#### 1.3.6 PARTITION REPLICA

kafka引入了分区副本机制，通过增加分区副本数量可以提高容灾机制。只有leader副本负责读写，follower副本负责从leader副本以异步的形式拉取数据。当leader副本所在节点服务失效时，AR(ALL REPLICAS) 中编号最小，同时在ISR集合中（默认）的副本会自动成为新的leader副本 继续提供服务。

#### 1.3.7 AR AND ISR

AR : ALL REPLICAS 所有的分区副本集合。

ISR: IN-SYNC REPLICAS 同步滞后范围可忍受的副本集合

#### 1.3.8 HW LEO

LEO: LAST  END OFFSET, 当前分区副本下一条待写入文件的offset.

HW: HIGH WATERMARK: 记录了一个特定的偏移量，消费者只能拉取该偏移量之前的消息。  HW即是ISR集合中最小的offset值。

kafka通过**异步拉取机制**保障了消息处理的性能。 同时又通过HW机制保障了数据的可靠性。

### 1.4 服务端参数

#### 1.4.1 zookeeper.connect
=
连接zookpeeper集群的服务迪纳之以及端口地址

#### 1.4.2 lISTENERS

监听客户端连接的地址列表

#### 1.4.3 BROKER.ID

集群中broker的唯一标识。 默认为-1

#### 1.4.4 LOG.DIR/ LOG.DIRS

kafka在磁盘保留日志消息的路径。

#### 1.4.5 MESSAGE.MAX.BYTES
**关键参数 message.max.bytes(broker), max.message.bytes(topic), max.request.size(producer)**


节点进程一次所能接受消息的最大值，默认为1000012B. 此参数和max.message.bytes(topic) max.request.size(producer)相关。

## 2. 第二章 kafka生产者

### 2.1 正常的生产逻辑

设置参数，创建生产者实例(**KafkaProducer 线程安全**); 构建待发送的消息; 发送消息;关闭生产者实例。

### 2.2 生产者必要的参数配置 

#### 2.2.1 bootstrap.servers

连接kafka集群所需要的broker节点清单

#### 2.2.2 KEY.SERIALIZER/VALUE.SERIALIZIER

 服务端接收到的消息必定是字节数组形式，因此，当我们消息内容传入参数非字节数组时，需要指定序列化器

#### 2.2.3 client.id

指定生产者实例对于的id标识符，如果不设置，kafkaProducer会指定一个默认的标识符。

### 2.3 发送消息的几种方式

send-then-forget(发后即忘), sync(同步), async(异步)

send函数会返回一个Future对象。 同步发送方法可以通过调用get来阻塞等待服务端的响应

```java
producer.send(record).get()
```
不调用get方法即是发后即忘模式。

异步方式则需要多传入一个callback的回调函数
```java
producer.send(record, callback)
```
同时重载onCompletition函数，记录异常信息。

### 2.4 序列化器(serializer): TODO


### 2.5 分区器(Partitioner)
分区器通过定义partiton方法，对传入的key(!= null)进行hash计算，采用MurmurHash2算法进行计算

```java
public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] vlaueBytes, Cluster clusetr)
public void close();
```
#### 2.5.1 MurmurHash2算法探究 TODO:

### 2.6 生产者拦截器 (Producer Interceptor) 0.10.0 版本后引入
主要功能: 过滤不合规的消息，修改消息的内容。 发送回调内容前做一些定制化的需求。
```java
public ProducerRecord(K, V) onSend(ProducerRecord<K, V> record);
public void onAcknowledgement(RecordMetadata,metadata, Exception exception);
public void close()
```
onSend 方法可以修改消息的key ,topic 以及具体消息。
一般来说，只建议修改消息的内容。
onAcknowledgement方法可以在消息被应答或者消息发送失败调用onCompletition方法之前执行。
此方法一般用来执行一些日志记录工作。
close方法用来执行资源的关闭

此方法执行之前需要在kafkaProducer的prop中指定。

kafkaProducer支持多个拦截器同时生效，按照配置项配置的拦截器一次执行。下游拦截器只会处理上游拦截器处理成功的消息。


### 2.7 生产者整体架构
**关键参数： buffer.memory, max.block.ms**

KafkasshInterceptors- --> --> ProduducersSerialixerzers ----> Partiitioners ---->  RecordAccumulator.(消息累加器)

一个生产者客户端会存在两个线程，分别为**主线程和Sender**线程。 主线程负责消息从生产者实例从拦截器处理，序列化器，分区器直到缓存到消息累加器的过程

**sender**线程负责从消息累加器中批量获取消息 并存储到kafka中。 消息累加器缓存大小可以通过生产者客户端参数**buffer.memory** (单位是B)配置, 默认为32MB。 当消息累加器缓存开始挤压时，kafka的Sender方法会开始阻塞， 超过客户端参数**max.block.ms**配置的值之后 会抛出TimeoutException异常。 默认60秒。


#### 2.7.1 RecordAccumulator（消息累加器内部解析）
**关键参数： buffer.memory, max.block.ms**
消息累加器内部维护了一个双端队列Deque\<ProducerBatch>，**每个分区都有**。
Sender线程会读取双端队列中的头部的第一个元素(ProducerBatch) 消息批次进行发送。一个消息批次ProducerBatch可能包含多个ProducerRecord. 

当生产者发送消息流量过大时，请调大**buffer.memory**或者**max.block.ms**值.

#### 2.7.2 BufferPool (缓冲池 消息累加器内部概念)  TODO: 学习概念 NIO
**关键参数 Batch.size**
Send线程向kafka broker发送消息时，需要创建一个**ByteBuffer**来保存发送的消息的。
频繁的创建和释放资源是很消耗内存的，因此生产者维护了一个**BufferPool**来**实现缓存的高效复用**

BufferPool的大小通过参数**Batch.size** （以B为单位，默认16KB）来设置.只有相同大小的ByteBuffer才能被缓存到BufferPool中。

**如何复用**
当新的消息流入时，会先判断已有的ProducerBatch是否还可以写入，如果可以，则写入。 如果不可以，则创建一个新的ProducerBatch，当新的消息评估大小超过**BatchSize**所设大小，则创建一个新的ProducerBatch,大小根据消息实际大小设置，如果未超过，则根据**BatchSize**所设大小设置。

#### 2.7.3 发送前的事
**重要参数 max.in.flight.requests.per.connection** 
**应用逻辑层到物理层的转换**
消息累加器RecordAccumlator中保存的双端队列是和分区一一对应的，也就是<Partition, Deque\<ProducerBatch>>的形式。 分区是逻辑概念，Sender线程会将其转换成物理上的概念
Send线程会将数据保存在kafka里面的InFlights里面
\<Node, Deque\<ProducerBatch>> 进一步封装为\<Node, Deque\<Request>\>的形式。
客户端和Node之间，最多缓存若干个未收到响应的请求。通过参数**max.in.flight.requests.per.connection**设置
我们可以通过比较Deque\<Request> 和 size这个参数大小 判断节点负荷。

####  2.7.4 leastLoadedNode(最小负载节点) 

**重要参数metadata.max.age.ms 默认5分钟** 

根据InFlightRequests中requests size中最小的来决定。

客户端会自动更新元数据的信息， 这个动作由sender线程发起，最后会同样存储到inflightRequests里面. 同时主线程也需要读取消息


## 3. kafka消费者

### 3.1 消息中间件传统的两种对接方式 
点对点(P2P) 以及 发布订阅模式(PUB/SUB)
传统的点对点方式是基于队列的，生产者向队列中投放消息，消费者从队列中获取消息
kafka又引入了消费者组的概念(Consumer Group)
同一条消息只能被消费者组中的某个消费者处理。不同消费者组之间的消费流程相互独立。

### 3.2 消费逻辑主要流程
1. 创建消费者客户端实例并配置参数
2. 订阅主题
3. 拉取消息
4. 消费完成提交位移
5. 关闭消费者实例
   
#### 3.2.1 创建消费者客户端实例并配置参数
创建ConsumerRecords 
配置参数 bootstrap.servers, group.id, key/value.deserializer 

#### 3.2.2 订阅主题
```java
public void subscribe(Collection<String> topic, ...)
public void subscribe(Pattern pattern, ...)
```
消费者订阅以最后一次订阅为准，支持通过正则表达式匹配。
消费者还可以通过assign方法指定订阅topic的部分分区。通过assign方法消费无消费者自动再均衡功能
当没有订阅topic时 会抛出IllegalStateException异常

#### 3.2.3 消息消费的具体方式
kafka消费者消费主要是采用poll模式进行消费。即消费者主动向broker发出请求 拉取消息。
poll方法的具体定义
```java
public ConsumerRecords<K, V> poll(final Duration timeout);
```

### 3.3 消费端内部逻辑

#### 3.3.1 消费位移offset提交

旧版本(0.9之前)，消费者位移保存在zookeeper中。
新版本消费位移保存在内部的主题**_consumer_offsets**中
**消费者提交的位移是下一次消费开始的位置，即LEO(Log End Offset)**

先提交位移，再消费消息 会导致消息丢失
先消费消息，再提交位移，会导致重复消费消息

主要的几种提交位移方式
1. 自动提交
通过消费者客户端参数

































