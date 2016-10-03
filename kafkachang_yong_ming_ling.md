# kafka常用命令

####查看kafka offset


```/opt/App/kafka_2.11-0.10.0.1/bin$ sh kafka-consumer-offset-checker.sh --zookeeper lushuai:2181 --group group-1 --topic page_visits```

![](2016-10-03 23:44:07屏幕截图.png)

####

####重置kafka的 offset
如果你在使用Kafka来分发消息，在数据处理的过程中可能会出现处理程序出异常或者是其它的错误，会造成数据丢失或不一致。这个时候你也许会想要通过kafka把数据从新处理一遍，我们知道kafka默认会在磁盘上保存到7天的数据，你只需要把kafka的某个topic的consumer的offset设置为某个值或者是最小值，就可以使该consumer从你设置的那个点开始消费。