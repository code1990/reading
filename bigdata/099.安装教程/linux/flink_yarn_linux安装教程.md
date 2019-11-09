**Flink on Yarn的两种运行方式**

第一种【yarn-session.sh(开辟资源)+flink run(提交任务)】
启动一个一直运行的flink集群
./bin/yarn-session.sh -n 2 -jm 1024 -tm 1024 [-d]
附着到一个已存在的flink yarn session
./bin/yarn-session.sh -id application_1463870264508_0029
执行任务
./bin/flink run ./examples/batch/WordCount.jar -input hdfs://hadoop100:9000/LICENSE -output hdfs://hadoop100:9000/wordcount-result.txt 
停止任务 【web界面或者命令行执行cancel命令】
第二种【flink run -m yarn-cluster(开辟资源+提交任务)】
启动集群，执行任务
./bin/flink run -m yarn-cluster -yn 2 -yjm 1024 -ytm 1024 ./examples/batch/WordCount.jar
注意：client端必须要设置YARN_CONF_DIR或者HADOOP_CONF_DIR或者HADOOP_HOME环境变量，通过这个环境变量来读取YARN和HDFS的配置信息，否则启动会失败

------------------

**./bin/yarn-session.sh 命令分析**

用法:  
   必选  
     -n,--container <arg>   分配多少个yarn容器 (=taskmanager的数量)  
   可选  
     -D <arg>                        动态属性  
     -d,--detached                   独立运行  
     -jm,--jobManagerMemory <arg>    JobManager的内存 [in MB]  
     -nm,--name                     在YARN上为一个自定义的应用设置一个名字  
     -q,--query                      显示yarn中可用的资源 (内存, cpu核数)  
     -qu,--queue <arg>               指定YARN队列.  
     -s,--slots <arg>                每个TaskManager使用的slots数量  
     -tm,--taskManagerMemory <arg>   每个TaskManager的内存 [in MB]  
     -z,--zookeeperNamespace <arg>   针对HA模式在zookeeper上创建NameSpace 
     -id,--applicationId <yarnAppId>        YARN集群上的任务id，附着到一个后台运行的yarn session中

------------------

**./bin/flink run 命令分析**

run [OPTIONS] <jar-file> <arguments>  
 "run" 操作参数:  
-c,--class <classname>  如果没有在jar包中指定入口类，则需要在这里通过这个参数指定  
-m,--jobmanager <host:port>  指定需要连接的jobmanager(主节点)地址，使用这个参数可以指定一个不同于配置文件中的jobmanager  
-p,--parallelism <parallelism>   指定程序的并行度。可以覆盖配置文件中的默认值。
默认查找当前yarn集群中已有的yarn-session信息中的jobmanager【/tmp/.yarn-properties-root】：
./bin/flink run ./examples/batch/WordCount.jar -input hdfs://hostname:port/hello.txt -output hdfs://hostname:port/result1
连接指定host和port的jobmanager：
./bin/flink run -m hadoop100:1234 ./examples/batch/WordCount.jar -input hdfs://hostname:port/hello.txt -output hdfs://hostname:port/result1
启动一个新的yarn-session：
./bin/flink run -m yarn-cluster -yn 2 ./examples/batch/WordCount.jar -input hdfs://hostname:port/hello.txt -output hdfs://hostname:port/result1
注意：yarn session命令行的选项也可以使用./bin/flink 工具获得。它们都有一个y或者yarn的前缀
例如：./bin/flink run -m yarn-cluster -yn 2 ./examples/batch/WordCount.jar 

----------

**Flink 在yarn上的分布**

Flink on Yarn
ResourceManager
NodeManager
AppMater(jobmanager和它运行在一个Container中)
Container(taskmanager运行在上面)
使用on-yarn的好处
提高集群机器的利用率
一套集群，可以执行MR任务，spark任务，flink任务等...