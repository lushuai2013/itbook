# 问题总结

###1. Hadoop启动，出现JAVA_HOME is not set 的错误，
原因是由于启动脚本ssh后环境变量丢失，  解决办法可以载hadoop-config.sh前source下环境变量的配置文件，或者载ssh后加载环境变量的文件。
   环境变量的问题可以参考：http://feihu.me/blog/2014/env-problem-when-ssh-executing-command-on-remote/
   
###2.Hadoop为什么没有采用RAID？

HDFS集群没有采用RAID(冗余磁盘阵列)作为datanodes的存储设备(尽管namenode使用RAID来保护元数据不会丢失)。由于HDFS是采用在节点之间块复制的方法，所以RAID提供的冗余机制对HDFS来说是多余的。
此外，RAID条带(RAID 0)常用于增加性能，但却比HDFS中用到的JBOD(Just a Bunch Of Disks)要慢，而且JBOD在所有的磁盘之间对HDFS块进行时间片的轮转。具体说，RAID 0读写操作受限于冗余磁盘阵列中最慢的那个磁盘的速度。在JBOD中，磁盘的操作是独立的，所以读写操作的平均速度要大于最慢磁盘的速度。实际应用中，磁盘性能多是可以改变的，即使是同一型号的磁盘。在Yahoo Hadoop Cluster的Benchmark中，测试Gridmix显示JBOD要比RAID 0快10%，另一个测试显示快30%(这里的测试指的是HDFS的写能力。
最后，当一个JBOD配置中的一个磁盘失效，HDFS可以继续操作；但是在RAID中，一个磁盘的失效将会导致整个阵列(节点也一样)变得不再可用。

翻译自：OReilly Hadoop 《The Definitive Guide》June 2009
   
   