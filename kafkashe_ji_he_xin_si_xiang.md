# kafka设计核心思想


Kafka是2010年12月份开源的项目，采用Scala语言编写，使用了多种效率优化机制，整体架构比较新颖（push/pull），更适合异构集群。
###设计目标：
(1) 数据在磁盘上的存取代价为O(1)
(2) 高吞吐率，在普通的服务器上每秒也能处理几十万条消息
(3) 分布式架构，能够对消息分区
(4) 支持将数据并行的加载到Hadoop
###架构：
Kafka实际上是一个消息发布订阅系统。producer向某个topic发布消息，而consumer订阅某个topic的消息，进而一旦有新的关于某个topic的消息，broker会传递给订阅它的所有consumer。 在kafka中，消息是按topic组织的，而每个topic又会分为多个partition，这样便于管理数据和进行负载均衡。同时，它也使用了zookeeper进行负载均衡。
Kafka中主要有三种角色，分别为producer，broker和consumer。
####(1) Producer
Producer的任务是向broker发送数据。
Kafka提供了两种producer接口，一种是low_level接口，使用该接口会向特定的broker的某个topic下的某个partition发送数据；
另一种那个是high level接口，该接口支持同步/异步发送数据，基于zookeeper的broker自动识别和负载均衡（基于Partitioner）。
其中，基于zookeeper的broker自动识别值得一说。producer可以通过zookeeper获取可用的broker列表，也可以在zookeeper中注册listener，该listener在以下情况下会被唤醒：
*a．添加一个broker
b．删除一个broker
c．注册新的topic
d．broker注册已存在的topic*
当producer得知以上时间时，可根据需要采取一定的行动。
0.8.0版本后，producer不再通过zookeeper连接broker, 而是通过brokerlist（192.168.0.1:9092,192.168.0.2:9092,192.168.0.3:9092配置和zookeeper时很像呐）直接和broker连接，只要能和一个broker连接上就能够获取到集群中其他broker上的信息
####(2) Broker
Broker采取了多种策略提高数据处理效率，包括sendfile和zero copy等技术。
####(3) Consumer
consumer的作用是将日志信息加载到中央存储系统上。kafka提供了两种consumer接口，
一种是low level的，它维护到某一个broker的连接，并且这个连接是无状态的，即，每次从broker上pull数据时，都要告诉broker数据的偏移量。
另一种是high-level 接口，它隐藏了broker的细节，允许consumer从broker上pull数据而不必关心网络拓扑结构。
更重要的是，对于大部分日志系统而言，consumer已经获取的数据信息都由broker保存，而在kafka中，由consumer自己维护所取数据信息。

###Kafka是如何实现其高吞吐率及高可用行的？
kafka的集群有多个Broker服务器组成，每个类型的消息被定义为topic，同一topic内部的消息按照一定的key和算法被分区(partition)存储在不同的Broker上，消息生产者producer和消费者consumer可以在多个Broker上生产/消费topic

以高吞吐率作为第一设计原则，kafka的结构设计在很多方面都做了激进的取舍。
####① 极简的数据结构和应用模式 
消息队列是以log文件的形式存储，消息生产者只能将消息添加到既有的文件尾部，没有任何ID信息用于消息的定位，完全依靠文件内的位移，因此消息的使用者只能依靠文件位移顺序读取消息，这样也就不需要维护复杂的支持随即读取的索引结构。
  kafka broker完全不维护和协调多用户使用消息的行为模式，consumer自己维护位移用来索引消息,consumer将位移维护在zookeeper中，默认1分钟自动提交一次，也可以手动commitoffset
  topic最小的并发访问单位就是partition分区，同一用户组内（group.id）的所有用户只能有一个访问同一分区，从0.8.0版本开始，分区个数是可以动态增加，可以通过增加分区数来优化发送性能。
  此外分区也带来一个问题就是消息只是分区内部有序而不是全局有序的。如果需要全局有序，应用需要自己靠别的机制来保证。
  使用Pull模式拉取消息，消息的使用情况，比如是否还有consumer没有读取，是否重复读取(改进中)等，在Broker端也完全不跟踪维护，消息的过期处理简单的由定时器定时删除（比如保留7天），或者只保留最近100G的数据，由此简化各种消息跟踪维护的开销。
####②追求最大化的数据传输效率
生产者和消费者可以批量读写消息减少RPC开销
使用Zero Copy在内核层直接将文件内容传送给网络Socket，避免应用层数据拷贝
在传输消息前，对数据进行压缩
####③激进的内存管理模式
kafka不在JVM进程内部维护消息Cache，消息直接从文件中读写，完全依赖操作系统在文件系统层面的Cache，避免在JVM中管理Cache带来的额外数据结构开销和GC带来的性能代价。基于批量处理和顺序读写的应用模式，最大化利用文件系统的Cache机制和规避文件读写相对内存读写的性能代价。对系统页面缓存的需求大。
####④高可用性
Kafka的0.8.0版本Topic开始支持replicas。
bin/kafka-create-topic.sh   --replica 3 --partition 8 --topic test  --zookeeper localhost:2181
topic:test的每个partition会有3个备份,均衡负载在broker集群，其中一个leader，其他2个follower做备胎，leader负责message的读写。leader和follower的信息会记录在zookeeper中，如果作为leader的broker挂了，zookeeper会在2个follower中选举一个做leader,用来读写message。leader收到producer发送的message后，会有独立的线程把massage同步到follower。
####⑤数据一致性
之前提到replica不为1的情况下，原leader失去信号， zookeeper会选举一个follower作为新的leader代替原leader的读写工作。
那么怎么保证原来leader和新leader之间的数据一致性呢？
Kafka producer的ack有3中机制，初始化producer时的producerconfig可以通过配置request.required.acks不同的值来实现。
0：这意味着生产者producer不等待来自broker同步完成的确认继续发送下一条（批）消息。
此选项提供最低的延迟但最弱的耐久性保证（当服务器发生故障时某些数据会丢失，如leader已死，但producer并不知情，发出去的信息broker就收不到）。
1：这意味着producer在leader已成功收到的数据并得到确认后发送下一条message。
此选项提供了更好的耐久性为客户等待服务器确认请求成功（被写入死亡leader但尚未复制将失去了唯一的消息）。
-1：这意味着producer在follower副本确认接收到数据后才算一次发送完成。
此选项提供最好的耐久性，我们保证没有信息将丢失，只要至少一个同步副本保持存活。
三种机制，性能依次递减 (producer吞吐量降低)，数据健壮性则依次递增。