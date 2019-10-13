### 配置中心

#### 1.github创建一个项目 提交application.yml文件

```yaml
spring: 
  profiles:
   active:
    - dev
---
spring: 
  profiles: dev #开发环境
  application:
    name: microservicecloud-config-dev
---
spring: 
  profiles: test  #测试环境
  application:
    name: microservicecloud-config-test

```

#### 2.创建一个config-server工程 整合如下的依赖

```xml
<!-- springCloud Config -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-config-server</artifactId>
		</dependency>
```

#### 3.修改yml文件

```java
server:
  port: 3344
 
spring:
  application:
    name:  microservicecloud-config
  cloud:
    config:
      server:
        git:
           uri: https://github.com/yjp245/microservicecloud-config.git  #git仓库

```

#### 4.主启动类添加注解

```java
@SpringBootApplication
@EnableConfigServer		#config服务端
public class Config_3344_StartSpringCloudApp
{
	public static void main(String[] args)
	{
		SpringApplication.run(Config_3344_StartSpringCloudApp.class, args);
	}
}
```

#### 5.修改hosts文件

```java
127.0.0.1 config-3344.com
```

#### 6.测试

http://config-3344.com:3344/master/application-dev.yml

http://config-3344.com:3344/master/application-test.yml
