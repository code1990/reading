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

```java

```
#### 17、SpringBoot_配置-配置文件的加载位置

```java

```
#### 18、SpringBoot_配置-外部配置加载顺序

```java

```
#### 19、SpringBoot_配置-自动配置原理

```java

```
#### 20、SpringBoot_配置-@Conditional&自动配置报告

```java

```