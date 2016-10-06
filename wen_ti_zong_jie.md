# 问题总结

1. Hadoop启动，出现JAVA_HOME is not set 的错误，原因是由于启动脚本ssh后环境变量丢失，  解决办法可以载hadoop-config.sh前source下环境变量的配置文件，或者载ssh后加载环境变量的文件。
   环境变量的问题可以参考：http://feihu.me/blog/2014/env-problem-when-ssh-executing-command-on-remote/
   
   