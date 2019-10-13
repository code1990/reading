### 配置类客户端

#### 1.新建microservicecloud-config-client.yml提交到github

```yaml
spring: 
  profiles:
   active:
    - dev
---
server:
  port: 8201
spring:
  profiles: dev
  application:
    name: microservicec loud-config-client
eureka:
  client:
    serviceUrl:
       defaultZone: http://eureka7001:7001/eureka/
---
server:
  port: 8202
spring:
  profiles: test
  application:
    name: microservicec loud-config-client
eureka:
  client:
    serviceUrl:
       defaultZone: http://eureka7001:7001/eureka/


```

#### 2.新建config-client工程 整合如下的依赖

```xml
<!-- SpringCloud Config客户端 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
```

#### 3.新増一个bootstrap.yml文件

bootstrap.yml 可以理解成系统级别的一些参数配置，这些参数一般是不会变动的。
application.yml 可以用来定义应用级别的

```yaml
spring:
  cloud:
    config:
      name: microservicecloud-config-client #需要从github上读取的资源名称，注意没有yml后缀名
      profile: test   #本次访问的配置项
      label: master   
      uri: http://config-3344.com:3344  #本微服务启动后先去找3344号服务，通过SpringCloudConfig获取GitHub的服务地址
```

#### 4.配置application.yml

```java
spring:
  application:
    name: microservicecloud-config-client
```

#### 5.获取文件信息的controller

```java
@RestController
public class ConfigClientRest
{

	@Value("${spring.application.name}")
	private String applicationName;

	@Value("${eureka.client.serviceUrl.defaultZone}")
	private String eurekaServers;

	@Value("${server.port}")
	private String port;

	@RequestMapping("/config")
	public String getConfig()
	{
		String str = "applicationName: " + applicationName + "\t eurekaServers:" + eurekaServers + "\t port: " + port;
		System.out.println("******str: " + str);
		return "applicationName: " + applicationName + "\t eurekaServers:" + eurekaServers + "\t port: " + port;
	}
}

```

#### 6.主启动类

```java
@SpringBootApplication
public class ConfigClient_3355_StartSpringCloudApp
{
	public static void main(String[] args)
	{
		SpringApplication.run(ConfigClient_3355_StartSpringCloudApp.class, args);
	}
}
```

#### 7.测试

1.启动注册中心

2.启动配置中心

3.启动配置客户端

http://client-config.com:8202/config 

bootstrap.yml配置的是test ,所有端口信息是8082
