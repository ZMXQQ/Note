# Spring

[官方文档，和任何一本spring书籍相比，它都更新更全](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html)

[TOC]



## **1. 控制反转**（Inversion of Control，IoC）

> 是面向对象编程中的一种设计原则，降低代码之间的耦合度。最常见的实现方式叫做依赖注入（Dependency Injection，DI），还有依赖查找（Dependency Lookup）
>



### 1.1 依赖注入（dependency injection，DI）

> **依赖：**a类中的某个属性是其他类，或构造方法中传入了其他类，就叫a依赖了这个类
>
> **作用：**DI所带来的最大收益——松耦合。如果一个对象只通过接口（而不是具体实现或初始化过程）来表明依赖关系，那么这种依赖就能够在对象本身毫不知情的情况下，用不同的具体实现进行替换。 
>
> **实现：**创建应用组件之间协作的行为通常称为装配（wiring）。





#### （1）Spring有三种装配bean的方式

> **xml，annotation（注解）和javaconfig**

##### 	**①xml配置方式装配：**[xml配置方式](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-constructor-injection)

> ​	配合setter方法注入<property>
>

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <!-- setter injection using the nested ref element -->
    <property name="beanOne">
        <ref bean="anotherExampleBean"/>
    </property>

    <!-- setter方法注入使用更简洁的ref属性 -->
    <property name="beanTwo" ref="yetAnotherBean"/>
    <property name="integerProperty" value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```

> ​	配合构造方法注入<constructor-arg>
>

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <!-- constructor injection using the nested ref element -->
    <constructor-arg>
        <ref bean="anotherExampleBean"/>
    </constructor-arg>

    <!-- 构造函数注入使用更简洁的ref属性 -->
    <constructor-arg ref="yetAnotherBean"/>

    <constructor-arg type="int" value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```



##### 	**②注解方式装配：**

1. @Component：可以用于注册所有bean
2. @Repository：主要用于注册dao层的bean
3. @Controller：主要用于注册控制层的bean
4. @Service：主要用于注册服务层的bean



##### 	**③JavaConfig装配:**

> @Configuration只能标记在类上，表示该类为JavaConfig类，使其可以被Spring IOC容器扫描识别并创建Bean加入到容器中。相当于以往的一个xml文件。
>
> @Bean只能标记在方法上，表示该方法返回一个Spring Bean，可以被IOC容器托管，相当于以前在xml文件中写的<bean/>元素。
>

```java
@Configuration
public class AppConfig{
    @Bean
    public MyBean myBean(){
        // instantiate, configure and return bean ...
        return new MyBean();
    }
}
```

*通过使用AnnotationConfigApplicationContext取代原来的ClassPathXmlApplicationContext，可以达到使用JavaConfig替代XmlConfig的目的。*



#### **（2）注入方式：**



##### 	**①基于构造方式注入**Constructor-based Dependency Injection

```Java
public class SimpleMovieLister {

    // the SimpleMovieLister has a dependency on a MovieFinder
    private MovieFinder movieFinder;

    // 用构造函数以便Spring容器可以注入依赖
    // a constructor so that the Spring container can inject a MovieFinder
    public SimpleMovieLister(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // 省略... business logic that actually uses the injected MovieFinder is omitted...
}
```



##### 	**②基于setter方法注入**Setter-based Dependency Injection

```Java
public class SimpleMovieLister {

    // the SimpleMovieLister has a dependency on the MovieFinder
    private MovieFinder movieFinder;

    // a setter method so that the Spring container can inject a MovieFinder
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // 省略... business logic that actually uses the injected MovieFinder is omitted...
}
```



*用构造器参数实现强制依赖，setter方法实现可选依赖。*



##### **③注解方式注入**

> `@Autowired`默认使用byType，匹配不到bean时再使用byName，此时的name是属性的名（xml中的byName是根据set方法名）。可以使用`@Qualifier()`指定bean的id。默认情况下必须要求依赖对象必须存在，如果要允许null值，可以设置它的required属性为false，如：@Autowired(required=false) 。
>
> `@Resource`默认使用byName，当找不到与名称匹配的bean时才按照类型进行装配。但是需要注意的是，如果name属性一旦指定，就只会按照名称进行装配。可以`@Resource(name = / type =)`指定name与type。



#### **（3）自动装配：**

**定义：**

仅仅需要在类的定义中提供依赖，取消在spring配置文件中的描述。

**优点：**

①自动装配可以显著减少指定属性或构造函数参数的需要

②自动装配可以随着对象的发展更新配置。

**缺点：**

①属性和构造参数设置中的显式依赖关系总是覆盖自动装配。

②自动装配不如显式布线精确。spring管理对象之间的关系不再被明确地记录。

③如果没有可用的唯一bean定义，则抛出异常。

**方式：**

使用`<bean />`元素的autowire属性为bean定义指定自动装配模式

例如`<bean id="" class="" autowire="byType|byName|constructor|default" />`

| Mode          | Explanation                                                  |
| :------------ | :----------------------------------------------------------- |
| `no`          | (Default) No autowiring. Bean references must be defined by `ref` elements. Changing the default setting is not recommended for larger deployments, because specifying collaborators explicitly gives greater control and clarity. To some extent, it documents the structure of a system.                                                                                                         (默认)没有自动装配。Bean引用必须由ref元素定义。对于较大的部署，不建议更改默认设置，因为显式地指定collaborator可以提供更好的控制和清晰度。在某种程度上，它记录了系统的结构。 |
| `byName`      | Autowiring by property name. Spring looks for a bean with the same name as the property that needs to be autowired. For example, if a bean definition is set to autowire by name and it contains a `master` property (that is, it has a `setMaster(..)` method), Spring looks for a bean definition named `master` and uses it to set the property.                                                                             按属性名称自动装配。Spring寻找与需要自动实现的属性同名的bean。例如，如果一个bean定义按名称设置为autowire，并且它包含一个master属性(也就是说，它有一个setMaster(..)方法)，那么Spring将查找一个名为master的bean定义，并使用它来设置该属性。 |
| `byType`      | Lets a property be autowired if exactly one bean of the property type exists in the container. If more than one exists, a fatal exception is thrown, which indicates that you may not use `byType` autowiring for that bean. If there are no matching beans, nothing happens (the property is not set).                                                                           如果容器中恰好存在该属性类型的一个bean，则允许该属性自动实现。如果存在多个，就会抛出一个致命异常，这表明您不能对该bean使用byType自动装配。如果没有匹配的bean，则什么也不会发生(没有设置属性)。 |
| `constructor` | Analogous to `byType` but applies to constructor arguments. If there is not exactly one bean of the constructor argument type in the container, a fatal error is raised.                                                                                   类似于byType，但适用于构造函数参数。如果容器中没有构造函数参数类型的确切bean，就会引发致命错误。 |





#### **（4）懒加载：**

**目的：**

> ​	默认情况下，作为初始化过程的一部分，ApplicationContext实现会急切地创建和配置所有的单例bean。通常，这种预实例化是可取的，因为配置或周围环境中的错误是立即发现的，而不是几小时甚至几天后发现的。当这种行为不合适时，可以通过将bean定义标记为延迟初始化来防止单例bean的预实例化。延迟初始化的bean告诉IoC容器在第一次请求bean实例时(而不是在启动时)创建bean实例。
>

**实现：**

> ​	在XML中，懒加载由`<bean />`元素上的lazy-init属性控制，如以下示例所示：
>

```xml
<bean id="lazy" class="com.something.ExpensiveToCreateBean" lazy-init="true"/>

// 设置容器级别的懒加载（所有bean都懒加载）
<beans default-lazy-init="true">
    <!-- no beans will be pre-instantiated... -->
</beans>
```

**注意：**

> ​	当延迟初始化的bean是未延迟初始化的单例bean的依赖项时，ApplicationContext在启动时（而不是第一次请求时）就创建延迟初始化的bean，因为它必须满足单例的依赖项。
>





#### **（5）Bean作用域：**

| Scope                                                        | Description                                                  |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [singleton](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-scopes-singleton)  单例模式 | (Default) Scopes a single bean definition to a single object instance for each Spring IoC container.                                                                                                                                         (默认情况下)将每个Spring IoC容器的单个bean定义定位到单个对象实例。 |
| [prototype ](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-scopes-prototype)原型模式 | Scopes a single bean definition to any number of object instances.                                                      将单个bean定义作用于任意数量的对象实例。 |
| [request](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-scopes-request) | Scopes a single bean definition to the lifecycle of a single HTTP request. That is, each HTTP request has its own instance of a bean created off the back of a single bean definition. Only valid in the context of a web-aware Spring `ApplicationContext`.                          将单个bean定义定位到单个HTTP请求的生命周期。也就是说，每个HTTP请求都有它自己的bean实例，该实例是在单个bean定义的后面创建的。仅在支持web的Spring ApplicationContext上下文中有效。 |
| [session](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-scopes-session) | Scopes a single bean definition to the lifecycle of an HTTP `Session`. Only valid in the context of a web-aware Spring `ApplicationContext`.                                                                            将单个bean定义作用于HTTP会话的生命周期。仅在支持web的Spring ApplicationContext上下文中有效。 |
| [application](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-scopes-application) | Scopes a single bean definition to the lifecycle of a `ServletContext`. Only valid in the context of a web-aware Spring `ApplicationContext`.                                                                           将单个bean定义作用于ServletContext的生命周期。仅在支持web的Spring ApplicationContext上下文中有效。 |
| [websocket](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#websocket-stomp-websocket-scope) | Scopes a single bean definition to the lifecycle of a `WebSocket`. Only valid in the context of a web-aware Spring `ApplicationContext`.                                                                                            将单个bean定义作用于WebSocket的生命周期。仅在支持web的Spring ApplicationContext上下文中有效。 |



##### ①单例模式

**定义：**

> ​	当定义一个bean并且其作用域为单例时，Spring IoC容器将为该bean所定义的对象创建一个且只有一个实例。 该单个实例存储在此类单例bean的高速缓存中，并且对该命名bean的所有后续请求和引用都返回该高速缓存的对象。
>

![singleton](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/images/singleton.png)

**实现：**

```xml
<!-- 单例作用域是默认的，可不加scope="singleton" -->
<bean id="accountService" class="com.something.DefaultAccountService" scope="singleton"/>

@Scope("singleton ")
```



##### ②原型模式

**定义：**

> ​	bean部署的非单例原型作用域导致每次对特定bean发出请求时都创建一个新的bean实例。应该对所有有状态bean使用原型作用域，对无状态bean使用单例作用域
>

![prototype](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/images/prototype.png)

**实现：**

```xml
<bean id="accountService" class="com.something.DefaultAccountService" scope="prototype"/>
```

**注意：**

> ​	Spring不管理原型bean的完整生命周期。容器实例化、配置和组装原型对象并将其交给客户端，而不进一步记录该原型实例。
>
> ​	当bean的生命周期不同时就会出现问题。假设单例bean A需要使用非单例(原型)bean B，可能在对A的每次方法调用上都是如此：容器只创建一次单例bean A，因此只获得一次设置属性的机会。容器不能在每次需要bean B的时候都向bean A提供一个新的实例。
>
> ​	一个解决办法是放弃一些控制反转。可以通过实现applicationcontexts taware接口，以及在每次bean A需要bean B实例时，通过对容器进行getBean(“B”)调用来让bean A接收容器。
>



##### ③其他模式

> ​	请求、会话、应用程序和websocket作用域只有在使用web感知的Spring ApplicationContext实现(如XmlWebApplicationContext)时才可用。
>





#### **（6）Bean生命周期的回调：**

> 三种初始化bean生命周期的方式：
>
> 1. 使用带有 `@PostConstruct`注解的方法
> 2. 实现 `InitializingBean` 接口的`afterPropertiesSet()` 方法
> 3. 自定义配置一个 `init()` 方法，通过`init-method="init"`属性
>
> 三种销毁bean生命周期的方式：
>
> 1. 使用带有 `@PreDestroy注解的方法`
> 2. 实现`DisposableBean` 接口的`destroy()` 方法
> 3. 自定义配置一个`destroy()`方法，通过`destroy-method="cleanup"`属性
>

##### 方式一、方式二

###### ①初始化回调函数

```java 
// 方式一 
<bean id="exampleInitBean" class="examples.ExampleBean" init-method="init"/>
    
public class ExampleBean {

    public void init() {
        // 一些初始化工作
    }
}
```

```Java
// 方式二 不建议使用
<bean id="exampleInitBean" class="examples.AnotherExampleBean"/>
    
public class AnotherExampleBean implements InitializingBean {

    @Override
    public void afterPropertiesSet() {
        // 一些初始化工作
    }
}
```

> 方式一通过`init-method="init"`属性，方式二通过`InitializingBean`接口的`afterPropertiesSet`方法，会使Spring与Java代码耦合，建议使用方式一。
>
> 建议不要使用InitializingBean接口，因为它不必要地将代码与Spring结合在一起。另外，建议使用@PostConstruct注解 或指定POJO初始化方法。在基于xml的配置元数据的情况下，可以使用init-method属性指定具有void无参方法的名称。使用Java配置，可以使用@Bean的initMethod属性。
>

###### ②销毁回调函数

```Java
//DisposableBean接口下的 destroy()，不建议使用
void destroy() throws Exception;
```

```Java
<bean id="exampleInitBean" class="examples.ExampleBean" destroy-method="cleanup"/>
    
public class ExampleBean {

    public void cleanup() {
        // do some destruction work (like releasing pooled connections)
    }
}
```

```Java
<bean id="exampleInitBean" class="examples.AnotherExampleBean"/>

public class AnotherExampleBean implements DisposableBean {

    @Override
    public void destroy() {
        // do some destruction work (like releasing pooled connections)
    }
}
```

> 建议不要使用DisposableBean回调接口，因为它不必要地将代码耦合到Spring。 另外，建议使用@PreDestroy注释或指定bean定义支持的通用方法。 使用基于XML的配置元数据时，可以在<bean/>上使用destroy-method属性。 通过Java配置，可以使用@Bean的destroyMethod属性。
>

##### 方式三

```JAVA 
public class CachingMovieLister {

    @PostConstruct
    public void populateMovieCache() {
        // 初始化时加载缓存
    }

    @PreDestroy
    public void clearMovieCache() {
        // 销毁时清除缓存
    }
}
```



















## 2.面向切面编程（aspect-oriented programming，AOP）

> **作用：**AOP能够使系统服务（例如：日志模块、安全模块、事务管理）模块化，并以声明的方式将它们应用到它们需要影响的组件中去。所造成的结果就是业务组件会具有更高的内聚性并且会更加关注自身的业务，完全不需要了解涉及系统服务所带来复杂性。
>
> **实现：**使用了Spring的aop配置命名空间把系统服务 bean声明为一个切面。首先，需要把系统服务声明为一个bean，然后 在 < aop:aspect > 元素中引用该bean。

[AOP](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop)



### 2.1 AOP概念

------

- `Aspect`切面
- `Join point`连接点（指目标对象的方法执行）

- `Pointcut`切入点（连接点的集合）

- `Target object`目标对象（原始对象）
- `AOP proxy`代理对象（增加了切面逻辑的原始对象）
- `Weaving`织入（将切面的逻辑加入到目标对象）
- `Advice`通知（切面在特定连接点上采取的操作）



### 2.2 AOP功能

----------------





### 2.3 AOP代理

--------------

> Spring AOP默认为AOP代理使用标准JDK动态代理。这允许代理任何接口(或一组接口)。
>
> Spring AOP也可以使用CGLIB代理。这对于代理类而不是接口是必要的。默认情况下，如果业务对象没有实现接口，则使用CGLIB。
>
> JDK源码中动态代理已经继承了一个Proxy，Java不能多继承，所以不能继承目标对象，只能实现目标对象的接口。CGLIB代理（基于继承）情况下目标对象等于代理对象，JDK动态代理（基于接口）情况下目标对象不等于代理对象。（但是目标对象和代理对象等于他俩共同拥有的接口）



### 2.4 @AspectJ支持

--------

> AspectJ是一个面向切面的框架，它扩展了Java语言。AspectJ定义了AOP语法。Spring Aop只是借助了AspectJ的AOP语法（注解）。
>



#### 2.4.1 启用@AspectJ支持

---------------

##### （1）Java Configuration方式启用@AspectJ支持

`@EnableAspectJAutoProxy`

```Java
@Configuration
@EnableAspectJAutoProxy
public class AppConfig {

}
```



##### （2）XML Configuration方式启用@AspectJ支持

`aop:aspectj-autoproxy`

```xml
<aop:aspectj-autoproxy/>
```



#### 2.4.2 声明一个Aspect切面

---------------

① 定义一个bean

② 使用@Aspectj注解

```Java
import org.aspectj.lang.annotation.Aspect;

@Component
@Aspect
public class AspectDemo {

}
```



#### 2.4.3 声明一个Pointcut切入点

--------

> 切入点签名返回值类型必须是**void**

```Java
import org.aspectj.lang.annotation.Aspect;

@Component
@Aspect
public class AspectDemo {
    @Pointcut("execution(* transfer(..))") // 切入点表达式（切入点指示器+声明需要执行的方法）
    private void anyOldTransfer() {} // 切入点签名（访问修饰符+void+方法名+任何参数）
}
```



##### ① 切入点指示器pointcut designators (PCD) 

- `execution`: 主要的切入点指示器。

```Java
// The execution of any public method:
execution(public * *(..))

// The execution of any method with a name that begins with set:
execution(* set*(..))

// The execution of any method defined by the AccountService interface:
execution(* com.xyz.service.AccountService.*(..))

// The execution of any method defined in the service package:
execution(* com.xyz.service.*.*(..))

// The execution of any method defined in the service package or one of its sub-packages:
execution(* com.xyz.service..*.*(..))
```

- `within`: 与execution相比只能限制到类，起辅助作用。

```Java
// Any join point (method execution only in Spring AOP) within the service package:
within(com.xyz.service.*)

// Any join point (method execution only in Spring AOP) within the service package or one of its sub-packages:
within(com.xyz.service..*)
```

- `this`: 限制代理对象（目标对象经过JDK动态代理或CGLIB代理变为代理对象）

```Java
// Any join point (method execution only in Spring AOP) where the proxy implements the AccountService interface:
this(com.xyz.service.AccountService)
```

- `target`: 限制目标对象

```Java
// Any join point (method execution only in Spring AOP) where the target object implements the AccountService interface:
target(com.xyz.service.AccountService)
```

- `args`: 只限制参数类型。

```Java
// Any join point (method execution only in Spring AOP) that takes a single parameter and where the argument passed at runtime is Serializable:
args(java.io.Serializable)
```

- `@target`: Limits matching to join points (the execution of methods when using Spring AOP) where the class of the executing object has an annotation of the given type.

```Java
// Any join point (method execution only in Spring AOP) where the target object has a @Transactional annotation:
@target(org.springframework.transaction.annotation.Transactional)
```

- `@args`: 限制传递的实际参数的运行时类型具有给定类型的注解。

```Java
// Any join point (method execution only in Spring AOP) which takes a single parameter, and where the runtime type of the argument passed has the @Classified annotation:

@args(com.xyz.security.Classified)
```

- `@within`: 限制类是否有给定的注解。

```Java
// Any join point (method execution only in Spring AOP) where the declared type of the target object has an @Transactional annotation:
@within(org.springframework.transaction.annotation.Transactional)
```

- `@annotation`: 将匹配限制为具有给定注解的连接点。

```Java
// Any join point (method execution only in Spring AOP) where the executing method has an @Transactional annotation:

@annotation(org.springframework.transaction.annotation.Transactional)
```



##### ② 声明需要执行的方法

> **切入点指示器的格式：**
>
> ？表示可要可不要
>
> ```Java
> execution(modifiers-pattern? ret-type-pattern declaring-type-pattern?name-pattern(param-pattern) throws-pattern?)
> ```
>
> **例子：**
>
> ```Java
> * com.xyz.service.AccountService.*(..)
> ```
>
> > - modifiers-pattern：方法访问修饰符，如public，protected
> > - ret-type-pattern：方法返回值类型 ，如void，int
> > - declaring-type-pattern：方法所在类的全路径名，如com.xyz.service
> > - name-pattern：方法名类型 ，如AccountService
> > - param-pattern：方法的参数类型，如java.lang.String，(..)
> > - throws-pattern：方法抛出的异常类型，如java.lang.Exception



##### ③ 切入点表达式其他方法

> 使用`&&` `||` `!`组合表达式，并且通过名称引入表达式。

```Java
@Pointcut("execution(public * *(..))")
private void anyPublicOperation() {} 

@Pointcut("within(com.xyz.myapp.trading..*)")
private void inTrading() {} 

@Pointcut("anyPublicOperation() && inTrading()")
private void tradingOperation() {} 
```



#### 2.4.4 声明通知Advice

-----------

> 通知与切入点表达式相关联，并在切入点匹配的方法执行之前、之后或周围运行。



##### ① 前置通知 `@Before`

```Java
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;

@Aspect
public class BeforeExample {

    @Before("execution(* com.xyz.myapp.dao.*.*(..))")
    public void doAccessCheck() {
        // ...
    }

}
```



##### ② 后置通知 `@AfterReturning`

> returning属性中使用的名称必须与通知方法中的参数名称对应。还将匹配限制为只匹配那些返回指定类型值的方法执行，(在本例中是Object，它匹配任何返回值)。

```Java
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.AfterReturning;

@Aspect
public class AfterReturningExample {

    @AfterReturning(
        pointcut="com.xyz.myapp.CommonPointcuts.dataAccessOperation()",
        returning="retVal")
    public void doAccessCheck(Object retVal) {
        // ...
    }

}
```



##### ③ 抛出异常后通知 `@AfterThrowing`

> throwing属性中使用的名称必须与通知方法中的参数名称对应。当方法执行通过抛出异常而退出时，异常将作为相应的参数值传递给通知方法。并限制指定类型异常，本例为DataAccessException

```Java
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.AfterThrowing;

@Aspect
public class AfterThrowingExample {

    @AfterThrowing(
        pointcut="com.xyz.myapp.CommonPointcuts.dataAccessOperation()",
        throwing="ex")
    public void doRecoveryActions(DataAccessException ex) {
        // ...
    }

}
```



##### ④ 后置最终通知 `@After`

```Java
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.After;

@Aspect
public class AfterFinallyExample {

    @After("com.xyz.myapp.CommonPointcuts.dataAccessOperation()")
    public void doReleaseLock() {
        // ...
    }

}
```



##### ⑤ 环绕通知 `@Around`

> **ProceedingJoinPoint** extends JoinPoint 正在增强（执行）的连接点（方法）
>
> JoinPoint 连接点，有getThis()，getTarget()等方法。
>
> 对ProceedingJoinPoint调用proceed()会导致底层方法运行。proceed方法还可以传入一个对象[]。数组中的值用作方法执行时的参数。

```Java
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.ProceedingJoinPoint;

@Aspect
public class AroundExample {

    @Around("com.xyz.myapp.CommonPointcuts.businessService()")
    public Object doBasicProfiling(ProceedingJoinPoint pjp) throws Throwable {
        // 开始
        Object retVal = pjp.proceed();
        // 结束
        return retVal;
    }

}
```



#### 2.4.5 引入

> 声明对象实现给定的接口,并提供该接口的一个实现代表的对象。通过`@DeclareParents`实现。
>
> 本例中给定一个名为UsageTracked的接口和该接口的一个名为DefaultUsageTracked的实现，之后实现UsageTracked接口的实现类默认调用DefaultUsageTracked中已实现的方法。

```Java
@Aspect
public class UsageTracking {

    @DeclareParents(value="com.xzy.myapp.service.*+",defaultImpl=DefaultUsageTracked.class)
    public static UsageTracked mixin;

    @Before("com.xyz.myapp.CommonPointcuts.businessService() && this(usageTracked)")
    public void recordUsage(UsageTracked usageTracked) {
        usageTracked.incrementUseCount();
    }

}
```



#### 2.4.6 切面实例化模型

> `perthis`  
>
> `pertarget`

```Java
@Aspect("perthis(com.xyz.myapp.CommonPointcuts.businessService())")
public class MyAspect {

    private int someState;

    @Before("com.xyz.myapp.CommonPointcuts.businessService()")
    public void recordServiceUsage() {
        // ...
    }

}
```



































```
##### 3.Spring容器*

*对象生存于Spring容器（container）中。Spring容器负责创建对象，装配它们，配置它们并管理它们的整个生命周期，从生存到死亡。*

***容器实现：***

- *~~bean工厂~~*
- *应用上下文* 

1. *AnnotationConfigApplicationContext：从一个或多个基于Java的配置类中加载Spring应用上下文。* 
2. *AnnotationConfigWebApplicationContext：从一个或多个基于Java的配置类中加载Spring Web应用上下文。* 
3. *ClassPathXmlApplicationContext：从类路径下的一个或多个XML配置文件中加载上下文定义，把应用上下文的定义文件作为类 资源。*
4. *FileSystemXmlapplicationcontext：从文件系统下的一个或多个XML配置文件中加载上下文定义。* 
5. *XmlWebApplicationContext：从Web应用下的一个或多个XML配置文件中加载上下文定义。* 

##### *4.bean生命周期*
```

