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

