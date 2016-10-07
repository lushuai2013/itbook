# spark总结
＃＃＃1.spark编译
有三种方式：SBT、MAVEN、make-distribution.sh。 SBT、MAVEN两种方式打出来的包比较大，不适合部署使用。因此我们通常使用第三种方式打包。
参考：
https://taoistwar.gitbooks.io/spark-operationand-maintenance-management/content/spark_install/spark_compile.html
###2.spark-shell yarn client 
16/10/07 16:23:05 ERROR spark.SparkContext: Error initializing SparkContext.
java.io.FileNotFoundException: File file:/tmp/spark-events does not exist
