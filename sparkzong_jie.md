# spark总结
＃＃＃1.spark编译
有三种方式：SBT、MAVEN、make-distribution.sh。 SBT、MAVEN两种方式打出来的包比较大，不适合部署使用。因此我们通常使用第三种方式打包。
参考：
https://taoistwar.gitbooks.io/spark-operationand-maintenance-management/content/spark_install/spark_compile.html
###2.spark-shell yarn client 
16/10/07 16:23:05 ERROR spark.SparkContext: Error initializing SparkContext.
java.io.FileNotFoundException: File file:/tmp/spark-events does not exist

参考：
spark.eventLog.compress	false	是否压缩事件日志。需要spark.eventLog.enabled为true<br/>
spark.eventLog.dir	file:///tmp/spark-events	Spark事件日志记录的基本目录。在这个基本目录下，Spark为每个应用程序创建一个子目录。各个应用程序记录日志到直到的目录。用户可能想设置这为统一的地点，像HDFS一样，所以历史文件可以通过历史服务器读取<br/>
spark.eventLog.enabled	false	是否记录Spark的事件日志。这在应用程序完成后，重新构造web UI是有用的
属性参考：http://blog.javachen.com/2015/06/07/spark-configuration.html

###3.Spark on yarn（External Shuffle Service）
参考：
[http://spark.apache.org/docs/latest/job-scheduling.html](http://spark.apache.org/docs/latest/job-scheduling.html)
[http://ifeve.com/spark-schedule/](http://ifeve.com/spark-schedule/)
  Spark系统在运行含shuffle过程的应用时，Executor进程除了运行task，还要负责写shuffle 数据，给其他Executor提供shuffle数据。当Executor进程任务过重，导致GC而不能为其 他Executor提供shuffle数据时，会影响任务运行。
    这里实际上是利用External Shuffle Service 来提升性能，External shuffle Service是长期存在于NodeManager进程中的一个辅助服务。通过该服务 来抓取shuffle数据，减少了Executor的压力，在Executor GC的时候也不会影响其他 Executor的任务运行。

启用方法：
1. 在NodeManager中启动External shuffle Service。
  a. 在“yarn-site.xml”中添加如下配置项：
      <property>
          <name>yarn.nodemanager.aux-services</name>
          <value>spark_shuffle</value>
      </property>
      <property>
          <name>yarn.nodemanager.aux-services.spark_shuffle.class</name>
          <value>org.apache.spark.network.yarn.YarnShuffleService</value>
      </property>
      <property>
          <name>spark.shuffle.service.port</name>
          <value>7337</value>
      </property>
  配置参数描述
  yarn.nodemanager.aux-services  ：NodeManager中一个长期运行的辅助服务，用于提升Shuffle计算性能。
  yarn.nodemanager.auxservices.spark_shuffle.class ：NodeManager中辅助服务对应的类。
  spark.shuffle.service.port ：Shuffle服务监听数据获取请求的端口。可选配置，默认值为“7337”。
  b. 添加依赖的jar包
  拷贝“${SPARK_HOME}/lib/spark-1.3.0-yarn-shuffle.jar”到“${HADOOP_HOME}/share/hadoop/yarn/lib/”目录下。
  c. 重启NodeManager进程，也就启动了External shuffle Service。

2. Spark应用使用External shuffle Service。 

在“spark-defaults.conf”中必须添加如下配置项： 
spark.shuffle.service.enabled true 
spark.shuffle.service.port 7337 
说明 
1.如果1.如果“yarn.nodemanager.aux-services”配置项已存在，则在value中添加 “spark_shuffle”，且用逗号和其他值分开。 
2.“spark.shuffle.service.port”的值需要和上面“yarn-site.xml”中的值一样。 
配置参数描述 
spark.shuffle.service.enabled   ：NodeManager中一个长期运行的辅助服务，用于提升Shuffle 计算性能。默认为false，表示不启用该功能。 cank
spark.shuffle.service.port   ：Shuffle服务监听数据获取请求的端口。可选配置，默认值 为“7337”。
###4. 配置 Viewing After the Fact
 参考：http://ifeve.com/spark-monitor/
 开启history server需要如下指令：

```./sbin/start-history-server.sh```
注意history server 只展示已经结束的Spark作业。一种通知Spark作业结束的方法是，显式地关闭SparkContext（通过调用 sc.stop()，或者在 使用 SparkContext() 处理其 setup 和 tear down 事件（适用于python），然后作业历史就会出现在web UI上了。