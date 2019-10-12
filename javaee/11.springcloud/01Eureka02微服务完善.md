### 主机映射名称修改

```yaml
eureka:
  client: #客户端注册进eureka服务列表内
    service-url: 
      defaultZone: http://localhost:7001/eureka
  instance: 
    instance-id: microservicecloud-dept8001 #主机映射名称修改
```



### 主机IP信息提示

```yaml
eureka:
  client: #客户端注册进eureka服务列表内
    service-url: 
      defaultZone: http://localhost:7001/eureka
  instance:
    instance-id: microservicecloud-dept8001   #自定义服务名称信息
    prefer-ip-address: true     #访问路径可以显示IP地址

```

### info内容构建

#### 1.provider添加pom.xml文件

```xml
  <!-- actuator监控信息完善 -->
   <dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-actuator</artifactId>
   </dependency>  
```

#### 2.eurekaserver工程添加编译插件

```xml
<build>
   <finalName>microservicecloud</finalName>
   <resources>
     <resource>
       <directory>src/main/resources</directory>
       <filtering>true</filtering>
     </resource>
   </resources>
   <plugins>
     <plugin>
       <groupId>org.apache.maven.plugins</groupId>
       <artifactId>maven-resources-plugin</artifactId>
       <configuration>
         <delimiters>
          <delimit>$</delimit>
         </delimiters>
       </configuration>
     </plugin>
   </plugins>
  </build>

```

#### 3.eurekaserver工程添加编译插件

```yaml
server:
  port: 8001
  
mybatis:
  config-location: classpath:mybatis/mybatis.cfg.xml  #mybatis所在路径
  type-aliases-package: com.springcloud.entities #entity别名类
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
      defaultZone: http://localhost:7001/eureka
  instance:
    instance-id: microservicecloud-dept8001   #自定义服务名称信息
    prefer-ip-address: true     #访问路径可以显示IP地址
# 配置服务info细节 ========================     
info:
  app.name: microservicecloud
  company.name: www.baidu.com
  build.artifactId: $project.artifactId$
  build.version: $project.version$
      
      
      


```

### 自我保护机制

什么是自我保护模式？

自我保护模式是一种应对网络异常的安全保护措施。它的架构哲学是宁可同时保留所有微服务，也不盲目注销任何健康的微服务。使用自我保护模式，可以让Eureka集群更加的健壮、稳定。

```properties
eureka.server.enable-self-preservation = false #禁用自我保护模式。
```

