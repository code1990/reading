### springbean的生命周期

#### 01、生命周期总结

1）、指定初始化和销毁方法；
		通过@Bean指定init-method和destroy-method；
2）、通过让Bean实现InitializingBean（定义初始化逻辑），
				DisposableBean（定义销毁逻辑）;
3）、可以使用JSR250；
		@PostConstruct：在bean创建完成并且属性赋值完成；来执行初始化方法
		@PreDestroy：在容器销毁bean之前通知我们进行清理工作
4）、BeanPostProcessor【interface】：bean的后置处理器；
		在bean初始化前后进行一些处理工作；
		postProcessBeforeInitialization:在初始化之前工作
		postProcessAfterInitialization:在初始化之后工作

Spring底层对 BeanPostProcessor 的使用；
bean赋值，注入其他组件，@Autowired，生命周期注解功能，@Async,xxx BeanPostProcessor;

-------------------------



#### 02、生命周期-@Bean指定初始化和销毁方法

```java
/**
 * bean的生命周期：
 * bean创建---初始化----销毁的过程
 * 容器管理bean的生命周期；
 * 我们可以自定义初始化和销毁方法；容器在bean进行到当前生命周期的时候来调用我们自定义的初始化和销毁方法
 * <p>
 * 构造（对象创建）
 * 单实例：在容器启动的时候创建对象
 * 多实例：在每次获取的时候创建对象\
 * <p>
 * 1）、指定初始化和销毁方法；
 * 通过@Bean指定init-method和destroy-method；
 */
@Configuration
public class BeanAnnotationLifeCycle {

    //@Scope("prototype") 多例模式容器不会销毁对象
    @Bean(initMethod = "init", destroyMethod = "destroy")
    public Car getCar() {
        return new Car();
    }

}

public class Car {

    public Car() {
        System.out.println("car constructor");
    }
    public void init(){
        System.out.println("init method");
    }
    public void destroy(){
        System.out.println("destroy method");
    }

}
```



#### 03、生命周期-InitializingBean和DisposableBean

```java
/*通过让Bean实现InitializingBean（定义初始化逻辑），
 * 				DisposableBean（定义销毁逻辑）;
 * */
@Component
public class InitializingDisposableBean implements InitializingBean, DisposableBean {

    public InitializingDisposableBean() {
        System.out.println("InitializingDisposableBean constructor");
    }

    @Override
    public void destroy() throws Exception {
        System.out.println("InitializingDisposableBean destroy");
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("InitializingDisposableBean afterPropertiesSet");
    }
}

//ComponentScan扫描part01lifecycle包下的使用@Component注解标识InitializingDisposableBean注册到容器中
@ComponentScan("part01lifecycle")
@Configuration
public class BeanAnnotationLifeCycle {

    //@Scope("prototype")
    @Bean(initMethod = "init", destroyMethod = "destroy")
    public Car getCar() {
        return new Car();
    }

}
```



#### 04、生命周期-@PostConstruct&@PreDestroy

```java
/**
 * @PostConstruct：在bean创建完成并且属性赋值完成；来执行初始化方法
 * @PreDestroy：在容器销毁bean之前通知我们进行清理工作
 */
@Component
public class PostConstructPreDestroy {
    public PostConstructPreDestroy() {
        System.out.println("PostConstructPreDestroy constructor");
    }

    /*//对象创建并赋值之后调用*/
    @PostConstruct
    public void init() {
        System.out.println("PostConstructPreDestroy @PostConstruct");
    }

    //容器移除对象之前
    @PreDestroy
    public void destroy() {
        System.out.println("PostConstructPreDestroy @PreDestroy");
    }
}
```



#### 05、生命周期-BeanPostProcessor-后置处理器

```java
/**
 * 后置处理器：初始化前后进行处理工作
 * 将后置处理器加入到容器中
 * <p>
 * 1.构造器构造对象
 * <p>
 * 2.BeanPostProcessor.postProcessBeforeInitialization
 * <p>
 * 3.初始化：对象创建完成，并赋值好，调用初始化方法。。。
 * <p>
 * 4.BeanPostProcessor.postProcessAfterInitialization
 * 5.销毁：
 * 
 * BeanPostProcessor【interface】：bean的后置处理器；
 * 	在bean初始化前后进行一些处理工作；
 * 	postProcessBeforeInitialization:在初始化之前工作
 * 	postProcessAfterInitialization:在初始化之后工作
 */
@Component
public class MyBeanPostProcessor implements BeanPostProcessor {

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("postProcessBeforeInitialization..." + beanName + "=>" + bean);
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("postProcessAfterInitialization..." + beanName + "=>" + bean);
        return bean;
    }

}
```



#### 06、生命周期-BeanPostProcessor原理

```java

```



#### 07、生命周期-BeanPostProcessor在Spring底层的使用

```java

```


