### 组件注册的几种方式

#### 01给容器中注册组件；

 1）、包扫描+组件标注注解（@Controller/@Service/@Repository/@Component）[自己写的类]
 2）、@Bean[导入的第三方包里面的组件]
 3）、@Import[快速给容器中导入一个组件]
 		1）、@Import(要导入到容器中的组件)；容器中就会自动注册这个组件，id默认是全类名
 		2）、ImportSelector:返回需要导入的组件的全类名数组；
 		3）、ImportBeanDefinitionRegistrar:手动注册bean到容器中
 4）、使用Spring提供的 FactoryBean（工厂Bean）;
 		1）、默认获取到的是工厂bean调用getObject创建的对象
 		2）、要获取工厂Bean本身，我们需要给id前面加一个&，&colorFactoryBean

------------------

#### 02.@Configuration&@Bean给容器中注册组件

以前使用xml配置文件的方式注入bean 现在可以采取@Configuration的方式

```java
//告诉Spring这是一个配置类 配置类==配置文件
@Configuration
public class ConfigurationAnnotation {
    /*给容器中注册一个Bean;类型为返回值的类型，id默认是用方法名作为id*/
    /*如果指定了value值 则id为给定的value值 如下id=person 而不是方法名*/
    @Bean("person")
    public Person person01(){
        return new Person("lisi",20);
    }
}
```



#### 03.@ComponentScan-自动扫描组件&指定扫描规则

```java
@Configuration
//如果是jdk1.8 可以分别指定多个扫描规则
//@ComponentScan()
//@ComponentScan()

//如果不是jdk1.8 可以使用 @ComponentScans 在内部指定扫描规则
@ComponentScans(
    value = {
            @ComponentScan(value="part00componetregister",includeFilters = {
/*						@Filter(type=FilterType.ANNOTATION,classes={Controller.class}),
                    @Filter(type=FilterType.ASSIGNABLE_TYPE,classes={BookService.class}),*/
                    @Filter(type=FilterType.CUSTOM,classes={MyTypeFilter.class})
            },useDefaultFilters = false)
    }
)
//@ComponentScan  value:指定要扫描的包
//excludeFilters = Filter[] ：指定扫描的时候按照什么规则排除那些组件
//includeFilters = Filter[] ：指定扫描的时候只需要包含哪些组件 必须设置useDefaultFilters = false 默认全部扫描useDefaultFilters = true 
//FilterType.ANNOTATION：按照注解类型扫描 @Filter(type=FilterType.ANNOTATION,classes={Controller.class})扫描所有的controller
//FilterType.ASSIGNABLE_TYPE：按照给定的类型扫描；@Filter(type=FilterType.ASSIGNABLE_TYPE,classes={BookService.class})扫描所有的BookService及其子类
//FilterType.ASPECTJ：使用ASPECTJ表达式（很少使用）
//FilterType.REGEX：使用正则指定
//FilterType.CUSTOM：使用自定义规则
public class ComponentScanAnnotation {
    @Bean("person")
    public Person person01(){
        return new Person("lisi",20);
    }
}
```



#### 04.自定义TypeFilter指定过滤规则

```java
public class MyTypeFilter implements TypeFilter {
    /**
     *metadataReader 当前正在读取的类
     * MetadataReaderFactory 可以获取任何的其他的类信息
     * @param
     * @return false 默认表示一个都不匹配 true表示匹配
     */
    @Override
    public boolean match(MetadataReader metadataReader, MetadataReaderFactory metadataReaderFactory) throws IOException {
        /*获取当前类的注解的信息*/
        AnnotationMetadata annotationMetadata = metadataReader.getAnnotationMetadata();
        /*获取当前正在扫描的类的类信息*/
        ClassMetadata classMetadata = metadataReader.getClassMetadata();
        /*获取当前类路径*/
        Resource resource = metadataReader.getResource();
        String className = classMetadata.getClassName();
        System.out.println(">>>>>>"+className);
        if(className.contains("er")){
            /*配置成功*/
            return  true;
        }
        return false;
    }
}
```



#### 05.@Scope-设置组件作用域

```java
@Configuration
public class ScopeAnnotation {

    /*默认是单实例的
    *
    *@Scope:调整bean作用域
    * prototype：多实例的：ioc容器启动并不会去调用方法创建对象放在容器中。
    * 					每次获取的时候才会调用方法创建对象；
    * singleton：单实例的（默认值）：ioc容器启动会调用方法创建对象放到ioc容器中。
    * 			以后每次获取就是直接从容器（map.get()）中拿，
    * request：同一次请求创建一个实例
    * session：同一个session创建一个实例
    * */
    @Scope("prototype")
    @Bean("person")
    public Person person(){
        System.out.println("给容器中添加Person....");
        return new Person("张三", 25);
    }
}

```



#### 06.@Lazy-bean懒加载
```java
@Configuration
public class LazyAnnotation {

    /*
     *
     *@Lazy:懒加载： 主要针对单例对象
     *
     * 		单实例bean：默认在容器启动的时候创建对象；
     *
     * 		懒加载：容器启动不创建对象。第一次使用(获取)Bean创建对象，并初始化；
     */
    @Lazy
    @Bean("person")
    public Person person() {
        System.out.println("给容器中添加Person....");
        return new Person("张三", 25);
    }
}

```




#### 07.@Conditional-按照条件注册bean
```java
//类中组件统一设置。满足当前条件，这个类中配置的所有bean注册才能生效；
@Conditional({WindowsCondition.class})
@Configuration
public class ConditionalAnnotation {
    /**
     * @Conditional({Condition}) ： 
     * 按照一定的条件进行判断，满足条件给容器中注册bean
     * 满足给定条件的类或者方法才能往ioc容器中添加
     *
     * 如果系统是windows，给容器中注册("bill")
     * 如果是linux系统，给容器中注册("linus")
     */
//    @Conditional({WindowsCondition.class})
    @Bean("bill")
    public Person person01(){
        return new Person("Bill Gates",62);
    }

    @Conditional(LinuxCondition.class)
    @Bean("linus")
    public Person person02(){
        return new Person("linus", 48);
    }
}

//判断是否windows系统
public class WindowsCondition implements Condition {

	@Override
	public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
		Environment environment = context.getEnvironment();
		String property = environment.getProperty("os.name");
		if(property.contains("Windows")){
			return true;
		}
		return false;
	}

}

//判断是否linux系统
public class LinuxCondition implements Condition {

    /**
     * ConditionContext：判断条件能使用的上下文（环境）
     * AnnotatedTypeMetadata：注释信息
     */
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        // TODO是否linux系统
        //1、能获取到ioc使用的beanfactory
        ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
        //2、获取类加载器
        ClassLoader classLoader = context.getClassLoader();
        //3、获取当前环境信息
        Environment environment = context.getEnvironment();
        //4、获取到bean定义的注册类
        BeanDefinitionRegistry registry = context.getRegistry();

        String property = environment.getProperty("os.name");

        //可以判断容器中的bean注册情况，也可以给容器中注册bean
        boolean definition = registry.containsBeanDefinition("person");
        if (property.contains("linux")) {
            return true;
        }

        return false;
    }

}
```




#### 08.@Import-给容器中快速导入一个组件
```java
@Configuration
@Import(ImportEntity.class)//快速导入ImportEntity类
public class ImportAnnotation {


}
```




#### 09.@Import-使用ImportSelector
```java
//自定义逻辑返回需要导入的组件
public class MyImportSelector implements ImportSelector {

    //返回值，就是到导入到容器中的组件全类名
    //AnnotationMetadata:当前标注@Import注解的类的所有注解信息
    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        // TODO Auto-generated method stub
        //importingClassMetadata
        //方法不要返回null值
        return new String[]{"Person","Person"};
    }

}
```




#### 10.@Import-使用ImportBeanDefinitionRegistrar
```java
public class MyImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {

    /**
     * AnnotationMetadata：当前类的注解信息
     * BeanDefinitionRegistry:BeanDefinition注册类；
     * 		把所有需要添加到容器中的bean；调用
     * 		BeanDefinitionRegistry.registerBeanDefinition手工注册进来
     */
    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {

        boolean definition = registry.containsBeanDefinition("part00componetregister.Person");
        boolean definition2 = registry.containsBeanDefinition("part00componetregister.Person");
        if(definition && definition2){
            //指定Bean定义信息；（Bean的类型，Bean。。。）
            RootBeanDefinition beanDefinition = new RootBeanDefinition(ImportEntity.class);
            //注册一个Bean，指定bean名
            registry.registerBeanDefinition("rainBow", beanDefinition);
        }
    }

}
```




#### 11.使用FactoryBean注册组件
```java
@Configuration
public class FactoryBeanAnnotation {

    @Bean("factoryBeanEntity")
    public FactoryBeanEntity getFactoryBean(){
        return new FactoryBeanEntity();
    }
}

//创建一个Spring定义的FactoryBean
public class FactoryBeanEntity implements FactoryBean<ImportEntity> {
    //返回一个ImportEntity对象，这个对象会添加到容器中
    @Override
    public ImportEntity getObject() throws Exception {
        System.out.println("FactoryBeanEntity...getObject...");
        return new ImportEntity();
    }

    @Override
    public Class<?> getObjectType() {
        return ImportEntity.class;
    }

    //是单例？
    //true：这个bean是单实例，在容器中保存一份
    //false：多实例，每次获取都会创建一个新的bean；
    @Override
    public boolean isSingleton() {
        return false;
    }
}
```




