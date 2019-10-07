### AOP功能测试

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aspects</artifactId>
    <version>4.3.12.RELEASE</version>
</dependency>
```



```java
/**
 * AOP：【动态代理】
 * 指在程序运行期间动态的将某段代码切入到指定方法指定位置进行运行的编程方式；
 * <p>
 * 1、导入aop模块；Spring AOP：(spring-aspects)
 * 2、定义一个业务逻辑类（MathCalculator）；在业务逻辑运行的时候将日志进行打印（方法之前、方法运行结束、方法出现异常，xxx）
 * 3、定义一个日志切面类（LogAspects）：切面类里面的方法需要动态感知MathCalculator.div运行到哪里然后执行；
 * 通知方法：
 * 前置通知(@Before)：logStart：在目标方法(div)运行之前运行
 * 后置通知(@After)：logEnd：在目标方法(div)运行结束之后运行（无论方法正常结束还是异常结束）
 * 返回通知(@AfterReturning)：logReturn：在目标方法(div)正常返回之后运行
 * 异常通知(@AfterThrowing)：logException：在目标方法(div)出现异常以后运行
 * 环绕通知(@Around)：动态代理，手动推进目标方法运行（joinPoint.procced()）
 * 4、给切面类的目标方法标注何时何地运行（通知注解）；
 * 5、将切面类和业务逻辑类（目标方法所在类）都加入到容器中;
 * 6、必须告诉Spring哪个类是切面类(给切面类上加一个注解：@Aspect)
 * [7]、给配置类中加 @EnableAspectJAutoProxy 【开启基于注解的aop模式】
 * 在Spring中很多的 @EnableXXX;
 * <p>
 * 三步：
 * 1）、将业务逻辑组件和切面类都加入到容器中；告诉Spring哪个是切面类（@Aspect）
 * 2）、在切面类上的每一个通知方法上标注通知注解，告诉Spring何时何地运行（切入点表达式）
 * 3）、开启基于注解的aop模式；@EnableAspectJAutoProxy
 */
@EnableAspectJAutoProxy//开启基于注解的aop模式
@Configuration
public class AopConfig {

    /*把业务逻辑类加入到容器中*/
    @Bean
    public MathCalculator calculator() {
        return new MathCalculator();
    }

    @Bean
    public LogAspects logAspects() {
        return new LogAspects();
    }
}

/*切面类 @Aspect： 告诉Spring当前类是一个切面类*/
@Aspect
public class LogAspects {
    /*
    *抽取公共的切点表达式
    * 1.本类引用
    * 2.其他的切面引入
    * */
    @Pointcut("execution(public int part05aop.MathCalculator.*(..))")
    public void pointCut(){}

    /*@Before在目标方法之前切入 切入点表达式 指定在哪一个方法切入*/
    @Before("pointCut()")
    public void logStart(JoinPoint joinPoint){
        Object[] args = joinPoint.getArgs();
        System.out.println(""+joinPoint.getSignature().getName()+"运行。。。@Before:参数列表是：{"+Arrays.asList(args)+"}");
    }

    @After("part05aop.LogAspects.pointCut()")
    public void logEnd(JoinPoint joinPoint){
        System.out.println(""+joinPoint.getSignature().getName()+"结束。。。@After");
    }

    //JoinPoint一定要出现在参数表的第一位
    @AfterReturning(value="pointCut()",returning="result")
    public void logReturn(JoinPoint joinPoint,Object result){
        System.out.println(""+joinPoint.getSignature().getName()+"正常返回。。。@AfterReturning:运行结果：{"+result+"}");
    }

    @AfterThrowing(value="pointCut()",throwing="exception")
    public void logException(JoinPoint joinPoint,Exception exception){
        System.out.println(""+joinPoint.getSignature().getName()+"异常。。。异常信息：{"+exception+"}");
    }
}

```

