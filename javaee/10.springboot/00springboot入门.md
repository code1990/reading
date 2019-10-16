### 1SpringBoot入门

#### 1.SpringBoot_入门-课程简介

#### 2.SpringBoot_入门-Spring Boot简介

#### 3.SpringBoot_入门-微服务简介

#### 4.SpringBoot_入门-环境准备

#### 5.SpringBoot_入门-springboot-helloworld

1.引入依赖
```xml
		<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>1.5.8.RELEASE</version>
        </dependency>
```
2.分别开发主启动类，HelloController
```java
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class,args);
    }
}

@Controller
public class HelloController {

    @ResponseBody
    @RequestMapping("/hello")
    public String hello(){
        return "hello world";
    }
}
```
3.测试 浏览器访问
http://localhost:9000/hello

#### 6.SpringBoot_入门-HelloWorld细节-场景启动器（starter）

#### 7.SpringBoot_入门-HelloWorld细节-自动配置

```java
/**
@SpringBootApplication : Spring Boot应用标注在某个类上说明这个类是SpringBoot的主配置类
@EnableAutoConfiguration将主配置类（@SpringBootApplication标注的类）的所在包及下面所有子包里面的所有组件扫描到Spring容器

**/
```
#### 08、SpringBoot_入门-使用向导快速创建Spring Boot应用

