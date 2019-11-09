**Flink集群安装部署**

1：standalone模式

集群节点规划(一主两从)
hadoop100--master(JobManager)
hadoop101--slave(TaskManager)
hadoop102--slave(TaskManager)

基础环境：
	jdk1.8及以上【需要配置JAVA_HOME】
	ssh免密码登录(至少要实现主节点能够免密登录到从节点)
	主机名hostname
	/etc/hosts文件配置主机名和ip的映射关系
	关闭防火墙
	

在hadoop100节点上主要需要修改的配置信息
cd /data/soft/flink-1.6.1/conf
vi flink-conf.yaml
jobmanager.rpc.address: hadoop100


vi slaves
hadoop101
hadoop102

然后再把修改好的flink目录拷贝到其他两个节点即可
scp -rq flink-1.6.1 hadoop101:/data/soft/
scp -rq flink-1.6.1 hadoop102:/data/soft/

------------------

**Flink-Standalone集群重要参数详解**



jobmanager.heap.mb：jobmanager节点可用的内存大小
taskmanager.heap.mb：taskmanager节点可用的内存大小
taskmanager.numberOfTaskSlots：每台机器可用的cpu数量
parallelism.default：默认情况下任务的并行度
taskmanager.tmp.dirs：taskmanager的临时数据存储目录

slot和parallelism总结
1.slot是静态的概念，是指taskmanager具有的并发执行能力
2.parallelism是动态的概念，是指程序运行时实际使用的并发能力
3.设置合适的parallelism能提高运算效率，太多了和太少了都不行



-----------------------

**集群节点重启及扩容**

启动jobmanager
如果集群中的jobmanager进程挂了，执行下面命令启动。
bin/jobmanager.sh start
bin/jobmanager.sh stop
启动taskmanager
添加新的taskmanager节点或者重启taskmanager节点
bin/taskmanager.sh start
bin/taskmanager.sh stop

**Flink standalone集群中job的容错**

jobmanager挂掉
正在执行的任务会失败
存在单点故障，(Flink支持HA，后面会讲到)
taskmanager挂掉
如果有多余的taskmanager节点，flink会自动把任务调度到其它节点执行