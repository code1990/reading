### 1springboot配置文件

#### 09.SpringBoot_配置-yaml简介

```java
/**
SpringBoot使用一个全局的配置文件，配置文件名是固定的；

application.properties
application.yml

YAML：以数据为中心，比json、xml等更适合做配置文件；


**/

```

#### 10.SpringBoot_配置-yaml语法
1、基本语法
k:(空格)v：表示一对键值对（空格必须有）；

以空格的缩进来控制层级关系；只要是左对齐的一列数据，都是同一个层级的

属性和值也是大小写敏感；
```yaml
person: 
    name: jim #字符串默认不需要加引号
    name1: "Jim\nGreen" #""会添加转义特殊字符
    name2: 'Jim\nGrren' #''不会转义特殊字符
        
friends: {lastName: zhangsan, age: 18} # Map行内写法
pets: [cat, dog, pig] #list set行内写法
```

#### 11.SpringBoot_配置-yaml配置文件值获取

```xml
<!--导入配置文件处理器，配置文件进行绑定就会有提示-->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-configuration-processor</artifactId>
			<optional>true</optional>
		</dependency>


```



```java
person:
    lastName: hello
    age: 18
    boss: false
    birth: 2017/12/12
    maps: {k1: v1, k2: 12}
    lists:
      - lisi
      - zhaoliu
    dog:
      name: 小狗
      age: 12
```
```properties
# idea的properties文件默认U8 需要转ascii码
person.last-name=\u674E\u56DB 
person.age=12
person.birth=2017/12/15
person.boss=false
person.maps.k1=v1
person.maps.k2=14
person.lists=a,b,c
person.dog.name=dog
person.dog.age=15
```

```java

/**
 * 将配置文件中配置的每一个属性的值，映射到这个组件中
 * @ConfigurationProperties：告诉SpringBoot将本类中的所有属性和配置文件中相关的配置进行绑定；
 *      prefix = "person"：配置文件中哪个下面的所有属性进行一一映射
 *
 * 只有这个组件是容器中的组件，才能容器提供的@ConfigurationProperties功能；
 *  @ConfigurationProperties(prefix = "person")默认从全局配置文件中获取值；
 *
 */
//@PropertySource(value = {"classpath:person.properties"})
@Component
@ConfigurationProperties(prefix = "person")
//@Validated
public class Person {

    /**
     * <bean class="Person">
     *      <property name="lastName" value="字面量/${key}从环境变量、配置文件中获取值/#{SpEL}"></property>
     * <bean/>
     */

   //lastName必须是邮箱格式
   // @Email
    //@Value("${person.last-name}")
    private String lastName;
    //@Value("#{11*2}")
    private Integer age;
    //@Value("true")
    private Boolean boss;

    private Date birth;
    //@Value("${person.maps}")
    private Map<String,Object> maps;
    private List<Object> lists;
    private Dog dog;
}
```



```java

@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringBoot02ConfigApplicationTests {

	@Autowired
	Person person;

	@Autowired
	ApplicationContext ioc;

	@Test
	public void testHelloService(){
		boolean b = ioc.containsBean("helloService02");
		System.out.println(b);
	}


	@Test
	public void contextLoads() {
		System.out.println(person);
	}

}
```



#### 12.SpringBoot_配置-properties配置文件编码问题

#### 13.SpringBoot_配置-@ConfigurationProperties与@Value区别



|                      | @ConfigurationProperties | @Value     |
| -------------------- | ------------------------ | ---------- |
| 功能                 | 批量注入配置文件中的属性 | 一个个指定 |
| 松散绑定（松散语法） | 支持                     | 不支持     |
| SpEL                 | 不支持                   | 支持       |
| JSR303数据校验       | 支持                     | 不支持     |
| 复杂类型封装         | 支持                     | 不支持     |

松散语法绑定：last_name = last-name = lastName 他们取的值都是相同的

配置文件yml还是properties他们都能获取到值；

如果说，我们只是在某个业务逻辑中需要获取一下配置文件中的某项值，使用@Value；

如果说，我们专门编写了一个javaBean来和配置文件进行映射，我们就直接使用@ConfigurationProperties；

#### 14.SpringBoot_配置-@PropertySource、@ImportResource、@Bean



@**PropertySource**：加载指定的配置文件；

@PropertySource(value = {"classpath:person.properties"})

@**ImportResource**：导入Spring的配置文件，让配置文件里面的内容生效；

@ImportResource(locations = {"classpath:beans.xml"})

```java
/**SpringBoot推荐给容器中添加组件的方式；推荐使用全注解的方式

1、配置类**@Configuration** --> Spring配置文件

2、使用**@Bean**给容器中添加组件**/

@Configuration
public class MyAppConfig {

    // 将方法的返回值添加到容器中；容器中这个组件默认的id就是方法名
    @Bean
    public HelloService helloService02(){
        System.out.println("配置类@Bean给容器中添加组件了...");
        return new HelloService();
    }
}
```

#### 15.SpringBoot_配置-配置文件占位符

```properties
#随机数
${random.value}、${random.int}、${random.long}
${random.int(10)}、${random.int[1024,65536]}
#占位符获取之前配置的值
person.last-name=张三${random.uuid}
person.age=${random.int}
person.birth=2017/12/15
person.boss=false
person.maps.k1=v1
person.maps.k2=14
person.lists=a,b,c
# 没有取到:后面是默认值
person.dog.name=${person.hello:hello}_dog
person.dog.age=15
```
#### 16、SpringBoot_配置-Profile多环境支持

##### 1、多Profile文件

我们在主配置文件编写的时候，文件名可以是 application-{profile}.properties/yml

默认使用application.properties的配置；

##### 2、yml支持多文档块方式

```yaml
# 文档块一
---
# 文档块二
---
# 文档块三
```

##### 3、激活指定profile

 1、在配置文件中指定 spring.profiles.active=dev

 2、命令行：

 java -jar spring-boot-02-config-0.0.1-SNAPSHOT.jar --spring.profiles.active=dev；

 可以直接在测试的时候，配置传入命令行参数

 3、虚拟机参数；

 -Dspring.profiles.active=dev

#### 17、SpringBoot_配置-配置文件的加载位置

springboot 启动会扫描以下位置的application.properties或者application.yml文件作为Spring boot的默认配置文件

–file:./config/ 项目目录下的config

–file:./ 项目目录下

–classpath:/config/ resources目录下的config

–classpath:/ resources目录下

优先级由高到底，高优先级的配置会覆盖低优先级的配置；

SpringBoot会从这四个位置全部加载主配置文件；**互补配置**；

#### 18、SpringBoot_配置-外部配置加载顺序

**SpringBoot也可以从以下位置加载配置； 优先级从高到低；高优先级的配置覆盖低优先级的配置，所有的配置会形成互补配置**

1.命令行参数

所有的配置都可以在命令行上进行指定

java -jar spring-boot-02-config-02-0.0.1-SNAPSHOT.jar --server.port=8087 --server.context-path=/abc

多个配置用空格分开； --配置项=值

由jar包外向jar包内进行寻找；

优先加载带profile

再来加载不带profile

#### 19、SpringBoot_配置-自动配置原理

### 1、**自动配置原理：**

1）、SpringBoot启动的时候加载主配置类，开启了自动配置功能 @EnableAutoConfiguration

**2）、@EnableAutoConfiguration 作用：**

- 利用EnableAutoConfigurationImportSelector给容器中导入一些组件？

- 可以查看selectImports()方法的内容；

- `List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);`获取候选的配置

  - ```java
    SpringFactoriesLoader.loadFactoryNames()
    扫描所有jar包类路径下  META-INF/spring.factories
    把扫描到的这些文件的内容包装成properties对象
    从properties中获取到EnableAutoConfiguration.class类（类名）对应的值，然后把他们添加在容器中
    
    ```

**将类路径下 META-INF/spring.factories 里面配置的所有EnableAutoConfiguration的值加入到了容器中**

每一个这样的 xxxAutoConfiguration类都是容器中的一个组件，都加入到容器中；用他们来做自动配置；

3）、每一个自动配置类进行自动配置功能；

4）、以**HttpEncodingAutoConfiguration（Http编码自动配置）**为例解释自动配置原理；()

根据当前不同的条件判断，决定这个配置类是否生效？

一但这个配置类生效；这个配置类就会给容器中添加各种组件；这些组件的属性是从对应的properties类中获取的，这些类里面的每一个属性又是和配置文件绑定的；

5）、所有在配置文件中能配置的属性都是在xxxxProperties类中封装者‘；配置文件能配置什么就可以参照某个功能对应的这个属性类

**精髓：**

 **1）、SpringBoot启动会加载大量的自动配置类**

 **2）、我们看我们需要的功能有没有SpringBoot默认写好的自动配置类；**

 **3）、我们再来看这个自动配置类中到底配置了哪些组件；（只要我们要用的组件有，我们就不需要再来配置了）**

 **4）、给容器中自动配置类添加组件的时候，会从properties类中获取某些属性。我们就可以在配置文件中指定这些属性的值；**

xxxxAutoConfigurartion：自动配置类；

给容器中添加组件

xxxxProperties:封装配置文件中相关属性；(HttpEncodingProperties )

#### 20、SpringBoot_配置-@Conditional&自动配置报告

#### 1、@Conditional派生注解（Spring注解版原生的@Conditional作用）

作用：必须是@Conditional指定的条件成立，才给容器中添加组件，配置配里面的所有内容才生效；

| @Conditional扩展注解            | 作用（判断是否满足当前指定条件）                 |
| ------------------------------- | ------------------------------------------------ |
| @ConditionalOnJava              | 系统的java版本是否符合要求                       |
| @ConditionalOnBean              | 容器中存在指定Bean；                             |
| @ConditionalOnMissingBean       | 容器中不存在指定Bean；                           |
| @ConditionalOnExpression        | 满足SpEL表达式指定                               |
| @ConditionalOnClass             | 系统中有指定的类                                 |
| @ConditionalOnMissingClass      | 系统中没有指定的类                               |
| @ConditionalOnSingleCandidate   | 容器中只有一个指定的Bean，或者这个Bean是首选Bean |
| @ConditionalOnProperty          | 系统中指定的属性是否有指定的值                   |
| @ConditionalOnResource          | 类路径下是否存在指定资源文件                     |
| @ConditionalOnWebApplication    | 当前是web环境                                    |
| @ConditionalOnNotWebApplication | 当前不是web环境                                  |
| @ConditionalOnJndi              | JNDI存在指定项                                   |

**自动配置类必须在一定的条件下才能生效；**

我们怎么知道哪些自动配置类生效；

**我们可以通过启用 debug=true属性；来让控制台打印自动配置报告**，这样我们就可以很方便的知道哪些自动配置类生效；

