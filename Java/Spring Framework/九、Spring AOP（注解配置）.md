### 一、关键注解
- `@Aspect`：指定切面类
- `@Pointcut`：指定切入点
- `@Before`：前置通知
- `@AfterReturning`：后置通知
- `@AfterThrowing`：异常通知
- `@After`：最终通知
- `@Around`：环绕通知
- `@EnableAspectJAutoProxy`：开启aop

### 二、示例代码
- 切面
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