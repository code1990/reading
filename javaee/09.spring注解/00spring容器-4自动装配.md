### 自动装配

自动装配;
	Spring利用依赖注入（DI），完成对IOC容器中中各个组件的依赖关系赋值；

1）、@Autowired：自动注入：
	1）、默认优先按照类型去容器中找对应的组件:applicationContext.getBean(BookDao.class);找到就赋值
	2）、如果找到多个相同类型的组件，再将属性的名称作为组件的id去容器中查找
						applicationContext.getBean("bookDao")
	3）、@Qualifier("bookDao")：使用@Qualifier指定需要装配的组件的id，而不是使用属性名
	4）、自动装配默认一定要将属性赋值好，没有就会报错；
		可以使用@Autowired(required=false);
	5）、@Primary：让Spring进行自动装配的时候，默认使用首选的bean；
			也可以继续使用@Qualifier指定需要装配的bean的名字
	BookService{
		@Autowired
		BookDao  bookDao;
	}

2）、Spring还支持使用@Resource(JSR250)和@Inject(JSR330)[java规范的注解]
	@Resource:
		可以和@Autowired一样实现自动装配功能；默认是按照组件名称进行装配的；
		没有能支持@Primary功能没有支持@Autowired（reqiured=false）;
	@Inject:
		需要导入javax.inject的包，和Autowired的功能一样。没有required=false的功能；
@Autowired:Spring定义的； @Resource、@Inject都是java规范

AutowiredAnnotationBeanPostProcessor:解析完成自动装配功能；		

3）、 @Autowired:构造器，参数，方法，属性；都是从容器中获取参数组件的值
	1）、[标注在方法位置]：@Bean+方法参数；参数从容器中获取;默认不写@Autowired效果是一样的；都能自动装配
	2）、[标在构造器上]：如果组件只有一个有参构造器，这个有参构造器的@Autowired可以省略，参数位置的组件还是可以自动从容器中获取
	3）、放在参数位置：

4）、自定义组件想要使用Spring容器底层的一些组件（ApplicationContext，BeanFactory，xxx）；
	自定义组件实现xxxAware；在创建对象的时候，会调用接口规定的方法注入相关组件；Aware；
	把Spring底层一些组件注入到自定义的Bean中；
	xxxAware：功能使用xxxProcessor；
		ApplicationContextAware==》ApplicationContextAwareProcessor；

-------------------------



#### 00、自动装配-@Autowired&@Qualifier&@Primary(案例合并)
#### 01、自动装配-@Resource&@Inject(案例合并)

```xml-dtd
<!-- https://mvnrepository.com/artifact/javax.inject/javax.inject -->
<!--@Inject注解需要额外引入-->
        <dependency>
            <groupId>javax.inject</groupId>
            <artifactId>javax.inject</artifactId>
            <version>1</version>
        </dependency>
```



```java
/*注入到容器时候 默认首字母小写*/
@Repository
public class BookDao {

    private boolean flag = false;

    public boolean isFlag() {
        return flag;
    }

    public void setFlag(boolean flag) {
        this.flag = flag;
    }

    @Override
    public String toString() {
        return "BookDao{" +
                "flag=" + flag +
                '}';
    }
}

@Service
public class BookService {

    //@Qualifier("bookDao") 指定别名
    //@Autowired(required=false) //默认装配则一定要注入required=true 否则报错
    //@Resource(name="bookDao2") 默认使用类名小写的方式 可以可以使用给定的名字
    @Inject//与@Autowired功能相同 无required=false
    private BookDao bookDao;

    public void print(){
        System.out.println(bookDao);
    }

    @Override
    public String toString() {
        return "BookService [bookDao=" + bookDao + "]";
    }
}

/**
 * 自动装配;
 * Spring利用依赖注入（DI），完成对IOC容器中中各个组件的依赖关系赋值；
 * <p>
 * 1）、@Autowired：自动注入：
 * 1）、默认优先按照类型去容器中找对应的组件:applicationContext.getBean(BookDao.class);找到就赋值
 * 2）、如果找到多个相同类型的组件，再将属性的名称作为组件的id去容器中查找
 * applicationContext.getBean("bookDao")
 * 3）、@Qualifier("bookDao")：使用@Qualifier指定需要装配的组件的id，而不是使用属性名
 * 4）、自动装配默认一定要将属性赋值好，没有就会报错；
 * 可以使用@Autowired(required=false);
 * 5）、@Primary：让Spring进行自动装配的时候，默认使用首选的bean；
 * 也可以继续使用@Qualifier指定需要装配的bean的名字
 * BookService{
 *
 * @Autowired BookDao  bookDao;
 * }
 * <p>
 * 2）、Spring还支持使用@Resource(JSR250)和@Inject(JSR330)[java规范的注解]
 * @Resource: 可以和@Autowired一样实现自动装配功能；默认是按照组件名称进行装配的；
 * 没有能支持@Primary功能没有支持@Autowired（reqiured=false）;
 * @Inject: 需要导入javax.inject的包，和Autowired的功能一样。没有required=false的功能；
 * @Autowired:Spring定义的； @Resource、@Inject都是java规范
 * <p>
 * AutowiredAnnotationBeanPostProcessor:解析完成自动装配功能；
 */
@Configuration
@ComponentScan("part03autowired")
public class AutowiredConfig {

    @Primary//设置优先级 多个当中 选择当前标注的这个
    @Bean("bookDao2")
    public BookDao bookDao() {
        BookDao bookDao = new BookDao();
        bookDao.setFlag(true);
        return bookDao;
    }

    
	/**
     * @param bookDao
     * @return
     * @Bean标注的方法创建对象的时候，方法参数BookDao的值从容器中获取
     * 可以写成@Autowired BookDao bookDao 因为从容器中获取所以可以省略不写
     */
    @Bean("getBookEntity")
    public BookEntity getBookEntity(BookDao bookDao) {
        BookEntity entity = new BookEntity(bookDao);
        return entity;
    }
}
```



#### 02、自动装配-方法、构造器位置的自动装配

```java
//默认加在ioc容器中的组件，容器启动会调用无参构造器创建对象，再进行初始化赋值等操作
/**
3）、 @Autowired:构造器，参数，方法，属性；都是从容器中获取参数组件的值
	1）、[标注在方法位置]：@Bean+方法参数；参数从容器中获取;默认不写@Autowired效果是一样的；都能自动装配
	2）、[标在构造器上]：如果组件只有一个有参构造器，这个有参构造器的@Autowired可以省略，参数位置的组件还是可以自动从容器中获取
	3）、放在参数位置：

**/
@Component
public class BookEntity {

    private BookDao bookDao;

    public BookEntity() {

    }
	//构造器要用的组件，都是从容器中获取
    //@Autowired 标注在构造器上只有一个有参数构造器 可以省去
    //public BookEntity(@AutowiredBookDao bookDao) { 标注在参数位置 参数从容器获取
    public BookEntity(BookDao bookDao) {
        this.bookDao = bookDao;
    }

    public BookDao getBookDao() {
        return bookDao;
    }
    //@Autowired
    //标注在方法，Spring容器创建当前对象，就会调用方法，完成赋值；
    //方法使用的参数，自定义类型的值从ioc容器中获取
    public void setBookDao(BookDao bookDao) {
        this.bookDao = bookDao;
    }

    @Override
    public String toString() {
        return "BookEntity{" +
                "bookDao=" + bookDao +
                '}';
    }
}

/**
     * @param bookDao
     * @return
     * @Bean标注的方法创建对象的时候，方法参数BookDao的值从容器中获取
     * 可以写成@Autowired BookDao bookDao 因为从容器中获取所以可以省略不写
     */
    @Bean("getBookEntity")
    public BookEntity getBookEntity(BookDao bookDao) {
        BookEntity entity = new BookEntity(bookDao);
        return entity;
    }
```



#### 03、自动装配-Aware注入Spring底层组件&原理

```java
/* * 4）、自定义组件想要使用Spring容器底层的一些组件（ApplicationContext，BeanFactory，xxx）；
 * 		自定义组件实现xxxAware；在创建对象的时候，会调用接口规定的方法注入相关组件；Aware；
 * 		把Spring底层一些组件注入到自定义的Bean中；
 * 		xxxAware：功能使用xxxProcessor；
 * 		ApplicationContextAware==》ApplicationContextAwareProcessor；
 */
@Component
public class AwareEntity implements ApplicationContextAware, BeanNameAware, EmbeddedValueResolverAware {

    private ApplicationContext applicationContext;

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        System.out.println("传入的ioc：" + applicationContext);
        this.applicationContext = applicationContext;
    }

    @Override
    public void setBeanName(String name) {
        System.out.println("当前bean的名字：" + name);
    }

    @Override
    public void setEmbeddedValueResolver(StringValueResolver resolver) {
        String resolveStringValue = resolver.resolveStringValue("你好 ${os.name} 我是 #{20*18}");
        System.out.println("解析的字符串：" + resolveStringValue);
    }

}
```



#### 04、自动装配-@Profile环境搭建

#### 05、自动装配-@Profile根据环境注册bean

```java
/**
 * Profile：
 * 		Spring为我们提供的可以根据当前环境，动态的激活和切换一系列组件的功能；
 *
 * 开发环境、测试环境、生产环境；
 * 数据源：(/A)(/B)(/C)；
 *
 *
 * @Profile：指定组件在哪个环境的情况下才能被注册到容器中，不指定，任何环境下都能注册这个组件
 *
 * 1）、加了环境标识的bean，只有这个环境被激活的时候才能注册到容器中。默认是default环境
 * 2）、写在配置类上，只有是指定的环境的时候，整个配置类里面的所有配置才能开始生效
 * 3）、没有标注环境标识的bean在，任何环境下都是加载的；
 */

@PropertySource("classpath:/jdbc.properties")
@Configuration
public class ProfileConfig implements EmbeddedValueResolverAware {

    @Value("${db.user}")
    private String user;

    private StringValueResolver valueResolver;

    private String  driverClass;

	//没有被 @Profile标识的 任何时候都会加载
    @Bean
    public NoProfileEntity noProfileEntity(){
        return new NoProfileEntity();
    }
	//@Profile("test") test环境下加载
    @Profile("test")
    @Bean("testDataSource")
    public DataSource dataSourceTest(@Value("${db.password}")String pwd) throws Exception{
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setUser(user);
        dataSource.setPassword(pwd);
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/test");
        dataSource.setDriverClass(driverClass);
        return dataSource;
    }

	//@Profile("dev") dev环境下加载
    @Profile("dev")
    @Bean("devDataSource")
    public DataSource dataSourceDev(@Value("${db.password}")String pwd) throws Exception{
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setUser(user);
        dataSource.setPassword(pwd);
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/ssm_crud");
        dataSource.setDriverClass(driverClass);
        return dataSource;
    }
	//@Profile("prod") prod环境下加载
    @Profile("prod")
    @Bean("prodDataSource")
    public DataSource dataSourceProd(@Value("${db.password}")String pwd) throws Exception{
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setUser(user);
        dataSource.setPassword(pwd);
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/scw_0515");

        dataSource.setDriverClass(driverClass);
        return dataSource;
    }

    @Override
    public void setEmbeddedValueResolver(StringValueResolver resolver) {
        this.valueResolver = resolver;
        driverClass = valueResolver.resolveStringValue("${db.driverClass}");
    }

}

public class ProfileTest {
    //1、使用命令行动态参数: 在虚拟机参数位置加载 -Dspring.profiles.active=test
    
    
    //2、代码的方式激活某种环境；
    @Test
    public void test01(){
        //1、创建一个空参数的applicationContext
        AnnotationConfigApplicationContext applicationContext =
                new AnnotationConfigApplicationContext();
        //2、设置需要激活的环境
        applicationContext.getEnvironment().setActiveProfiles("dev");
        //3、注册主配置类
        applicationContext.register(ProfileConfig.class);
        //4、启动刷新容器
        applicationContext.refresh();

		//遍历所有的容器对象
        String[] namesForType = applicationContext.getBeanNamesForType(DataSource.class);
        for (String string : namesForType) {
            System.out.println(string);
        }
		
        NoProfileEntity bean = applicationContext.getBean(NoProfileEntity.class);
        System.out.println(bean);
        applicationContext.close();
    }
}
```


