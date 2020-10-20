# 设计模式------Proxy

[链接：AOP原理之动态代理](https://blog.csdn.net/litianxiang_kaola/article/details/85335700)

## 什么是代理?

增强一个对象的功能

> 买火车票，app就是一个代理,代理了火车站的售票处



## Java中如何实现代理?

> Java实现代理的两种办法：静态代理和动态代理



### 代理的名词

代理对象：	增强后的对象

目标对象：	被增强的对象

> 他们的身份不是绝对的，会根据情况发生变化



### 静态代理

#### 继承

代理对象继承目标对象，重写需要增强的方法。

缺点：代理类过多，复杂

```Java
public class LogExtendsTime extends Time {
    @Override
    public void query(){
        /** ... 一些log相关的代理方法 */
        super.query();
    }
}
```



#### 聚合

目标对象和代理对象实现同一个接口，代理对象当中要包含目标对象。

缺点：也会产生类爆炸，只不过比继承少

```Java
public class Test {
    public static void main(String[] args) {
        // 实现了本来的功能 + log
        Service target = new LogService(new ServiceImpl);
        // 又添加了time功能 
        Service proxy = new TimeService(target);
        proxy.query();
    }
}
```



> 总结：如果在不确定的情况下，尽量不要使用静态代理。



### 动态代理

> 静态代理会为每一个业务增强都提供一个代理类, 由代理类来创建代理对象, 而动态代理并不存在代理类, 代理对象直接由代理生成工具动态生成.



#### JDK动态代理

> JDK动态代理是使用 java.lang.reflect 包下的代理类来实现. JDK动态代理动态代理必须要有接口.

```Java
public class Client {
    public static void main(String[] args) {
        //创建目标对象
        Service target = new LogService();
        //创建代理对象
        Service proxy = (Service) Proxy.newProxyInstance(target.getClass().getClassLoader(),
                target.getClass().getInterfaces(),
                new TimeService(target)
        );
        proxy.query();
    }
}
```



#### CGLIB动态代理

> JDK动态代理必须要有接口, 但如果要代理一个没有接口的类该怎么办呢? 这时我们可以使用CGLIB动态代理. CGLIB动态代理的原理是生成目标类的子类, 这个子类对象就是代理对象, 代理对象是被增强过的.

> 注意: 不管有没有接口都可以使用CGLIB动态代理, 而不是只有在无接口的情况下才能使用.

```Java
public class Client {
    public static void main(String[] args) {
        //创建目标对象
        LogService target = new LogService();
        //
        //创建代理对象
        LogService proxy = (LogService) Enhancer.create(target.getClass(),
                new TimeSeervice());
        proxy.query();
    }
}
```

