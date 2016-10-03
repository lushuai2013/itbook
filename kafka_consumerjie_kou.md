# Kafka Consumer接口


1. Kafka提供了两种Consumer API
High Level Consumer API
Low Level Consumer API(Kafka诡异的称之为Simple Consumer API，实际上非常复杂)
在选用哪种Consumer API时，首先要弄清楚这两种API的工作原理，能做什么不能做什么，能做的话怎么做的以及用的时候，有哪些可能的问题
 
2. High Level Consumer API概述
High Level Consumer API围绕着Consumer Group这个逻辑概念展开，它屏蔽了每个Topic的每个Partition的Offset管理（自动读取zookeeper中该Consumer group的last offset ）、Broker失败转移以及增减Partition、Consumer时的负载均衡(当Partition和Consumer增减时，Kafka自动进行负载均衡）
对于多个Partition，多个Consumer
如果consumer比partition多，是浪费，因为kafka的设计是在一个partition上是不允许并发的，所以consumer数不要大于partition数
如果consumer比partition少，一个consumer会对应于多个partitions，这里主要合理分配consumer数和partition数，否则会导致partition里面的数据被取的不均匀。最好partiton数目是consumer数目的整数倍，所以partition数目很重要，比如取24，就很容易设定consumer数目
如果consumer从多个partition读到数据，不保证数据间的顺序性，kafka只保证在一个partition上数据是有序的，但多个partition，根据你读的顺序会有不同
增减consumer，broker，partition会导致rebalance，所以rebalance后consumer对应的partition会发生变化
High-level接口中获取不到数据的时候是会block的
关于Offset初始值的问题：
先produce一些数据，然后再用consumer读的话，需要加上一句offset读取设置
 
Java代码  收藏代码
props.put("auto.offset.reset", "smallest"); //必须要加，如果要读旧数据  
 
因为初始的offset默认是非法的，然后这个设置的意思 是，当offset非法时，如何修正offset，默认是largest，即最新，所以不加这个配置，你是读不到你之前produce的数据的，而且这个 时候你再加上smallest配置也没用了，因为此时offset是合法的，不会再被修正了，需要手工或用工具改重置offset
 
 
 
3. Low Level Consumer API概述
3.1Low Level Consumer API控制灵活性
Low Level Consumer API，作为底层的Consumer API，提供了消费Kafka Message更大的控制，如：
Read a message multiple times(重复读取）
Consume only a subset of the partitions in a topic in a process（跳读）
Manage transactions to make sure a message is processed once and only once（Exactly Once原语）
3.2 Low Level Consumer API的复杂性
软件没有银弹，Low Level Consumer API提供更大灵活控制是以复杂性为代价的：
Offset不再透明
Broker自动失败转移需要处理
增加Consumer、Partition、Broker需要自己做负载均衡
 
You must keep track of the offsets in your application to know where you left off consuming.（Offset自己管理）
You must figure out which Broker is the lead Broker for a topic and partition(如果一个Partition有多个副本，那么Lead Partition所在的Broker就称为这个Partition的Lead Broker)
You must handle Broker leader changes（Broker Leader是个什么概念）
3.3 使用Low Level Consumer API的步骤
Find an active Broker and find out which Broker is the leader for your topic and partition
Determine who the replica Brokers are for your topic and partition
Build the request defining what data you are interested in
Fetch the data
Identify and recover from leader changes



---






对于kafka的consumer接口，提供两种版本，

 

#high-level

一种high-level版本，比较简单不用关心offset, 会自动的读zookeeper中该Consumer group的last offset 
参考，https://cwiki.apache.org/confluence/display/KAFKA/Consumer+Group+Example

不过要注意一些注意事项，对于多个partition和多个consumer 
1. 如果consumer比partition多，是浪费，因为kafka的设计是在一个partition上是不允许并发的，所以consumer数不要大于partition数 
2. 如果consumer比partition少，一个consumer会对应于多个partitions，这里主要合理分配consumer数和partition数，否则会导致partition里面的数据被取的不均匀 
    最好partiton数目是consumer数目的整数倍，所以partition数目很重要，比如取24，就很容易设定consumer数目 
3. 如果consumer从多个partition读到数据，不保证数据间的顺序性，kafka只保证在一个partition上数据是有序的，但多个partition，根据你读的顺序会有不同 
4. 增减consumer，broker，partition会导致rebalance，所以rebalance后consumer对应的partition会发生变化 
5. High-level接口中获取不到数据的时候是会block的

简单版，

简单的坑，如果测试流程是，先produce一些数据，然后再用consumer读的话，记得加上第一句设置 
因为初始的offset默认是非法的，然后这个设置的意思是，当offset非法时，如何修正offset，默认是largest，即最新，所以不加这个配置，你是读不到你之前produce的数据的，而且这个时候你再加上smallest配置也没用了，因为此时offset是合法的，不会再被修正了，需要手工或用工具改重置offset

复制代码
        Properties props = new Properties();
        props.put("auto.offset.reset", "smallest"); //必须要加，如果要读旧数据
         props.put("zookeeper.connect", "localhost:2181");
        props.put("group.id", "pv");
        props.put("zookeeper.session.timeout.ms", "400");
        props.put("zookeeper.sync.time.ms", "200");
        props.put("auto.commit.interval.ms", "1000");
        
        ConsumerConfig conf = new ConsumerConfig(props);
        ConsumerConnector consumer = kafka.consumer.Consumer.createJavaConsumerConnector(conf);
        String topic = "page_visits";
        Map<String, Integer> topicCountMap = new HashMap<String, Integer>();
        topicCountMap.put(topic, new Integer(1));
        Map<String, List<KafkaStream<byte[], byte[]>>> consumerMap = consumer.createMessageStreams(topicCountMap);
        List<KafkaStream<byte[], byte[]>> streams = consumerMap.get(topic);
        
        KafkaStream<byte[], byte[]> stream = streams.get(0); 
        ConsumerIterator<byte[], byte[]> it = stream.iterator();
        while (it.hasNext()){
            System.out.println("message: " + new String(it.next().message()));
        }
        
        if (consumer != null) consumer.shutdown();   //其实执行不到，因为上面的hasNext会block
复制代码
在用high-level的consumer时，两个给力的工具，

1. bin/kafka-run-class.sh kafka.tools.ConsumerOffsetChecker --group pv

可以看到当前group offset的状况，比如这里看pv的状况，3个partition

Group           Topic                          Pid Offset          logSize         Lag             Owner 
pv              page_visits                    0   21              21              0               none 
pv              page_visits                    1   19              19              0               none 
pv              page_visits                    2   20              20              0               none

关键就是offset，logSize和Lag 
这里以前读完了，所以offset=logSize，并且Lag=0

2. bin/kafka-run-class.sh kafka.tools.UpdateOffsetsInZK earliest config/consumer.properties  page_visits

3个参数， 
[earliest | latest]，表示将offset置到哪里 
consumer.properties ，这里是配置文件的路径 
topic，topic名，这里是page_visits

我们对上面的pv group执行完这个操作后，再去check group offset状况，结果如下，

Group           Topic                          Pid Offset          logSize         Lag             Owner 
pv              page_visits                    0   0               21              21              none 
pv              page_visits                    1   0               19              19              none 
pv              page_visits                    2   0               20              20              none

可以看到offset已经被清0，Lag=logSize



#SimpleConsumer

另一种是SimpleConsumer，名字起的，以为是简单的接口，其实是low-level consumer，更复杂的接口

参考，https://cwiki.apache.org/confluence/display/KAFKA/0.8.0+SimpleConsumer+Example

什么时候用这个接口?

Read a message multiple times
Consume only a subset of the partitions in a topic in a process
Manage transactions to make sure a message is processed once and only once
 

当然用这个接口是有代价的，即partition,broker,offset对你不再透明，需要自己去管理这些，并且还要handle broker leader的切换，很麻烦 
所以不是一定要用，最好别用

You must keep track of the offsets in your application to know where you left off consuming.
You must figure out which Broker is the lead Broker for a topic and partition
You must handle Broker leader changes
使用SimpleConsumer的步骤：

Find an active Broker and find out which Broker is the leader for your topic and partition
Determine who the replica Brokers are for your topic and partition
Build the request defining what data you are interested in
Fetch the data
Identify and recover from leader changes
首先，你必须知道读哪个topic的哪个partition 
然后，找到负责该partition的broker leader，从而找到存有该partition副本的那个broker 
再者，自己去写request并fetch数据 
最终，还要注意需要识别和处理broker leader的改变

 

逐步来看，

Finding the Lead Broker for a Topic and Partition

思路就是，遍历每个broker，取出该topic的metadata，然后再遍历其中的每个partition metadata，如果找到我们要找的partition就返回 
根据返回的PartitionMetadata.leader().host()找到leader broker