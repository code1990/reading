## kafka的Windows搭建及单机测试

### 1.1 安装JDK（见文档）

<p style="background-color: yellow;">reading\bigdata\099.安装教程\jdk-windows安装教程.md</p>
### 1.2 安装Zookeeper（见文档）

Kafka的运行依赖于Zookeeper，所以在运行Kafka之前我们需要安装并运行Zookeeper

<p style="background-color: yellow;">reading\bigdata\099.安装教程\zookeeper-windows安装教程.md</p>
### 1.3安装并运行Kafka

3.1 下载安装文件： http://kafka.apache.org/downloads.html

3.2 解压文件（本文解压到 G:\kafka_2.11-0.10.0.1）

3.3 打开G:\kafka_2.11-0.10.0.1\config

3.4 从文本编辑器里打开 server.properties

3.5 把log.dirs的值改成 “G:\kafka_2.11-0.10.0.1\kafka-logs”

3.6 打开cmd

3.7 进入kafka文件目录: cd /d G:\kafka_2.11-0.10.0.1\

**3.8 输入并执行以打开kafka:**

```
set classpath=.
.\bin\windows\kafka-server-start.bat.\config\server.properties
```

### 1.4创建topics

4.1 打开cmd 并进入G:\kafka_2.11-0.10.0.1\bin\windows

4.2 创建一个topic：

```
kafka-topics.bat --create --zookeeper localhost:2181--replication-factor 1 --partitions 1 --topic test 
```

5.打开一个Producer:

cd /d G:\kafka_2.11-0.10.0.1\bin\windows

```
kafka-console-producer.bat --broker-list localhost:9092 --topictest
```

6. 打开一个Consumer:

cd /d G:\kafka_2.11-0.10.0.1\bin\windows

```
kafka-console-consumer.bat --zookeeper localhost:2181 --topic test
```

然后就可以在Producer控制台窗口输入消息了。在消息输入过后，很快Consumer窗口就会显示出Producer发送的消息
