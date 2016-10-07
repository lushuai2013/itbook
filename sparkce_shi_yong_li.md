# spark测试用例


###1.运行 Spark 示例
在 ./examples/src/main 目录下有一些 Spark 的示例程序，有 Scala、Java、Python、R 等语言的版本。我们可以先运行一个示例程序 SparkPi（即计算 π 的近似值），执行如下命令：
 sh spark-submit --master yarn --deploy-mode client ../examples/src/main/python/pi.py