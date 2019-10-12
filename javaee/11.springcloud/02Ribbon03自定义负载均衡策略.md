### Ribbon自定义负载均衡策略

#### 1.继承AbstractLoadBalancerRule自定义负载均衡算法

```java
public class MyRandomRule extends AbstractLoadBalancerRule {


    Random rand;

    public MyRandomRule() {
        rand = new Random();
    }

    /**
     * Randomly choose from all living servers
     * 如果需要执行自定义的负载均衡策略 则需要重写如下的方法 订制化自己的需求
     */
   
    public Server choose(ILoadBalancer lb, Object key) {
    //重写该方法
    }

    @Override
    public Server choose(Object key) {
        return choose(getLoadBalancer(), key);
    }

    @Override
    public void initWithNiwsConfig(IClientConfig clientConfig) {
        

    }
}
```

#### 2.配置自定义的轮询算法

```java
@Configuration
public class MyRuleConfig {

    @Bean //修改轮询规则为随机
    public IRule iRule(){
        return new MyRandomRule();//默认轮旋 修改为随机
    }
}
```

#### 3.配置主启动类

```java
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

