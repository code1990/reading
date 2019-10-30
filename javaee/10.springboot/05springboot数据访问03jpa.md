#### 66、SpringBoot_数据访问-SpringData JPA简介

JPA:ORM（Object Relational Mapping）；

#### 67、SpringBoot_数据访问-整合JPA

1）、编写一个实体类（bean）和数据表进行映射，并且配置好映射关系；

```java
//使用JPA注解配置映射关系
@Entity //告诉JPA这是一个实体类（和数据表映射的类）
@Table(name = "tbl_user") //@Table来指定和哪个数据表对应;如果省略默认表名就是user；
public class User {

    @Id //这是一个主键
    @GeneratedValue(strategy = GenerationType.IDENTITY)//自增主键
    private Integer id;

    @Column(name = "last_name",length = 50) //这是和数据表对应的一个列
    private String lastName;
    @Column //省略默认列名就是属性名
    private String email;

```

2）、编写一个Dao接口来操作实体类对应的数据表（Repository）

```java
//继承JpaRepository来完成对数据库的操作
public interface UserRepository extends JpaRepository<User,Integer> {
}


```

3）、基本的配置JpaProperties

```yaml
spring:
  datasource:
    url: jdbc:mysql://127.0.0.1:3306/jpa
    username: root
    password: 123456
    driver-class-name: com.mysql.jdbc.Driver
  jpa:
    hibernate:
      # 更新或者创建数据表结构
      ddl-auto: update
    # 控制台显示SQL
    show-sql: true

```

```xml
		<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
```



4）、controller

```jAVA
@RestController
public class UserController {

    @Autowired
    UserRepository userRepository;

    @GetMapping("/user/{id}")
    public User getUser(@PathVariable("id") Integer id) {
        // 用2.0这快会报错 换1.5就好了
        User user = userRepository.findOne(id);
        return null;
    }

    @GetMapping("/user")
    public User insertUser(User user) {
        User save = userRepository.save(user);
        return save;
    }
}
```

```java

```
