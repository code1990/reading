### 属性赋值

#### 01.属性赋值-@Value赋值

```java
public class PropertyEntity {

    //使用@Value赋值；
    //1、基本数值
    //2、可以写SpEL； #{}
    //3、可以写${}；取出配置文件【properties】中的值（在运行环境变量里面的值）

    @Value("zhangsan")
    private String name;
    @Value("#{20-2}")
    private Integer age;
    @Value("${PropertyEntity.nickName}")
    private String nickName;
    
}    
```



#### 02.属性赋值-@PropertySource加载外部配置文件

```java
/*指定配置文件的位置 
   1.可以使用多个@PropertySource
 * 2.使用@PropertySources指定多个PropertySource
 * 3.一个PropertySource指定多个value数组值
 
 配置文件的参数值会被设置为运行时环境变量 通过key-value方式取值
 ConfigurableEnvironment environment = applicationContext.getEnvironment();
 String property = environment.getProperty("PropertyEntity.nickName");
 
 * */
@PropertySource(value={"classpath:/propertyEntity.properties"})
@Configuration
public class PropertyConfig {
    @Bean("getPropertyEntity")
    public PropertyEntity getPropertyEntity(){
        return new PropertyEntity();
    }
}
```

