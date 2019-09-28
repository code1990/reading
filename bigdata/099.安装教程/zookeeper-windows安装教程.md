## zookeeper windows安装教程

2.1 下载安装文件： http://zookeeper.apache.org/releases.html

2.2 解压文件（本文解压到 G:\zookeeper-3.4.8）

2.3 打开G:\zookeeper-3.4.8\conf，把zoo_sample.cfg重命名成zoo.cfg

2.4 从文本编辑器里打开zoo.cfg

2.5 把dataDir的值改成“:\zookeeper-3.4.8\data”

2.6 添加如下系统变量：

```
1)ZOOKEEPER_HOME: G:\zookeeper-3.4.8

2)Path: 在现有的值后面添加";%ZOOKEEPER_HOME%\bin;"
```

2.7 运行Zookeeper: 打开cmd然后执行zkserver.cmd

