# spark测试用例


###1.运行 Spark 示例
在 ./examples/src/main 目录下有一些 Spark 的示例程序，有 Scala、Java、Python、R 等语言的版本。我们可以先运行一个示例程序 SparkPi（即计算 π 的近似值），执行如下命令：
 
``` 1. ./run-example --master yarn --deploy-mode client SparkPi
###2. 使用spark-submit提交python 版SparkPi
 sh spark-submit --master yarn --deploy-mode client ../examples/src/main/python/pi.py```
 
###3.使用awk和spark统计文本文件每行出现单词最多行的单词数
####awk：
cat README.md| awk '{lens=split($0,as," ");print lens;}' |sort -nr|head -1

split用法:The awk function split(s,a,sep) splits a string s into an awk array a using the delimiter sep.<br/>
set time = 12:34:56<br/>
set hr = `echo $time | awk '{split($0,a,":" ); print a[1]}'` # = 12<br/>
set sec = `echo $time | awk '{split($0,a,":" ); print a[3]}'` # = 56<br/>

sort用法：<br/>
注:-k为排序关键列 <br/>
-r为降序排序<br/>
-n按算术值对数字字段排序。数字字段可包含前导空格、可选减号、十进制数字、千分位分隔符和可选基数符。对包含任何非数字字符的字段进行数字排序会出现无法预知的结果。<br/>

head -1 为：取第一条记录

####spark
textFile.map(line => line.split(" ").size).reduce((a, b) => if (a > b) a else b)


###wordcount
####awk
cat anyfile.txt|awk -F " " '{for(i=1;i<=NF;i++) a[$i]++} END {for(k in a) print k,a[k]}'
参考 http://stackoverflow.com/questions/28628667/word-count-using-awk <br/>
-F " " 以空格分割单词<br/>
NF  在awk中大家都知道NF的作用，它是一个awk的内建变量，代表是每行的字段数量。常用的几种方式我给大家慢慢到来。最多的就是在读取每个字段内容 for(i=1;i<=NF;i++) 这个运用非常之多<br/>

a是分割数组

####spark
val wordcount=textfile.flatMap(line=>line.split(" ")).map(word=> (word,1)).reduceByKey((a,b)=>(a+b))
wordcount.collect()


 
 