# Spark 1.6.1测试环境 部署
Spark Thrift Server部署在test-002上
编译Spark 1.6.1代码，需要添加如下两个maven依赖（编译后的spark安装包在test-002, /export/App/spark-1.6.1-bin-2.6.1.1.tgz）
编译脚本：./make-distribution.sh --name 2.6.1.1 --tgz -Phadoop-2.6 -Phive -Phive-thriftserver -Pyarn  -Dhadoop.version=2.6.1.1 -Pspark-ganglia-lgpl
1. Caused by: java.lang.ClassNotFoundException: org.apache.hadoop.hive.schshim.FairSchedulerShim

在spark-hive-thriftserver模块的pom.xml中添加如下依赖

<dependency>
<groupId>org.apache.hive.shims</groupId>
<artifactId>hive-shims-scheduler</artifactId>
<version>1.2.1</version>
</dependency>

2. Caused by: java.lang.ClassNotFoundException: org.apache.hadoop.yarn.server.resourcemanager.scheduler.fair.AllocationFileLoaderService$Listener

在spark-yarn模块的pom.xml文件中添加如下依赖

<dependency>
<groupId>org.apache.hadoop</groupId>
<artifactId>hadoop-yarn-server-resourcemanager</artifactId>
<version>2.6.1</version>
</dependency>


3. 包含patch 
https://github.com/apache/spark/pull/9113 


4. lzo问题
两种方法：
a. 在spark-env.sh中配置classpatah
b. 在启动spark thrift server时指定--jar
./sbin/start-thriftserver.sh  --hiveconf hive.server2.thrift.port=10001 --jars /export/App/hadoop-2.6.1-security/lib/hadoop-lzo.jar


5. Spark相关的配置(在spark-default.conf中）

###3rd party jar and library dependencies
spark.executor.extraLibraryPath=/export/App/hadoop-2.6.1-security/lib/native
spark.driver.extraLibraryPath=/export/App/hadoop-2.6.1-security/lib/native
spark.executor.extraClassPath=/export/App/hadoop-2.6.1-security/lib/hadoop-lzo.jar:/export/App/apache-hive-1.2.1-bin/lib/hive-shims-scheduler-1.2.1.jar:/export/App/hadoop-2.6.1-security/share/hadoop/common/lib/guava-11.0.2.jar
spark.driver.extraClassPath=/export/App/hadoop-2.6.1-security/lib/hadoop-lzo.jar:/export/App/apache-hive-1.2.1-bin/lib/hive-shims-scheduler-1.2.1.jar:/export/App/hadoop-2.6.1-security/share/hadoop/common/lib/guava-11.0.2.jar

###history server kerberos authentication，use hive
spark.history.kerberos.enabled true
spark.history.kerberos.principal hive/bds-test-002@HADOOP.JD
spark.history.kerberos.keytab /export/keytabs/hive.keytab

####Spark YARN authentication，use hive
spark.yarn.principal    hive/bds-test-002@HADOOP.JD
spark.yarn.keytab       /export/keytabs/hive.keytab
spark.yarn.security.tokens.hive.enabled true




6.测试验证
1). 使用hive账户启动spark thrift server，
local模式：./sbin/start-thriftserver.sh --hiveconf hive.server2.thrift.port=10001 --jars /export/App/hadoop-2.6.1-security/lib/hadoop-lzo.jar

yarn-client模式(Spark Thrift Server不支持yarn-cluster模式)： ./start-thriftserver.sh --master yarn-client --num-executors 4 --executor-memory 4G --driver-memory 4G --hiveconf hive.server2.thrift.port=10001 --jars /export/App/hadoop-2.6.1-security/lib/hadoop-lzo.jar

2). 使用hive启动beeline, ./beeline
beeline>!connect jdbc:hive2://localhost:10001/default;principal=hive/bds-test-002@HADOOP.JD;kerberosAuthType=kerberos;hive.server2.proxy.user=test107

3). 执行sql语句
a. create table t1(line string)
b. load data local inpath '/home/hive/abc.txt' into table t1;
c. create table t2(line string) as select * from t1;
d. create table t3(line string)
e. load data local inpath '/home/hive/def.txt' into table t3
f. insert overwrite table t2 select * from t3

4)Verify https://issues.apache.org/jira/browse/SPARK-11075



