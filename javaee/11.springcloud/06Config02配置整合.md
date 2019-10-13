### 配置整合

#### 1.新增2个配置文件，提交到github

1.microservicecloud-config-eureka-client.yml(客户端配置)

```yaml
spring: 
  profiles:
   active:
    - dev #生效规则
---
server: 
  port: 7001
spring:
  profiles: dev #生效规则
  application:
    name: microservicecloud-config-eureka-client
eureka:
  instance:
    hostname: eureka7001 #eureka服务端的实例名称
    prefer-ip-address: true     #访问路径可以显示IP地址
  client: 
    register-with-eureka: false     #false表示不向注册中心注册自己。
    fetch-registry: false     #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    service-url: 
      defaultZone: http://eureka7001:7001/eureka/
---
server: 
  port: 7001
spring:
  profiles: test #生效规则
  application:
    name: microservicecloud-config-eureka-client
eureka:
  instance:
    hostname: eureka7001 #eureka服务端的实例名称
    prefer-ip-address: true     #访问路径可以显示IP地址
  client: 
    register-with-eureka: false     #false表示不向注册中心注册自己。
    fetch-registry: false     #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    service-url: 
      defaultZone: http://eureka7001:7001/eureka/
```

 2.microservicecloud-config-dept-client.yml(服务提供者)

```yaml
spring: 
  profiles:
   active:
    - dev #生效规则
---
server:
  port: 8001
 
spring:
   profiles: dev #生效规则
   application:
    name: microservicecloud-config-dept-client
   datasource:
    type: com.alibaba.druid.pool.DruidDataSource            # 当前数据源操作类型
    driver-class-name: org.gjt.mm.mysql.Driver              # mysql驱动包
    url: jdbc:mysql://localhost:3306/cloudDB01              # 数据库名称
    username: root
    password: 671354
    dbcp2:
      min-idle: 5                                           # 数据库连接池的最小维持连接数
      initial-size: 5                                       # 初始化连接数
      max-total: 5                                          # 最大连接数
      max-wait-millis: 200                                  # 等待连接获取的最大超时时间
 
mybatis:
  config-location: classpath:mybatis/mybatis.cfg.xml        # mybatis配置文件所在路径
  type-aliases-package: com.smxy.lq.entities    # 所有Entity别名类所在包
  mapper-locations:
  - classpath:mybatis/mapper/**/*.xml                       # mapper映射文件
 
eureka:
  client: #客户端注册进eureka服务列表内
    service-url:
       defaultZone: http://eureka7001:7001/eureka/
  instance:
    instance-id: microservicecloud-dept8001  #服务名称
    prefer-ip-address: true     #访问路径可以显示IP地址

---
server:
  port: 8001
 
spring:
   profiles: test #生效规则
   application:
    name: microservicecloud-config-dept-client
   datasource:
    type: com.alibaba.druid.pool.DruidDataSource            # 当前数据源操作类型
    driver-class-name: org.gjt.mm.mysql.Driver              # mysql驱动包
    url: jdbc:mysql://localhost:3306/cloudDB02              # 数据库名称
    username: root
    password: 671354
    dbcp2:
      min-idle: 5                                           # 数据库连接池的最小维持连接数
      initial-size: 5                                       # 初始化连接数
      max-total: 5                                          # 最大连接数
      max-wait-millis: 200                                  # 等待连接获取的最大超时时间
 
mybatis:
  config-location: classpath:mybatis/mybatis.cfg.xml        # mybatis配置文件所在路径
  type-aliases-package: com.smxy.lq.entities    # 所有Entity别名类所在包
  mapper-locations:
  - classpath:mybatis/mapper/**/*.xml                       # mapper映射文件
 
eureka:
  client: #客户端注册进eureka服务列表内
    service-url:
       defaultZone: http://eureka7001:7001/eureka/
  instance:
    instance-id: microservicecloud-dept8001  #服务名称
    prefer-ip-address: true     #访问路径可以显示IP地址

```

#### 2.新增注册中心 整合如下的依赖

```xml
<!-- SpringCloudConfig配置 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka-server</artifactId>
        </dependency>

```

#### 3.新增配置文件
1.bootstrap.yml文件


```yaml
spring:
  cloud:
    config:
      name: microservicecloud-config-eureka-client #需要从github上读取的资源名称，注意没有yml后缀名
      profile: dev   #本次访问的配置项
      label: master   
      uri: http://config-3344.com:3344  #本微服务启动后先去找3344号服务，通过SpringCloudConfig获取GitHub的服务地址

```
2.application.yml文件


```yaml
spring:
  application:
    name: microservicecloud-config-eureka-client

```
#### 4.注册中心主启动类

```java
@SpringBootApplication
@EnableEurekaServer
public class Config_Git_EurekaServerApplication
{
	public static void main(String[] args)
	{
		SpringApplication.run(Config_Git_EurekaServerApplication.class, args);
	}
}
```

#### 5.新增服务提供者 整合如下的依赖

```xml
<!-- 将微服务provider侧注册进eureka -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-eureka</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-config</artifactId>
		</dependency>
```

#### 6. 新增配置文件
1.bootstrap.yml文件


```yaml
spring:
  cloud:
    config:
      name: microservicecloud-config-dept-client #需要从github上读取的资源名称，注意没有yml后缀名
      #profile配置是什么就取什么配置dev or test
      profile: dev
      #profile: test
      label: master
      uri: http://config-3344.com:3344  #SpringCloudConfig获取的服务地址

```
2.application.yml文件


```yaml
spring:
  application:
    name: microservicecloud-config-dept-client

```


#### 7.服务提供者主启动类


```java
@SpringBootApplication
@EnableEurekaClient //本服务启动后会自动注册进eureka服务中
@EnableDiscoveryClient //服务发现
public class DeptProvider8001_Config_App
{
	public static void main(String[] args)
	{
		SpringApplication.run(DeptProvider8001_Config_App.class, args);
	}
}

```

#### 8.测试
1.启动配置中心
2.启动注册中心
3.启动服务提供者

**更改bootstrap.yml中的profile的属性就可以实现动态更改策略**