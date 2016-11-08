
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
            </snapshots>    <build>
        <resources>
            <resource>
                <directory>src/main/resources</directory>
                <excludes>
                    <exclude>**/*</exclude>
                </excludes>
            </resource>
        </resources>

        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-dependency-plugin</artifactId>
                <version>2.8</version>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>copy-dependencies</goal>
                        </goals>
                        <configuration>
                            <outputDirectory>${project.build.directory}/lib</outputDirectory>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-war-plugin</artifactId>
                <configuration>
                    <webResources>
                        <resource>
                            <directory>src/main/resources</directory>
                            <targetPath>WEB-INF/classes</targetPath>
                        </resource>
                        <resource>
                            <directory>/home/lushuai/lib</directory>
                            <targetPath>WEB-INF/lib</targetPath>
                        </resource>
                    </webResources>
                    <warName>bds-rest-dbus</warName>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.3</version>
                <configuration>
                    <source>1.7</source>
                    <target>1.7</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
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

###6. Specify dependencies for Maven classifiers
[http://blog.csdn.net/lovingprince/article/details/5894459](http://blog.csdn.net/lovingprince/article/details/5894459)
how tow look for all dependencies pom :
https://github.com/bh-lushuai/Json-lib


###7. maven build 配置参考
 ```   <build>
        <resources>
            <resource>
                <directory>src/main/resources</directory>
                <excludes>
                    <exclude>**/*</exclude>
                </excludes>
            </resource>
        </resources>

        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-dependency-plugin</artifactId>
                <version>2.8</version>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>copy-dependencies</goal>
                        </goals>
                        <configuration>
                            <outputDirectory>${project.build.directory}/lib</outputDirectory>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-war-plugin</artifactId>
                <configuration>
                    <webResources>
                        <resource>
                            <directory>src/main/resources</directory>
                            <targetPath>WEB-INF/classes</targetPath>
                        </resource>
                        <resource>
                            <directory>/home/lushuai/lib</directory>
                            <targetPath>WEB-INF/lib</targetPath>
                        </resource>
                    </webResources>
                    <warName>bds-rest-dbus</warName>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.3</version>
                <configuration>
                    <source>1.7</source>
                    <target>1.7</target>
                </configuration>
            </plugin>
        </plugins>
    </build>```
###8.用maven assembly插件打jar包实现依赖包归档
参考：[http://blog.csdn.net/e5945/article/details/7777286](http://blog.csdn.net/e5945/article/details/7777286)
参考： [http://maven.apache.org/plugins-archives/maven-assembly-plugin-2.6/examples/sharing-descriptors.html](http://maven.apache.org/plugins-archives/maven-assembly-plugin-2.6/examples/sharing-descriptors.html)
```<plugin>
    <artifactId>maven-assembly-plugin</artifactId>
    <version>2.6</version>
    <configuration>
        <descriptorRefs>
            <descriptorRef>jar-with-dependencies</descriptorRef>
        </descriptorRefs>
    </configuration>
</plugin>```
mvn assembly:assembly命令你会在${project}/target 文件夹下发现新生成的 {artifactId}-jar-with-dependencies.jar 这个文件
在上面的这个 命令执行的过程中，maven会将jar包所依赖的包导出，并且解压（unpackage），一并放在
这个{artifactId}-jar-with-dependencies.jar 包中


各个第三方包单独部署，比如ibatis归ibatis包，spring包归spring包，这样以后单独为某个第三方包升级也比较方便。

这个jar-with-dependencies是assembly预先写好的一个，组装描述引用（assembly descriptor）我们来看一下这个定义这个组装描述（assemly descriptor）的xml文件

```<assembly xmlns="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.0"  
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
  xsi:schemaLocation="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.0 http://maven.apache.org/xsd/assembly-1.1.0.xsd">  
  <id>jar-with-dependencies</id>  
  <formats>  
    <format>jar</format>  
  </formats>  
  <includeBaseDirectory>false</includeBaseDirectory>  
  <dependencySets>  
    <dependencySet>  
      <unpack>true</unpack>  
      <scope>runtime</scope>  
    </dependencySet>  
  </dependencySets>  
  <fileSets>  
    <fileSet>  
      <directory>${project.build.outputDirectory}</directory>  
    </fileSet>  
  </fileSets>  
</assembly>  ```

在 main/assembly 下创建 src.xml文件，将刚才修改过的内用写入文件中，内容为：
```<assembly  
    xmlns="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.0"  
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
    xsi:schemaLocation="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.0 http://maven.apache.org/xsd/assembly-1.1.0.xsd">  
    <id>jar-with-dependencies</id>  
    <formats>  
        <format>jar</format>  
    </formats>  
    <includeBaseDirectory>false</includeBaseDirectory>  
    <dependencySets>  
        <dependencySet>  
            <unpack>false</unpack>  
            <scope>runtime</scope>  
        </dependencySet>  
    </dependencySets>  
    <fileSets>  
        <fileSet>  
            <directory>${project.build.outputDirectory}</directory>  
        </fileSet>  
    </fileSets>  
</assembly>  ```
将之前pom.xml 中的plugin改成如下：
           ```<plugin>  
                <artifactId>maven-assembly-plugin</artifactId>  
                <configuration>  
                    <descriptors>  
                        <descriptor>src/main/assembly/src.xml</descriptor>  
                    </descriptors>  
                </configuration>  
            </plugin>  ```
            
###9.  解决在IDEA14 的Maven下 出现 Cannot access in offline mode 问题
Working in Offline Mode   :[https://www.jetbrains.com/help/idea/2016.2/working-in-offline-mode.html](https://www.jetbrains.com/help/idea/2016.2/working-in-offline-mode.html)
On each launch, Maven visits the remote repositories and checks for updates. Executing a Maven command can result in downloading new archives and changing Maven itself. When you switch to offline mode, Maven has to use those resources that are available locally, and report about the problems, if something is missing.

The offline mode is helpful, when you need to work offline, or when your network connection is slow.

解决方法：http://blog.csdn.net/lisq037/article/details/43935165


###9. maven assembly例子
参考   https://github.com/bh-lushuai/dubbox/tree/master/dubbo-demo/dubbo-demo-consumer/src/main

###10. guo配置私服示例

<?xml version="1.0" encoding="UTF-8"?>

<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0" 
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">

  <localRepository>D:\softs\m2</localRepository>
 
  <pluginGroups>
  </pluginGroups>
  <proxies>
  </proxies>
  <servers>
  </servers>

  <mirrors>
    <mirror>  
        <id>nexus</id>  
        <mirrorOf>central</mirrorOf>  
        <name>Nexus osc</name>  
        <url>http://maven.aliyun.com/nexus/content/groups/public/</url>  
    </mirror>  
  </mirrors>
 
    <profiles>
          <profile>  
                <id>gou</id>  
                <activation>  
                    <jdk>1.4</jdk>  
                </activation>  
                <repositories>  
                    <repository>  
                        <id>gou</id>  
                        <name>local private nexus</name>  
                        <url>http://10.*.*.*:8081/nexus/content/groups/public/</url>  
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
                        <id>gou</id>  
                        <name>local private nexus</name>  
                        <url>http://10.*.*.*:8081/nexus/content/groups/public/</url>  
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
        <activeProfile>gou</activeProfile>
    </activeProfiles>
</settings>
