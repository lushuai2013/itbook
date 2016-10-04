# 使用新的Kafka消费者客户端

参考：http://zqhxuyuan.github.io/2016/02/20/Kafka-Consumer-New/


介绍了新kafka接口的设计原理及如何使用，不错的文档。

新的Comsumer API不再有high-level、low-level之分了，而是自己维护offset。这样做的好处是避免应用出现异常时，数据未消费成功，但Position已经提交，导致消息未消费的情况发生。通过查看API，新的Comsumer API有以下功能：

Kafka可以自行维护Offset、消费者的Position。也可以开发者自己来维护Offset，实现相关的业务需求。
消费时，可以只消费指定的Partitions
可以使用外部存储记录Offset，如数据库之类的。
自行控制Consumer消费消息的位置。
可以使用多线程进行消费
Kafka 0.9这个版本，大的变化主要就是上面三点了。不过对于这个版本的稳定性还有待在实际使用中进行观察。通过在JIRA中查看，目前这个版本的Bug只帖出了一个，估计跟没有大规模的被用户使用有关。如果小伙伴们在使用过程中有什么心得体会，欢迎在博客下面留言分享！