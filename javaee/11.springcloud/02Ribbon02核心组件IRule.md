### Ribbon核心组件IRule

#### 一：Ribbon核心组件IRule

1、根据特定的算法中从服务列表中选取一个要访问的服务

2、SpringCloud结合Ribbon默认的自带有七种算法

3、可用在GitHub上查询源码https://github.com/Netflix/ribbon

##### 1、RoundRobinRule

1、轮询，依次执行每个执行一次(默认)

##### 2、RandomRule

1、随机
2、在客户端(80)的配置类上(ConfigBean.java)加上新的Bean覆盖默认的轮询

##### 3、AvailabilityFilteringRule

1、会先过滤掉多次访问故障而处于断路器跳闸状态的服务

2、和过滤并发的连接数量超过阀值得服务，然后对剩余的服务列表安装轮询策略进行访问

##### 4、WeightedResponseTimeRule

1、根据平均响应时间计算所有的服务的权重，响应时间越快服务权重越大，容易被选中的概率就越高。

2、刚启动时，如果统计信息不中，则使用RoundRobinRule(轮询)策略，等统计的信息足够了会自动的切换到WeightedResponseTimeRule

##### 5、RetryRule

1、先按照RoundRobinRule(轮询)的策略获取服务，如果获取的服务失败侧在指定的时间会进行重试，进行获取可用的服务

2、如多次获取某个服务失败，这不会再再次获取该服务如(高德地图上某条道路堵车，司机不会走那条道路)

##### 6、BestAvailableRule

1、会先过滤掉由于多次访问故障而处于断路器跳闸状态的服务，然后选择一个并发量最小的服务

##### 7、ZoneAvoidanceRule

1、默认规则，复合判断Server所在区域的性能和Server的可用性选择服务器



### 2.修改默认的轮询规则

```java
//MyRuleConfig 不能与主启动类在一个文件夹下或者是子文件夹 必须单独搞一个文件夹
@Configuration
public class MyRuleConfig {

    @Bean //修改轮询规则为随机
    public IRule iRule(){
        return new RandomRule();//默认轮旋 修改为随机
    }
}

//主启动类上添加注解
//1、加上注解@RibbonClient(name="对外曝光微服务的名称",configuration=自定义的Rlue配置类.class)
@SpringBootApplication
@EnableEurekaClient
@EnableDiscoveryClient
//为服务PROVIDERPRODUCT指定自定义的负载均衡算法
@RibbonClient(name = "PROVIDERPRODUCT",configuration = MyRuleConfig.class)
public class ConsumerApplication {

}    
```

