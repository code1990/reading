## Kafka集群部署

### 1.环境准备

#### 1.1 集群规划

| hadoop102 | hadoop103 | hadoop104 |
| --------- | --------- | --------- |
| zk        | zk        | zk        |
| kafka     | kafka     | kafka     |

#### 1.2下载kafka

http://kafka.apache.org/downloads.html

#### 1.3虚拟机准备

- 配置ip地址 见教程
<p style="background-color: yellow;">reading\bigdata\099.安装教程\linux\ip-linux修改静态ip.md</p>


- 配置主机名称
<p style="background-color: yellow;">reading\bigdata\099.安装教程\linux\p-linux配置主机名</p>

- 3台主机分别关闭防火墙

```shell
chkconfig iptables off
chkconfig iptables off
chkconfig iptables off
```

#### 1.4安装jdk

<p style="background-color: yellow;">reading\bigdata\099.安装教程\linux\jdk-linux安装教程.md</p>

#### 1.5安装zookeeper

<p style="background-color: yellow;">reading\bigdata\099.安装教程\linux\zookeeper-linux安装教程.md</p>

-------------

### 2.kafka集群环境搭建

```shell
#1）解压安装包

 tar -zxvf kafka_2.11-0.11.0.0.tgz -C /opt/module/

#2）修改解压后的文件名称

 mv kafka_2.11-0.11.0.0/ kafka

#3）在/opt/module/kafka目录下创建logs文件夹

 mkdir logs

#4）修改配置文件

 cd config/

 vi server.properties

#主要配置如下内容：
#broker的全局唯一编号，不能重复
broker.id=0
#删除topic功能使能
delete.topic.enable=true
#kafka运行日志存放的路径
log.dirs=/opt/module/kafka/logs
#配置连接Zookeeper集群地址
zookeeper.connect=hadoop102:2181,hadoop103:2181,hadoop104:2181

#5）配置环境变量

vi /etc/profile

#KAFKA_HOME
export KAFKA_HOME=/opt/module/kafka
export PATH=PATH:KAFKA_HOME/bin

source /etc/profile

#6）分发安装包

 xsync profile

 xsync kafka/

#7）分别在hadoop103和hadoop104上修改配置文件/opt/module/kafka/config/server.properties中的broker.id=1、broker.id=2 注：broker.id不得重复

#8）启动集群

#依次在hadoop102、hadoop103、hadoop104节点上启动kafka

 bin/kafka-server-start.sh config/server.properties &

 bin/kafka-server-start.sh config/server.properties &

 bin/kafka-server-start.sh config/server.properties &

```

### 3.kafka集群操作

```shell
# 1）查看当前服务器中的所有topic
 bin/kafka-topics.sh --list --zookeeper hadoop102:2181
 
# 2）创建topic
 bin/kafka-topics.sh --create --zookeeper hadoop102:2181 --replication-factor 3 --partitions 1 --topic first
 
#选项说明：
--topic 定义topic名
--replication-factor  定义副本数
--partitions  定义分区数

# 3）删除topic
 bin/kafka-topics.sh --delete --zookeeper hadoop102:2181 --topic first
 
#需要server.properties中设置delete.topic.enable=true否则只是标记删除或者直接重启。

# 4）发送消息
 bin/kafka-console-producer.sh --broker-list hadoop102:9092 --topic first
>hello world

# 5）消费消息
 bin/kafka-console-consumer.sh --zookeeper hadoop102:2181 --from-beginning --topic first

# 6）查看某个Topic的详情
 bin/kafka-topics.sh --topic first --describe --zookeeper hadoop102:2181
```

