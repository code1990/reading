### 服务熔断

服务熔断

当某个微服务不可用或者响应时间太长时，会进行服务的降级，进而熔断该节点微服务的调用，快速返回"错误"的响应信息。

#### 1.复制provider工程形成provider-hystrix,添加依赖

```xml
   <!--  hystrix -->
   <dependency>
     <groupId>org.springframework.cloud</groupId>
     <artifactId>spring-cloud-starter-hystrix</artifactId>
   </dependency>

```

#### 2.修改yaml文件

```yaml
server:
  port: 8001
  
mybatis:
  config-location: classpath:mybatis/mybatis.cfg.xml  #mybatis所在路径
  type-aliases-package: com..springcloud.entities #entity别名类
  mapper-locations:
  - classpath:mybatis/mapper/**/*.xml #mapper映射文件
    
spring:
   application:
    name: microservicecloud-dept 
   datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name: org.gjt.mm.mysql.Driver
    url: jdbc:mysql://localhost:3306/cloudDB01
    username: root
    password: 123456
    dbcp2:
      min-idle: 5
      initial-size: 5
      max-total: 5
      max-wait-millis: 200
      
eureka:
  client: #客户端注册进eureka服务列表内
    service-url: 
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
  instance:
    instance-id: microservicecloud-dept8001-hystrix   #自定义服务名称信息（文件修改）
    prefer-ip-address: true     #访问路径可以显示IP地址
  

```

#### 3.修改控制层逻辑

```java
@RestController
public class DeptController
{
  @Autowired
  private DeptService service = null;
  
  @RequestMapping(value="/dept/get/{id}",method=RequestMethod.GET)
    @HystrixCommand(fallbackMethod = "processHystrix_Get")
  public Dept get(@PathVariable("id") Long id)
  {
   Dept dept =  this.service.get(id);
   if(null == dept)
   {
     throw new RuntimeException("该ID："+id+"没有没有对应的信息");
   }
   return dept;
  }
  
  public Dept processHystrix_Get(@PathVariable("id") Long id)
  {
   return new Dept().setDeptno(id)
           .setDname("该ID："+id+"没有没有对应的信息,null--@HystrixCommand")
           .setDb_source("no this database in MySQL");
  }
}
```

#### 4.修改主启动类添加支持

```java
@SpringBootApplication
@EnableEurekaClient //本服务启动后会自动注册进eureka服务中
@EnableCircuitBreaker//对hystrixR熔断机制的支持
public class DeptProvider8001_Hystrix_App
{
  public static void main(String[] args)
  {
   SpringApplication.run(DeptProvider8001_Hystrix_App.class, args);
  }
}

```

#### 5.测试

1.启动eurekaserver注册中心

2.启动provider-hystrix

3.启动comsuer

4.http://localhost/consumer/dept/get/112

