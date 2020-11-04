# AOP实例---日志模块

AOP能够使系统服务（例如：日志模块、安全模块、事务管理）模块化，并以声明的方式将它们应用到它们需要影响的组件中去。使业务组件会具有更高的内聚性并且会更加关注自身的业务，完全不需要了解涉及系统服务所带来复杂性。



------------------

> 日志模块就属于一种系统服务，业务组件不需要自己编写日志逻辑，而是将日志这种渗透到整个系统的服务切面化，利用AOP编写好日志逻辑，并声明到需要记录日志的组件当中。本文以项目中比较常用的日志为例，讲解AOP及注解的部分功能 。
>



### 1.切面化日志模块



##### 1.1导入jar包

使用AOP首先需要导入AspectJ的jar包**aspectjweaver-1.9.5.jar**

我用的是springboot，可以直接在pom中配置spring-boot-starter-aop，通过maven导入

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

可以看到**aspectjweaver-1.9.5.jar**就在spring-boot-starter-aop下。

![image-20201030093044599](C:%5CUsers%5C%E6%9D%8E%E4%BD%B3%E5%BA%86%5CDesktop%5C%E7%AC%94%E8%AE%B0%E6%9C%AC%5C%E5%9B%BE%E7%89%87%E5%BA%93%5Cimage-20201030093044599.png)

AspectJ是一个面向切面的框架，它扩展了Java语言。AspectJ定义了AOP语法。Spring Aop只是借助了AspectJ的AOP语法，即注解（例如@Aspect，@Pointcut）。



##### 1.2自定义日志注解

```Java
@Target(ElementType.METHOD)	//将注解作用于方法上。（因为要在执行方法时打印日志）
@Retention(RetentionPolicy.RUNTIME)	// jvm加载后注解仍然存在，用于反射的实现
public @interface AopLog {
    // 方法所属模块
    String module() default "";
    // 方法具体内容
    String operation() default "";
}

```

需要什么就加什么。*对于自定义注解的创建、处理及调用请参考 [自定义注解](https://www.cnblogs.com/peida/archive/2013/04/26/3038503.html)*

日志实体类：

```Java
public class AopLog {
    // 所属模块
    private String module;
    // 具体操作
    private String operation;
    // 执行时间
    private String execTime;

    public String getModule() {
        return module;
    }

    public void setModule(String module) {
        this.module = module;
    }

    public String getOperation() {
        return operation;
    }

    public void setOperation(String operation) {
        this.operation = operation;
    }

    public String getExecTime() {
        return execTime;
    }

    public void setExecTime(String execTime) {
        this.execTime = execTime;
    }

    @Override
    public String toString() {
        return "AopLog{" +
                "module='" + module + '\'' +
                ", operation='" + operation + '\'' +
                ", execTime='" + execTime + '\'' +
                '}';
    }
}
```



##### 1.3切面声明

*AOP的概念可以参考[spring官网](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop)*

```Java
import org.aspectj.lang.annotation.Aspect;

@Component	// 开启组建扫描
@Aspect	// 声明为切面
public class LogAspect {
    
    //声明一个Pointcut切入点
    @Pointcut("@annotation(com.zmxqq.annotation.AopLog)")//（切入点指示器+声明需要执行的方法）
    public void pointcut() {}// 切入点签名（访问修饰符+void+方法名+任何参数）
    
    // 声明一个‘切入点pointcut()’的环绕通知
    @Around("pointcut()")
    public Object aroundLog(ProceedingJoinPoint pjp) throws Throwable {
        Object result;

        // ProceedingJoinPoint:正在增强的连接点，也就是正在执行的方法
        // 获取目标对象的Class对象
        Class<?> targetClass = pjp.getTarget().getClass();
        // 通过pjp获取当前执行方法的标签(包括方法名称，参数等)
        MethodSignature signature = (MethodSignature) pjp.getSignature();
        // 根据方法名及参数获取指定的方法。
        Method targetMethod = getDeclaredMethod(targetClass,signature.getName(),signature.getMethod().getParameterTypes());
        // 获取当前方法的注解的对象(这里的AopLog是自定义注解)
        AopLog aopLogAnnotation = targetMethod.getAnnotation(AopLog.class);

        // 记录方法执行的时间与执行结束的时间
        long starTime = System.currentTimeMillis();
            // 调用proceed()，开始执行方法
        result = pjp.proceed();
        long endTime = System.currentTimeMillis();

        // 获取当前方法注解的参数，并存入到AopLog注解实体类中
        String module = aopLogAnnotation.module();
        String operation = aopLogAnnotation.operation();
            // 注解与实体类有一个写上全路径名避免名字冲突
        com.zmxqq.model.AopLog aopLog = new com.zmxqq.model.AopLog();
        aopLog.setModule(module);
        aopLog.setOperation(operation);
        aopLog.setExecTime((double)(endTime-starTime) + "秒");

        // 打印日志
        System.out.println(aopLog.toString());

        return result;
    }

    // 根据方法名，参数类型获取本类方法，如果没有就去父类中查找
    protected Method getDeclaredMethod(Class<?> clazz, String name, Class<?>... parameterTypes) {
        try {
            return clazz.getDeclaredMethod(name, parameterTypes);
        } catch (NoSuchMethodException e) {
            Class<?> superClass = clazz.getSuperclass();
            if (superClass != null) {
                return getDeclaredMethod(superClass, name, parameterTypes);
            }
        }
        return null;
    }
}
```

在controller层的方法中加上@AopLog日志注解，然后调用此方法：

```Java
@GetMapping("/Test/AopTest")
@AopLog(module = "测试模块", operation = "打印日志")
public void AopTest() {
    System.out.println("日志注解测试")
}
```

**详细解读下都需要哪些步骤：**

1. 首先利用最开始导入的AspectJ的jar包所提供的注解@Aspect声明一个切面类，@Component注解开启组件扫描，让spring可以扫描到这个切面。

2. 切面类内部用@Pointcut声明一个切入点（当然@Pointcut也是AspectJ包中的），格式如代码中所示，其中切入点指示器@annotation将只匹配给定注解的连接点（连接点是指目标对象的方法执行，切入点是连接点的集合），上述代码中就将匹配限制在了使用了com.zmxqq.annotation路径下的AopLog注解的连接点，即所有使用@AopLog注解的方法。*[注意：切入点签名返回值类型必须是void]（其他切入点指示器规则可以查看官网或我的[Spring官方文档笔记](https://blog.csdn.net/ZMXQQ233/article/details/109321131)）*

3. 利用切入点限制了具体的方法之后，就需要对这些方法执行某些操作，也就是AOP所说的通知。通知与切入点表达式相关联，并在切入点匹配的方法执行之前、之后或周围运行。我这里用了环绕通知，其中ProceedingJoinPoint继承自JoinPoint，表示正在增强的连接点，也就是正在执行的方法。再通过反射获取正在执行的这个方法的注解与注解的参数信息，保存到数据库或打印这些信息。我这里打印出来：

   ​	![image-20201103142936135](C:%5CUsers%5C%E6%9D%8E%E4%BD%B3%E5%BA%86%5CDesktop%5C%E7%AC%94%E8%AE%B0%E6%9C%AC%5C%E5%9B%BE%E7%89%87%E5%BA%93%5Cimage-20201103142936135.png)

4. 调用pjp.proceed();将执行当前方法并返回一个Object对象，代码中在方法执行前后记录了时间用来计算执行时间。这部分可以自由发挥。



一个简单的基于AOP的日志模块完成，此外还有① 前置通知 `@Before`，② 后置通知 `@AfterReturning，`③ 抛出异常后通知 `@AfterThrowing`等通知注解，可以自行扩展。







