1. HA 集群环境规划

Jobmanager：hadoop100 hadoop101【一个active，一个standby】
Taskmanager：hadoop101 hadoop102
zookeeper: hadoop100【建议使用外置zk 集群，在这里我使用单节点zk 来代替】



注意：
要启用JobManager 高可用性，必须将高可用性模式设置为zookeeper， 配置一个ZooKeeper
quorum，并配置一个masters 文件存储所有JobManager hostname 及其Web UI 端口号。
Flink 利用ZooKeeper 实现运行中的JobManager 节点之间的分布式协调。ZooKeeper 是独立
于Flink 的服务，它通过领导选举制和轻量级状态一致性存储来提供高度可靠的分布式协调。
2. 开始配置+启动
集群内所有节点的配置都一样，所以先从第一台机器hadoop100 开始配置
ssh hadoop100

```shell
#首先按照之前配置standalone 的参数进行修改
vi conf/flink-conf.yaml
jobmanager.rpc.address: hadoop100
vi conf/slaves
hadoop101
hadoop102
# 然后修改配置HA 需要的参数
vi conf/masters

hadoop100:8081
hadoop101:8081
vi conf/flink-conf.yaml
high-availability: zookeeper
high-availability.zookeeper.quorum: hadoop100:2181
# ZooKeeper 节点根目录，其下放置所有集群节点的namespace
high-availability.zookeeper.path.root: /flink
# ZooKeeper 节点集群id，其中放置了集群所需的所有协调数据
high-availability.cluster-id: /cluster_one
# 建议指定hdfs 的全路径。如果某个flink 节点没有配置hdfs 的话，不指定全路径无法识别
# storageDir 存储了恢复一个JobManager 所需的所有元数据。
high-availability.storageDir: hdfs://hadoop100:9000/flink/ha
# 把hadoop100 节点上修改好配置的flink 安装目录拷贝到其他节点
cd /data/soft/
scp -rq flink-1.4.2 hadoop101:/data/soft
scp -rq flink-1.4.2 hadoop102:/data/soft
# 【先启动hadoop 服务】
sbin/start-all.sh
# 【先启动zk 服务】
bin/zkServer.sh start
# 启动flink standalone HA 集群，在hadoop100 节点上启动如下命令
bin/start-cluster.sh

# 启动之后会显示如下日志信息
Starting HA cluster with 2 masters.
Starting standalonesession daemon on host hadoop100.
Starting standalonesession daemon on host hadoop101.
Starting taskexecutor daemon on host hadoop101.
Starting taskexecutor daemon on host hadoop102.
```

3. 验证HA 集群进程
查看机器进程会发现如下情况【此处只列出flink 自身的进程信息，不包含zk，hadoop 进程
信息】

登录hadoop100 节点
执行jps:
20159 StandaloneSessionClusterEntrypoint
登录hadoop101 节点
执行jps:
7795 StandaloneSessionClusterEntrypoint

8156 TaskManagerRunner
登录hadoop102 节点
执行jps:
5046 TaskManagerRunner



因为jobmanager 节点都会启动web 服务，也可以通过web 界面进行验证
访问http://hadoop100:8081/#/jobmanager/config



4. 模拟jobmanager 进程挂掉
现在hadoop100 节点上的jobmanager 是active 的。我们手工把这个进程kill 掉，模拟进程
挂掉的情况，来验证hadoop101 上的standby 状态的jobmanager 是否可以正常切换到active。
ssh hadoop100
执行jps：
20159 StandaloneSessionClusterEntrypoint
kill 20159
5. 验证HA 切换
hadoop100 节点上的jobmanager 进程被手工kill 掉了，然后hadoop101 上的jobmanager 会
自动切换为active，中间需要有一个时间差，稍微等一下
访问http://hadoop101:8081/#/jobmanager/config
如果可以正常访问并且能看到jobmanager 的信息变为hadoop101，则表示jobmanager 节点
切换成功

6. 重启之前kill 掉的jobmanager
进入到hadoop100 机器
ssh hadoop100
执行下面命令启动jobmanager
bin/jobmanager.sh start
启动成功之后，可以访问http://hadoop100:8081/#/jobmanager/config
这个节点重启启动之后，就变为standby 了。hadoop101 还是active。

