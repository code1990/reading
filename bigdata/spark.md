## 第1 章 Spark 概述

### 1.1 什么是spark

spark是一种基于内存的大数据分析引擎

### 1.2 Spark 内置模块

- spark core实现了spark的基本功能
- spark sql实现数据查询功能
- spark streaming实现了数据的流式计算
- spark MLlib实现了机器学习常用的库'
- spark graphX 图计算

### 1.3 Spark 特点

- 快 基于内存
- 易用 提供了java,python,scalaAPI
- 通用 针对多个场景提供了统一的解决方案
- 兼容性 兼容其他的大数据框架

## 第2章 Spark 运行模式

### 2.1 Spark 安装地址

### 2.3Local 模式

Local模式就是运行在一台计算机上的模式，通常就是用于在本机上练手和测试

### 2.4 Standalone模式

构建一个由Master+Slave 构成的Spark 集群，Spark 运行在集群中。

### 2.5 Yarn 模式

### 2.6 Mesos模式

