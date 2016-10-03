# kafka常用命令

####查看kafka offset


```/opt/App/kafka_2.11-0.10.0.1/bin$ sh kafka-consumer-offset-checker.sh --zookeeper lushuai:2181 --group group-1 --topic page_visits```

![](2016-10-03 23:44:07屏幕截图.png)

####查询topic的offset的范围
用下面命令可以查询到topic:page_visits broker:localhost:9092的offset的最小值：

```/bin/kafka-run-class.sh kafka.tools.GetOffsetShell --broker-list localhost:9092 —topic page_visits --time -2
输出： DynamicRange:0:10


从上面的输出可以看出topic:DynamicRange只有一个partition:0 offset范围为:[10,128]

查询offset的最大值：
bin/kafka-run-class.sh kafka.tools.GetOffsetShell --broker-list localhost:9092 —topic page_visits --time -1
输出：DynamicRange:0:128```

####重置kafka的 offset
如果你在使用Kafka来分发消息，在数据处理的过程中可能会出现处理程序出异常或者是其它的错误，会造成数据丢失或不一致。这个时候你也许会想要通过kafka把数据从新处理一遍，我们知道kafka默认会在磁盘上保存到7天的数据，你只需要把kafka的某个topic的consumer的offset设置为某个值或者是最小值，就可以使该consumer从你设置的那个点开始消费。

启动zookeeper client

$ /opt/cloudera/parcels/CDH/lib/zookeeper/bin/zkCli.sh
通过下面命令设置consumer group:DynamicRangeGroup topic:localhost partition:10的offset为128:

set /consumers/group-1/offsets/page_visits/0 128
注意如果你的kafka设置了zookeeper root，比如为/kafka，那么命令应该改为：

set /kafka/consumers/group-1/offsets/page_visits/0 128

###在用high-level的consumer时，两个给力的工具，

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