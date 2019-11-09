linux机器
jdk1.8及以上【配置JAVA_HOME环境变量】
下载地址
https://mirrors.tuna.tsinghua.edu.cn/apache/flink/flink-1.6.1/flink-1.6.1-bin-hadoop27-scala_2.11.tgz
local模式快速安装启动
解压：tar -zxvf flink-1.6.1-bin-hadoop27-scala_2.11.tgz 
cd flink-1.6.1
启动：./bin/start-cluster.sh  
停止：./bin/stop-cluster.sh 
访问web界面
http://hostname:8081



编译
需要在pom文件中添加build配置，打包时指定入口类全类名【或者在运行时动态指定】
<scope>provided</scope>
mvn clean package
执行
1：在hadoop100上启动local flink集群
2：在hadoop100上执行 nc -l 9000
3：在hadoop100上执行./bin/flink run FlinkExample-xxxxxx.jar --port 9000
4：在hadoop100上执行tail -f log/flink-*-taskexecutor-*.out 查看日志输出
5：停止任务
1：web ui界面停止
2：命令行执行bin/flink cancel <job-id>



==打包编译说明==

```xml
<build>
        <plugins>
            <!-- 编译插件 -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.6.0</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
            <!-- scala编译插件 -->
            <plugin>
                <groupId>net.alchim31.maven</groupId>
                <artifactId>scala-maven-plugin</artifactId>
                <version>3.1.6</version>
                <configuration>
                    <scalaCompatVersion>2.11</scalaCompatVersion>
                    <scalaVersion>2.11.12</scalaVersion>
                    <encoding>UTF-8</encoding>
                </configuration>
                <executions>
                    <execution>
                        <id>compile-scala</id>
                        <phase>compile</phase>
                        <goals>
                            <goal>add-source</goal>
                            <goal>compile</goal>
                        </goals>
                    </execution>
                    <execution>
                        <id>test-compile-scala</id>
                        <phase>test-compile</phase>
                        <goals>
                            <goal>add-source</goal>
                            <goal>testCompile</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
            <!-- 打jar包插件(会包含所有依赖) -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-assembly-plugin</artifactId>
                <version>2.6</version>
                <configuration>
                    <descriptorRefs>
                        <descriptorRef>jar-with-dependencies</descriptorRef>
                    </descriptorRefs>
                    <archive>
                        <manifest>
                            <!-- 可以设置jar包的入口类(可选) -->
                            <mainClass>xuwei.tech.SocketWindowWordCountJava</mainClass>
                        </manifest>
                    </archive>
                </configuration>
                <executions>
                    <execution>
                        <id>make-assembly</id>
                        <phase>package</phase>
                        <goals>
                            <goal>single</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>

```

