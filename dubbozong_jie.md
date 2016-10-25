# dubbo总结

###1. dubbo 使用zookeeper实现服务发现
1.启动provider zookeeper生成metadata
ls /dubbo                                       
[com.znn.provider.DemoService]

ls /dubbo/com.znn.provider.DemoService          
[providers, configurators]


ls /dubbo/com.znn.provider.DemoService/providers
[dubbo%3A%2F%2F192.168.242.1%3A20880%2Fcom.znn.provider.DemoService%3Fanyhost%3Dtrue%26application%3Dhello-world-app%26dubbo%3D2.5.3%26interface%3Dcom.znn.provider.DemoService%26methods%3DsayHello%26pid%3D10116%26side%3Dprovider%26timestamp%3D1477378243524]



