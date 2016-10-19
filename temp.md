#spark 

1.Spark快速入门指南 – Spark安装与基础使用
http://www.powerxing.com/spark-quick-start-guide/
2.spark app
  http://colobu.com/2014/12/08/spark-quick-start/
  
3.使用IDEA开发Spark应用
http://debugo.com/idea-spark/ <br/>



------



#storm
1. Apache Storm技术实战之1 -- WordCountTopology
 http://www.cnblogs.com/hseagle/p/3505938.html 
 
 
 ###大数据demo
 https://github.com/mapr-demos
 
 
 ###Google Guava 库用法整理
 http://macrochen.iteye.com/blog/737058
 
 
 ###JVM原理讲解和调优
 http://www.mamicode.com/info-detail-1028149.html
 
 ###$spark
 http://ifeve.com/spark-sql-dataframes/
 
 
 ##spark
 1. 探究原理（统一Java和Scala API），底层如何实现api统一的。

在Spark 1.3之前，有单独的java兼容类（JavaSQLContext和JavaSchemaRDD）及其在Scala API中的镜像。Spark 1.3中将Java API和Scala API统一。两种语言的用户都应该使用SQLContext和DataFrame。一般这些类中都会使用两种语言中都有的类型（如：Array取代各语言独有的集合）。有些情况下，没有通用的类型（例如：闭包或者maps），将会使用函数重载来解决这个问题。

另外，java特有的类型API被删除了。Scala和java用户都应该用org.apache.spark.sql.types来编程描述一个schema。