# dubbo总结

###1. dubbo 使用zookeeper实现服务发现
1.启动provider zookeeper生成metadata
ls /dubbo                                       
[com.znn.provider.DemoService]

ls /dubbo/com.znn.provider.DemoService          
[providers, configurators]


ls /dubbo/com.znn.provider.DemoService/providers
[dubbo%3A%2F%2F192.168.242.1%3A20880%2Fcom.znn.provider.DemoService%3Fanyhost%3Dtrue%26application%3Dhello-world-app%26dubbo%3D2.5.3%26interface%3Dcom.znn.provider.DemoService%26methods%3DsayHello%26pid%3D10116%26side%3Dprovider%26timestamp%3D1477378243524]

2.consumer启动
 ls /dubbo/com.znn.provider.DemoService 
[consumers, routers, providers, configurators]

[zk: localhost:2181(CONNECTED) 60] ls /dubbo/com.znn.provider.DemoService/consumers
[]
[zk: localhost:2181(CONNECTED) 61] ls /dubbo/com.znn.provider.DemoService/routers  
[]

###2. dubbo 原理解析
[http://blog.csdn.net/u010311445/article/category/2745121](http://blog.csdn.net/u010311445/article/category/2745121)


