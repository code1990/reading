### 1Flink基础入门

#### 01.Flink基本原理介绍		
```java
Apache Flink 是一个开源的分布式，高性能，高可用，准确的流处理框架。
主要由 Java 代码实现。
支持实时流(stream)处理和批(batch)处理，批数据只是流数据的一个极限特例。
Flink原生支持了迭代计算、内存管理和程序优化。

```
#### 02.Flink与Storm与SparkStream的关系		
```java
1：需要关注流数据是否需要进行状态管理
2：At-least-once或者Exectly-once消息投递模式是否有特殊要求
3：对于小型独立的项目，并且需要低延迟的场景，建议使用storm
4：如果你的项目已经使用了spark，并且秒级别的实时处理可以满足需求的话，建议使用sparkStreaming
5：要求消息投递语义为 Exactly Once 的场景；数据量较大，要求高吞吐低延迟的场景；需要进行状态管理或窗口统计的场景，建议使用flink

```
#### 03.Flink案例需求分析		
```xml
<dependency>  
  <groupId>org.apache.flink</groupId>  
  <artifactId>flink-java</artifactId>  
  <version>1.6.1</version>   
</dependency>  
<dependency>  
  <groupId>org.apache.flink</groupId>  
  <artifactId>flink-streaming-java_2.11</artifactId>  
  <version>1.6.1</version>  
  <scope>provided</scope>  
</dependency> 

<dependency>  
  <groupId>org.apache.flink</groupId>  
  <artifactId>flink-scala_2.11</artifactId>  
  <version>1.6.1</version>  
</dependency>  
<dependency>  
  <groupId>org.apache.flink</groupId>  
  <artifactId>flink-streaming-scala_2.11</artifactId>  
  <version>1.6.1</version>  
</dependency> 
```
#### 04.滑动窗口单词计数-java代码实现		
```java

```
#### 05.滑动窗口单词计数-scala代码实现		
```java

```
#### 06.batch批处理之java代码实现		
```java

```
#### 07.batch批处理之scala代码实现		
```java

```
#### 08.Flink streaming和Batch代码层面的使用		
```java

```
#### 09.Flink standalone集群安装部署		
```java

```
#### 10.Flink和yarn的两种实现方式		
```java

```
#### 11.Flink 和yarn内部实现		
```java

```
#### 12.Flink standalone集群HA配置		
```java

```
#### 13.如何解决集群启动失败的问题		
```java

```
#### 14.Flink和yarn集群HA配置		
```java

```
#### 15.Flink scala shell代码调试		
```java

```
