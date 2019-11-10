1. HA 集群环境规划
  flink on yarn 的HA 其实是利用yarn 自己的恢复机制。
  在这需要用到zk，主要是因为虽然flink-on-yarn cluster HA 依赖于Yarn 自己的集群机制，但
  是Flink Job 在恢复时，需要依赖检查点产生的快照，而这些快照虽然配置在hdfs，但是其元
  数据信息保存在zookeeper 中，所以我们还要配置zookeeper 的信息
  hadoop 搭建的集群，在hadoop100，hadoop101，hadoop102 节点上面【flink on yarn 使用
  伪分布hadoop 集群和真正分布式hadoop 集群，在操作上没有区别】
  zookeeper 服务也在hadoop100 节点上

2. 开始配置+启动
   主要在hadoop100 这个节点上配置即可

   **首先需要修改hadoop 中yarn-site.xml 中的配置**，设置提交应用程序的最大尝试次数

   ```xml
   <property>
   <name>yarn.resourcemanager.am.max-attempts</name>
   <value>4</value>
   <description>
   The maximum number of application master execution attempts.
   </description>
   </property>
   
   # 把修改后的配置文件同步到hadoop 集群的其他节点
scp -rq etc/hadoop/yarn-site.xml hadoop101:/data/soft/hadoop-2.7.5/etc/hadoop/
scp -rq etc/hadoop/yarn-site.xml hadoop102:/data/soft/hadoop-2.7.5/etc/hadoop/
   ```
**然后修改flink 部分相关配置**

```xml
可以解压一份新的flink-1.6.1 安装包
tar -zxvf flink-1.6.1-bin-hadoop27-scala_2.11.tgz
修改配置文件【标红的目录名称建议和standalone HA 中的配置区分开】
vi conf/flink-conf.yaml
high-availability: zookeeper
high-availability.zookeeper.quorum: hadoop100:2181
high-availability.storageDir: hdfs://hadoop100:9000/flink/ha-yarn
   high-availability.zookeeper.path.root: /flink-yarn
   yarn.application-attempts: 10
   ```
   
   
   
3. **启动flink on yarn，测试HA**
  先启动hadoop100 上的zookeeper 和hadoop

  ```shell
  bin/zkServer.sh start
  sbin/start-all.sh
  ```

  

  在hadoop100 上启动Flink 集群

  ```shell
  cd /data/soft/flink-1.6.1
  bin/yarn-session.sh -n 2
  ```

  到resoucemanager 的web 界面上查看对应的flink 集群在哪个节点上

  jobmanager 进程就在对应的节点的(YarnSessionClusterEntrypoint)进程里面
  所以想要测试jobmanager 的HA 情况，只需要拿YarnSessionClusterEntrypoint 这个进程进行
  测试即可。
  执行下面命令手工模拟kill 掉jobmanager（YarnSessionClusterEntrypoint）、
  ssh hadoop102
  jps
  5325 YarnSessionClusterEntrypoint
  kill 5325
  然后去yarn 的web 界面进行查看：

  如果想查看jobmanager 的webui 界面可以点击下面链接：

4. 注意：针对上面配置文件中的一些配置参数的详细介绍信息可以参考此文章
   https://blog.csdn.net/xu470438000/article/details/79633824
   但是需要注意一点，此链接文章中使用的flink 版本是1.4.2。我们本课程中使用的flink 版本
   是1.6.1，这两个版本中的一些参数名称会有细微不同。但是参数的含义基本没有什么变化。