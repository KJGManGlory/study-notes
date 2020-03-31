### 一、关键注解
- `@Aspect`：指定切面类
- `@Component`：将切面类作为Spring容器的bean交由Spring容器管理
- `@Pointcut`：指定切入点
- `@Before`：前置通知
- `@AfterReturning`：后置通知
- `@AfterThrowing`：异常通知
- `@After`：最终通知
- `@Around`：环绕通知
- `@EnableAspectJAutoProxy`：开启aop

### 二、使用步骤
1. 使用`@EnableAspectJAutoProxy`注解开启AOP支持
2. 编写切面，使用`@Component`注解和`@Aspect`注解来指定切面类
3. 使用`@Pointcut`注解指定切入点，并且编写`execution(* com.lizza..*())`表达式来指定要切入的包
4. 使用`@Before`，`@AfterReturning`，`@AfterThrowing`，`@After`，`@Around`注解来指定前置通知，后置通知，异常通知，最终通知，环绕通知

### 三、示例代码
- 配置切面
```
@Aspect
@Component
public class Log {

    @Pointcut("execution(* com.lizza..*())")
    public void pointCut() {}

    /**
     * 前置通知：切入点执行之前执行
     */
    @Before("pointCut()")
    public void beforeLog() {
        System.out.println("前置通知：记录日志...");
    }

    /**
     * 后置通知：切入点执行之后执行
     */
    @AfterReturning("pointCut()")
    public void afterLog() {
        System.out.println("后置通知：记录日志...");
    }

    /**
     * 异常通知：切入点执行发生异常时执行
     */
    @AfterThrowing("pointCut()")
    public void exceptionLog() {
        System.out.println("异常通知：记录日志...");
    }

    /**
     * 最终通知：无论切入点执行发生异常与否，都会执行
     */
    @After("pointCut()")
    public void finalLog() {
        System.out.println("最终通知：记录日志...");
    }

    /**
     * 环绕通知
     */
    @Around("pointCut()")
    public Object aroundLog(ProceedingJoinPoint point) {
        Object result;
        try {
            System.out.println("环绕通知-前置：记录日志...");
            Object[] args = point.getArgs();            // 获取方法执行的参数
            result = point.proceed(args);               // 执行切入点方法
            System.out.println("环绕通知-后置：记录日志...");
            return result;
        } catch (Throwable t) {
            System.out.println("环绕通知-异常：记录日志...");
            throw new RuntimeException(t);
        } finally {
            System.out.println("环绕通知-最终：记录日志...");
        }
    }
}

```

- 开启aop
```
@Configuration
@EnableAspectJAutoProxy
@ComponentScan("com.lizza")
public class SpringConfig {

}
```

### 三、注意问题
- spring aop注解配置时，最终通知会在异常通知或后置通知之前执行，故实际应用时建议使用环绕通知
- spring aop环绕通知使用`ProceedingJoinPoint`来获取切入点的方法，参数

> 源码地址：[https://github.com/KJGManGlory/spring-framework](https://github.com/KJGManGlory/spring-framework)