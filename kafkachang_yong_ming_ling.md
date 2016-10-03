# kafka常用命令

####查看topic offset
/opt/App/kafka_2.11-0.10.0.1/bin$ sh kafka-consumer-offset-checker.sh --zookeeper lushuai:2181 --group group-1 --topic page_visits
[2016-10-03 23:39:50,750] WARN WARNING: ConsumerOffsetChecker is deprecated and will be dropped in releases following 0.9.0. Use ConsumerGroupCommand instead. (kafka.tools.ConsumerOffsetChecker$)
[2016-10-03 23:39:50,872] WARN Connected to an old server; r-o mode will be unavailable (org.apache.zookeeper.ClientCnxnSocket)
Group           Topic                          Pid Offset          logSize         Lag             Owner
group-1         page_visits                    0   300             300             0               group-1_lushuai-1475508000007-4cf45967-0