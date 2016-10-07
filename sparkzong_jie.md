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
