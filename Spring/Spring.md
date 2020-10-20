# Spring

[官方文档](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-dependencies)

控制反转（Inversion of Control，IoC），是面向对象编程中的一种设计原则，降低代码之间的耦合度。最常见的实现方式叫做依赖注入（Dependency Injection，DI），还有依赖查找（Dependency Lookup）



- **依赖：**

  a类中的某个属性是其他类，或构造方法中传入了其他类，就叫a依赖了这个类

- **spring实现IOC：**

1. 应用程序中提供类，提供依赖关系（属性或者构造方法）
2. 把需要交给容器管理的对象通过配置信息告诉容器（xml、annotation(注解)，java Configuration）
3. 把各个类之间的依赖关系通过配置信息告诉容器







































------

- IoC和DI的区别？

  IoC控制反转是我们实现代码低耦合的目标，DI依赖注入是IoC的主要实现方式。
  
- 

