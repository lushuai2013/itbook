
# maven

###1.maven镜像配置参考：
https://yq.aliyun.com/articles/46991

摘要： 综述 用maven做项目，最郁闷的莫过于某些依赖库下载不了。被墙了，你懂的。使用maven镜像仓库及其重要，特别是国内的镜像，可以有效缓解被墙疼痛。 常用的镜像 国外镜像 ibiblio.org <mirror> <id>ibiblio</id> <mirrorOf>c...

综述
用maven做项目，最郁闷的莫过于某些依赖库下载不了。被墙了，你懂的。使用maven镜像仓库及其重要，特别是国内的镜像，可以有效缓解被墙疼痛。

常用的镜像
国外镜像
ibiblio.org

<mirror>  
     <id>ibiblio</id>  
     <mirrorOf>central</mirrorOf>  
     <name>ibiblio Mirror of http://repo1.maven.org/maven2/</name>  
     <url>http://mirrors.ibiblio.org/pub/mirrors/maven2/</url>  
</mirror>  
jboss

<mirror>  
     <id>jboss-public-repository-group</id>  
     <mirrorOf>central</mirrorOf>  
     <name>JBoss Public Repository Group</name>  
     <url>http://repository.jboss.org/nexus/content/groups/public</url>  
</mirror>
repo2

<mirror>
  <id>repo2</id>
  <mirrorOf>central</mirrorOf>
  <name>Human Readable Name for this Mirror.</name>
  <url>http://repo2.maven.org/maven2/</url>
</mirror>
uk.maven.org

<mirror>
  <id>ui</id>
  <mirrorOf>central</mirrorOf>
  <name>Human Readable Name for this Mirror.</name>
 <url>http://uk.maven.org/maven2/</url>
</mirror>
国内镜像
oschina.net

<mirror>
    <id>nexus-osc</id>
    <mirrorOf>*</mirrorOf>
    <name>Nexus osc</name>
    <url>http://maven.oschina.net/content/groups/public/</url>
</mirror>
net.cn

<mirror>
    <id>net-cn</id>
    <mirrorOf>central</mirrorOf>
    <name>Human Readable Name for this Mirror.</name>
    <url>http://maven.net.cn/content/groups/public/</url>   
 </mirror>
使用镜像
下文以oschina.net的镜像为例子.

1.Maven 的安装目录下的 conf 文件下有个settings.xml文件,编辑该文件

2.在<mirrors>中插入:

<mirror>
    <id>nexus-osc</id>
    <mirrorOf>*</mirrorOf>
    <name>Nexus osc</name>
    <url>http://maven.oschina.net/content/groups/public/</url>
</mirror>
3.这里是配置 Maven 的 mirror 地址指向OSChina 的 Maven 镜像地址。 在执行 Maven 命令的时候， Maven 还需要安装一些插件包，这些插件包的下载地址也让其指向 oschina.net 的 Maven 地址。在<profiles>中插入:

<profile>
    <id>jdk-1.4</id>
    <activation>
    <jdk>1.4</jdk>
    </activation>
    <repositories>
        <repository>
            <id>nexus</id>
            <name>local private nexus</name>
            <url>http://maven.oschina.net/content/groups/public/</url>
            <releases>
                <enabled>true</enabled>
            </releases>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>
    </repositories>
    <pluginRepositories>
        <pluginRepository>
            <id>nexus</id>
            <name>local private nexus</name>
            <url>http://maven.oschina.net/content/groups/public/</url>
            <releases>
                <enabled>true</enabled>
            </releases>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </pluginRepository>
    </pluginRepositories>
</profile>
参考：http://maven.oschina.net/help.html

###2. maven配置示例
     
```  <mirrors> 
    <mirror>  
        <id>nexus</id>  
        <mirrorOf>central</mirrorOf>  
        <name>Nexus osc</name>  
        <url>http://maven.aliyun.com/nexus/content/groups/public/</url>  
    </mirror>  
    <mirror>  
        <id>nexus-thirdparty</id>  
        <mirrorOf>thirdparty</mirrorOf>  
        <name>Nexus osc thirdparty</name>  
        <url>http://maven.oschina.net/content/repositories/thirdparty/</url>  
    </mirror>  
  </mirrors>```
  
  
     <profiles>
        <profile>  
                <id>nexus</id>  
              
                <activation>  
                    <jdk>1.4</jdk>  
                </activation>  
              
                <repositories>  
                    <repository>  
                        <id>nexus</id>  
                        <name>local private nexus</name>  
                        <url>http://maven.aliyun.com/nexus/content/groups/public/</url>  
                        <releases>  
                            <enabled>true</enabled>  
                        </releases>  
                        <snapshots>  
                            <enabled>false</enabled>  
                        </snapshots>  
                    </repository>  
                </repositories>  
                <pluginRepositories>  
                    <pluginRepository>  
                        <id>nexus</id>  
                        <name>local private nexus</name>  
                        <url>http://maven.aliyun.com/nexus/content/groups/public/</url>  
                        <releases>  
                            <enabled>true</enabled>  
                        </releases>  
                        <snapshots>  
                            <enabled>false</enabled>  
                        </snapshots>  
                    </pluginRepository>  
                </pluginRepositories>  
          </profile>
    </profiles>

    <activeProfiles>
        <activeProfile>nexus</activeProfile>
    </activeProfiles>

###3. maven Maven实战（九）——打包的技巧
参考：[http://www.infoq.com/cn/news/2011/06/xxb-maven-9-package](http://www.infoq.com/cn/news/2011/06/xxb-maven-9-package)
###4. maven maven-assembly-plugin 参数配置参考：[http://maven.apache.org/plugins/maven-assembly-plugin/assembly.html](http://maven.apache.org/plugins/maven-assembly-plugin/assembly.html)

###5. 使用Maven为一个项目生成多个Jar包
[http://agiledon.github.io/blog/2013/11/10/create-two-jars-from-one-project-using-maven/](http://agiledon.github.io/blog/2013/11/10/create-two-jars-from-one-project-using-maven/)
