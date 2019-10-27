### 1
#### 01课程介绍

#### 02项目价值说明

#### 03项目架构讲解

#### 04数据来源说明

#### 05静态信息和动态信息说明

#### 06用户画像之还原真实场景表结构定义讲解

创建表

user_info信息表
user_detail表

用户表：用户ID、用户名、密码、性别、手机号、邮箱、年龄、注册时间、收货地址、终端类型

用户详情补充表：学历、收入、职业、婚姻、是否有小孩、是否有车有房、使用手机品牌、用户id


#### 07用户画像之flink画像分析模块项目构建

1.创建一个父类的工程flinkuser

2.创建一个module工程

3.项目加入如下的依赖

```xml
<modelVersion>4.0.0</modelVersion>
    <artifactId>service</artifactId>
    <parent>
        <groupId>org.apache.flink</groupId>
        <artifactId>flink-examples</artifactId>
        <version>1.7.0</version>
    </parent>

    <properties>
        <scala.version>2.11.12</scala.version>
        <scala.binary.version>2.11</scala.binary.version>
    </properties>


    <dependencies>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-streaming-java_${scala.binary.version}</artifactId>
            <version>${project.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-scala_2.11</artifactId>
            <version>${project.version}</version>
        </dependency>
    </dependencies>
```
4.编写一个类

```java
public class Test {
    public static void main(String[] args) {
        //把main方法的参数设置为params
        final ParameterTool params = ParameterTool.fromArgs(args);
        //设置启动环境
        final ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        //设置参数为全局参数
        env.getConfig().setGlobalJobParameters(params);
        //文件输入
        DataSet<String> text = env.readTextFile(params.get("input"));
        //
        DataSet map = text.flatMap(null);
        DataSet reduce = map.groupBy("groupbyfield").reduce(null);
        try {
            env.execute("test");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

#### 08用户画像之hadoop环境搭建

```java

```
#### 09用户画像之hbase环境搭建
```java

```
#### 10用户画像之mongo环境搭建
```java

```
#### 11用户画像之年代标签代码编写1
```java

```
#### 12用户画像之flink结合hbase保存年代标签代码编写
```java

```
#### 13用户画像之年代群体数量统计代码编写1
```java

```
#### 14用户画像之flink结合mongo保存年代群体数量
```java

```
#### 15用户画像之手机运营商标签代码编写1
```java

```
#### 16用户画像之手机运营商标签代码编写2
```java

```
#### 17用户画像之邮件运营商标签代码编写1
```java

```
#### 18用户画像之邮件运营商标签代码编写2
```java

```
#### 19用户画像之还原真实消费信息表结构定义
```java

```
#### 20用户画像之败家指数计算规则定义
```java

```
#### 21用户画像之败家指数代码编写1
```java

```
#### 22用户画像之败家指数代码编写2
```java

```
#### 23用户画像之败家指数代码编写3
```java

```
#### 24用户画像之败家指数代码编写4
```java

```
#### 25用户画像之败家指数代码编写5
```java

```
#### 26用户画像之败家指数之最终得分计算代码编写
```java

```
#### 27用户画像之败家指数之最终得分保存代码编写
```java

```
#### 28用户画像之用户行为日志结构讲解以及实体定义
```java

```
#### 29基于springboot+springcloud之2.0版本构建实时数据收集服务之注册中心代码编写1
```java

```
#### 30基于springboot+springcloud之2.0版本构建实时数据收集服务之注册中心补充
```java

```
#### 31基于springboot+springcloud之2.0版本构建实时数据收集服务之服务搭建代码编写
```java

```
#### 32用户画像之基于springboot+springcloud之2.0版本构建实时数据收集服务代码编写
```java

```
#### 33用户画像之kafka环境搭建
```java

```
#### 34用户画像之实时收集服务整合kafka代码编写1
```java

```
#### 35用户画像之实时收集服务整合kafka代码编写2
```java

```
#### 36用户画像之实时品牌偏好设计以及代码编写实现实时更新用户品牌偏好
```java

```
#### 37用户画像之实时品牌偏好代码编写2
```java

```
#### 38用户画像之实时品牌偏好代码编写3
```java

```
#### 39-41用户画像之实时终端偏好代码编写123
```java

```
#### 42用户画像之flume环境搭建
```java

```
#### 43用户画像之梯度下降法大白话讲解
```java

```
#### 44用户画像之结合数据微分以及数学公式讲解梯度下降法
```java

```
#### 45用户画像之java实现逻辑回归算法
```java

```
#### 46用户画像之flink实现分布式逻辑回归算法代码编写1
```java

```
#### 47用户画像之flink实现分布式逻辑回归算法代码编写2
```java

```
#### 48用户画像之flink逻辑回归预测性别代码编写1
```java

```
#### 49用户画像之flink逻辑回归预测性别代码编写2
```java

```
#### 50用户画像之flink逻辑回归预测性别代码编写3
```java

```
#### 51用户画像之kmeans之原理讲解
```java

```
#### 52用户画像之java实现kmeans代码编写
```java

```
#### 53用户画像之flink实现分布式kmeans代码编写
```java

```
#### 54用户画像之flink实现分布式kmeans代码编写2
```java

```
#### 55用户画像之flink实现分布式kmeans代码编写3
```java

```
#### 56用户画像之flink实现分布式kmeans代码编写4
```java

```
#### 57用户画像之flink分布式kmeans实现用户分群代码编写1
```java

```
#### 58用户画像之flink分布式kmeans实现用户分群代码编写2
```java

```
#### 59用户画像之flink分布式kmeans实现用户分群代码编写3
```java

```
#### 60用户画像之flink分布式kmeans实现用户分群代码编写4
```java

```
#### 61用户画像之flink分布式kmeans实现用户分群代码编写5
```java

```
#### 62用户画像之潮男族潮女族标签代码编写1
```java

```
#### 63用户画像之潮男组潮女族标签代码编写2
```java

```
#### 64用户画像之潮男族潮女族标签代码编写3
```java

```
#### 65用户画像之潮男族潮女族标签代码编写4
```java

```
#### 66用户画像之消费水平标签代码编写1
```java

```
#### 67用户画像之消费水平标签代码编写2
```java

```
#### 68用户画像之消费水平标签代码编写3
```java

```
#### 69用户画像之vuejs+nodejs构建前端项目讲解
```java

```
#### 70用户画像之vuejs+highcharts构建图表代码编写
```java

```
#### 71用户画像之vuejs+hightcharts构建图表效果演示
```java

```
#### 72用户画像之接口查询服务构建
```java

```
#### 73用户画像之年代接口代码编写
```java

```
#### 74用户画像之前端查询服务构建
```java

```
#### 75用户画像之基于springcloud+Feign服务调用代码编写
```java

```
#### 76用户画像之基于springcloud+Feign服务调用代码编写2
```java

```
#### 77用户画像之vuejs整合前端查询接口代码编写
```java

```
#### 78用户画像之vuejs整合前段查询接口之跨域问题解决
```java

```
#### 79用户画像之前端查询接口进一步封装代码编写
```java

```
#### 80用户画像之接口重构代码编写
```java

```
#### 81用户画像之前端查询接口重用改造代码编写
```java

```
#### 82用户画像vuejs完善剩余图表代码编写1
```java

```
#### 83用户画像之vuejs完善剩余图表代码编写2
```java

```
#### 84用户画像之vuejs完善剩余图表代码编写3
```java

```
#### 85用户画像之vuejs配置路由代码编写
```java

```
#### 86用户画像之接口服务前端查询服务以及前端展示服务联调以及效果展示.zip
```java

```
#### 87用户画像之TF-IDF通俗讲解
```java

```
#### 88用户画像之分词工具ik讲解以及代码编写.zip
```java

```
#### 89用户画像之java实现TF-IDF代码编写1
```java

```
#### 90用户画像之java实现TF-IDF代码编写2
```java

```
#### 91用户画像之flink实现分布式TF-IDF代码编写1
```java

```
#### 92用户画像之flink实现分布式TF-IDF代码编写2、
```java

```
#### 93用户画像之fink分布式TF-IDF实现用户年度、月度，季度商品关键词代码编写1
```java

```
#### 94用户画像之fink分布式TF-IDF实现用户年度、月度，季度商品关键词代码编写2
```java

```
#### 95用户画像之fink分布式TF-IDF实现用户年度、月度，季度商品关键词代码编写3
```java

```
#### 96用户画像之fink分布式TF-IDF实现用户年度、月度，季度商品关键词代码编写4
```java

```
#### 97用户画像之标签接口之败家指数接口代码编写
```java

```
#### 98用户画像之全部标签接口代码编写
```java

```
#### 99用户画像之前端标签查询服务代码编写
```java

```
#### 100用户画像之vue.js标签显示代码编写1
```java

```
#### 101用户画像之vue.js标签显示代码编写2以及效果演示
```java

```
