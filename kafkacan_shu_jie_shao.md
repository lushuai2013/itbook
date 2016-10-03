# kafka参数介绍

参考，

https://cwiki.apache.org/confluence/display/KAFKA/0.8.0+Producer+Example

http://kafka.apache.org/08/configuration.html , 0.8版本，关于producer，consumer，broker所有的配置

 

因为Producer相对于consumer比较简单，直接看代码，需要注意的点

1. 配置参数，详细参考上面链接 
    1.1 metadata.broker.list， 不同于0.7，不需要给出zk的地址，而是给出一些broker地址，不用全部，这里建议给两个防止一个不可用 
          Kafka会自己找到相应topic，partition的leader broker 
    1.2 serializer.class，需要给出message的序列化的encoder，这里使用的是简单的StringEncoder 
          并且对于key还可以单独的设定，"key.serializer.class"  
          注意，除非明确知道message编码，否则不要直接使用StringEncoder， 
          因为源码中的逻辑是如果没有在初始化时指定编码会默认按UTF8转码，会导致乱码 
          所以不明确的时候，不要指定serializer.class，默认的encoder逻辑是直接将byte[]放入broker，不会做转码 
    1.3 partitioner.class，可以不设置，默认就是random partition，当然这里可以自定义，如何根据key来选择partition 
    1.4 request.required.acks， 是否要求broker给出ack，如果不设置默认是'fire and forget'， 会丢数据 
          默认为0，即和0.7一样，发完不会管是否成功，lowest latency but the weakest durability 
          1, 等待leader replica的ack，否则重发，折中的方案，当leader在同步数据前dead，会丢数据 
          -1，等待all in-sync replicas的ack，只要有一个replica活着，就不会丢数据 
    1.5 producer.type，  
         sync，单条发送 
         async，buffer一堆请求后，再一起发送 
         如果不是对丢数据非常敏感，请设为async，因为对throughput帮助很大，但是当client crash时，会丢数据 
    1.6 compression.codec 
         支持"none", "gzip" and "snappy" 
         可以通过，compressed.topics，来指定压缩的topic

    当producer.type选择async的时候，需要关注如下配置 
    queue.buffering.max.ms (5000), 最大buffer数据的时间，默认是5秒 
    batch.num.messages (200), batch发送的数目，默认是200，producer会等待buffer的messages数目达到200或时间超过5秒，才发送数据 
    queue.buffering.max.messages (10000), 最多可以buffer的message数目，超过要么producer block或把数据丢掉 
    queue.enqueue.timeout.ms (-1), 默认是-1，即达到buffer最大meessage数目时，producer会block 
                                                       设为0，达到buffer最大meessage数目时会丢掉数据

 

2. Producer发送的是kv数据 
无论Producer或KeyedMessage都是<String, String>的泛型，这里是指key和value的类型

复制代码
import java.util.*;
 
import kafka.javaapi.producer.Producer;
import kafka.producer.KeyedMessage;
import kafka.producer.ProducerConfig;
 
public class TestProducer {
    public static void main(String[] args) {
        long events = Long.parseLong(args[0]);
        Random rnd = new Random();
 
        Properties props = new Properties();
        props.put("metadata.broker.list", "host1:9092, host2:9092 "); //
        props.put("serializer.class", "kafka.serializer.StringEncoder");
        props.put("partitioner.class", "example.producer.SimplePartitioner"); //可以不设置
        props.put("request.required.acks", "1");
 
        ProducerConfig config = new ProducerConfig(props);
 
        Producer<String, String> producer = new Producer<String, String>(config);
 
        for (long nEvents = 0; nEvents < events; nEvents++) { 
               long runtime = new Date().getTime();  
               String ip = “192.168.2.” + rnd.nextInt(255); 
               String msg = runtime + “,www.example.com,” + ip; 
               KeyedMessage<String, String> data = new KeyedMessage<String, String>("page_visits", ip, msg); //指定topic，key，value
               producer.send(data);
        }
        producer.close();
    }
}

 

对于自定义partitioner也很简单，

对于partition，两个参数，key和partitions的数目 
所要完成的逻辑就是，如果根据key在partitions中挑选一个合适的partition

复制代码
import kafka.producer.Partitioner;
import kafka.utils.VerifiableProperties;
 
public class SimplePartitioner implements Partitioner {
    public SimplePartitioner (VerifiableProperties props) {
 
    }
 
    public int partition(String key, int a_numPartitions) {
        int partition = 0;
        int offset = key.lastIndexOf('.');
        if (offset > 0) {
           partition = Integer.parseInt( key.substring(offset+1)) % a_numPartitions;
        }
       return partition;
  }
 
}

