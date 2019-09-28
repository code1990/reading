## Zookeeper-Linux安装教程(root用户)

#### 0）集群规划

在hadoop102、hadoop103和hadoop104三个节点上部署Zookeeper。

#### 1）解压安装

```shell
##（1）解压zookeeper安装包到/opt/module/目录下

 tar -zxvf zookeeper-3.4.10.tar.gz -C /opt/module/

##（2）在/opt/module/zookeeper-3.4.10/这个目录下创建zkData

mkdir -p zkData

##（3）重命名/opt/module/zookeeper-3.4.10/conf这个目录下的zoo_sample.cfg为zoo.cfg

mv zoo_sample.cfg zoo.cfg
```

#### 2）配置zoo.cfg文件

```shell
##（1）具体配置

dataDir=/opt/module/zookeeper-3.4.10/zkData

##增加如下配置

#######################cluster##########################

server.2=hadoop102:2888:3888

server.3=hadoop103:2888:3888

server.4=hadoop104:2888:3888

##（2）配置参数解读

##Server.A=B:C:D。

##A是表示这个是第几号服务器；

##B是这个服务器的ip地址；

##C是这个服务器与集群中的Leader服务器交换信息的端口；

##D是这个端口就是用来执行选举时服务器相互通信的端口。

```

#### 3）集群操作

```shell
##（1）在/opt/module/zookeeper-3.4.10/zkData目录下创建一个myid的文件

touch myid

##添加myid文件，注意一定要在linux里面创建，在notepad++里面很可能乱码

##（2）编辑myid文件

vi myid

##在文件中添加与server对应的编号：如2

##（3）拷贝配置好的zookeeper到其他机器上

scp -r zookeeper-3.4.10/ [root@hadoop103:/opt/app/]

scp -r zookeeper-3.4.10/ [root@hadoop104:/opt/app/]

## 并分别修改myid文件中内容为3、4

##（4）分别启动zookeeper

[root@hadoop102 zookeeper-3.4.10]# bin/zkServer.sh start

[root@hadoop103 zookeeper-3.4.10]# bin/zkServer.sh start

[root@hadoop104 zookeeper-3.4.10]# bin/zkServer.sh start

##（5）查看状态

[root@hadoop102 zookeeper-3.4.10]# bin/zkServer.sh status

[root@hadoop103 zookeeper-3.4.10]# bin/zkServer.sh status

[root@hadoop104 zookeeper-3.4.5]# bin/zkServer.sh status

```

