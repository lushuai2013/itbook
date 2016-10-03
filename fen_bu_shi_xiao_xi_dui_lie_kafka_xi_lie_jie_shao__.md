# 分布式消息队列kafka系列介绍 — 核心API介绍及实例


参考：http://www.inter12.org/archives/834
一 PRODUCER的API
1.Producer的创建，依赖于ProducerConfig
public Producer(ProducerConfig config);
2.单个或是批量的消息发送
public void send(KeyedMessage<K,V> message);
public void send(List<KeyedMessage<K,V>> messages);
3.关闭Producer到所有broker的连接
public void close();
二 CONSUMER的高层API
主要是Consumer和ConsumerConnector，这里的Consumer是ConsumerConnector的静态工厂类
class Consumer {
public static kafka.javaapi.consumer.ConsumerConnector createJavaConsumerConnector(config: ConsumerConfig);
}
具体的消息的消费都是在ConsumerConnector中
创建一个消息处理的流，包含所有的topic，并根据指定的Decoder
public <K,V> Map<String, List<KafkaStream<K,V>>>
createMessageStreams(Map<String, Integer> topicCountMap, Decoder<K> keyDecoder, Decoder<V> valueDecoder);
创建一个消息处理的流，包含所有的topic，使用默认的Decoder
public Map<String, List<KafkaStream<byte[], byte[]>>> createMessageStreams(Map<String, Integer> topicCountMap);
获取指定消息的topic,并根据指定的Decoder
public <K,V> List<KafkaStream<K,V>>
createMessageStreamsByFilter(TopicFilter topicFilter, int numStreams, Decoder<K> keyDecoder, Decoder<V> valueDecoder);
获取指定消息的topic,使用默认的Decoder
public List<KafkaStream<byte[], byte[]>> createMessageStreamsByFilter(TopicFilter topicFilter);
提交偏移量到这个消费者连接的topic
public void commitOffsets();
关闭消费者
public void shutdown();
高层的API中比较常用的就是public List<KafkaStream<byte[], byte[]>> createMessageStreamsByFilter(TopicFilter topicFilter);和public void commitOffsets();
三 CONSUMER的简单API–SIMPLECONSUMER
批量获取消息
public FetchResponse fetch(request: kafka.javaapi.FetchRequest);
获取topic的元信息
public kafka.javaapi.TopicMetadataResponse send(request: kafka.javaapi.TopicMetadataRequest);
获取目前可用的偏移量
public kafka.javaapi.OffsetResponse getOffsetsBefore(request: OffsetRequest);
关闭连接
public void close();
对于大部分应用来说，高层API就已经足够使用了，但是若是想做更进一步的控制的话，可以使用简单的API，例如消费者重启的情况下，希望得到最新的offset，就该使用SimpleConsumer.
四 KAFKA HADOOP CONSUMER API
提供了一个可水平伸缩的解决方案来结合hadoop的使用参见
https://github.com/linkedin/camus/tree/camus-kafka-0.8/

实战参考原文：http://www.inter12.org/archives/834

