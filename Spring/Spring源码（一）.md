# Spring循环依赖

-------------------

[TOC]

<img src="C:\Users\李佳庆\Desktop\APP\笔记本\图片库\640-1602662423724.webp" alt="img" style="zoom: 40%;" />

> 如图，类之间的相互引用就是循环依赖，spring是允许这样的循环依赖(前提是单例的,非构造方法注入的情况下)

*原型模式时每次注入bean（例如B）都会创建一个新的bean实例，创建B的实例又会注入bean（C），再创建C的实例则又要创建A的实例。所以原型模式使用循环依赖会直接抛出BeanCurrentlyInCreationException*







## Spring解决循环依赖

首先，Spring内部维护了三个Map，也就是我们通常说的三级缓存。

- *singletonObjects* 俗称“单例池”“容器”，缓存创建完成单例Bean的地方。
- *singletonFactories* 映射创建Bean的原始工厂
- *earlySingletonObjects* 映射Bean的早期引用，也就是说在这个Map里的Bean不是完整的，甚至还不能称之为“Bean”，只是一个Instance.

![img](C:\Users\李佳庆\Desktop\APP\笔记本\图片库\640.gif)

所谓的循环依赖其实无非就是属性注入，或者自动注入， 故而搞明白循环依赖就需要去研究spring自动注入的源码；spring的属性注入属于spring bean的生命周期一部分。

理解Spring生命周期：

> 1. **spring bean**——受spring容器管理的对象，可能经过了完整的spring bean生命周期，最终存在spring容器当中；一个bean一定是个对象
> 2. **对象**——任何符合java语法规则实例化出来的对象，但是一个对象并不一定是spring bean；



### Spring生命周期

Class--------》beanDefinition----------》bean	与普通对象不同的是bean对象要经过

首先，Java虚拟机将类的class文件加载到方法区，然后扫描是否有注入bean的操作。 对于要注入的，spring会new一个`BeanDefination`接口（这个接口包括各种对bean的属性判断方法，例如`GenericBeanDefinition`）的实现类，然后将实现类对象即bean的各种属性（属性例如：beanClassName="A"， beanClass=A.class，scope="singleton"，...）put到`BeanDefinitionMap`的beanDefinitionMap中去，通过finishiBeanFactoryInitialization(beanFactory)方法获取beanDefinitionMap中的beanName开始验证，验证成功后new一个bean对象放到spring单例池`SingletonObjects`（map类型）中，【注意：原型对象不是在spring初始化的时候new一个bean，而是在getBean的时候】bean对象到单例池之前还可以调用`BeanFactoryPostProcessor`接口的实现类进行扩展。

具体步骤如下：

#### 1.扫描：扫描是否有bean注入

例如通过xml方式，JavaConfig方式或注解方式的bean注入。

> ```Java
> public AnnotationConfigApplicationContext(Class<?>... annotatedClasses) {
>     this();
>     register(annotatedClasses);//注册配置类
>     refresh();// 真正初始化bean的方法
> }
> ```
>
> 

#### 2.解析：解析是不是单例模式，是否懒加载...

在一个for循环中 {

​		new一个`BeanDefinition`类的子类`GenericBeanDefinition`对象genericBeanDefinition，这个对象包括setBeanClass，setBeanClassName，setScope...等方法。

​		定义一个`BeanDefinitionMap`的beanDefinitionMap，将`GenericBeanDefinition`对象put到beanDefinitionMap<"xxx",genericBeanDefinition>中。

​		此外还会定义一个List<String>的beanNames，beanNames.add(xxx)，负责存储beanDefinitionMap中的key，用于遍历`GenericBeanDefinition`对象。

}

#### 3.调用扩展：查看是否赋予额外的扩展功能

如何扩展：

创建一个类实现`BeanFactoryPostProcessor`接口，继承postProcessBeanFactory方法，方法会传一个beanFactory（就是Spring工厂，Spring工厂中就包括了上一步定义的beanDefinitionMap），通过beanFactory调用getBeanDefinition获取beanDefinitionMap中的BeanDefinition对象。

```Java
// 扩展实现，更改beanClass
@Component	//加上才会扫描到，否则不起作用
public class test implements BeanFactoryPostProcessor {
    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
        GenericBeanDefinition bd = beanFactory.getBeanDefinition("A");
        // 这里写一些扩展功能，比如更改beanName，禁止循环依赖...
        bd.setBeanClass(B.Class);
    }
}
```



```Java
// getBeanDefinition内部方法
@Override
public BeanDefinition getBeanDefinition(String beanName) throws NoSuchBeanDefinitionException {
    // 在这里获取map中的beanDefinition对象，beanName未声明则默认为自动驼峰命名。
    BeanDefinition bd = this.beanDefinitionMap.get(beanName);
    if (bd == null) {
        if (this.logger.isTraceEnabled()) {
            this.logger.trace("No bean named '" + beanName + "' found in " + this);
        }
        throw new NoSuchBeanDefinitionException(beanName);
    }
    return bd;
}
```



#### 4.验证：根据第二步解析获得的属性判断要不要new一个Bean

具体判断bean是不是原型的；装配模式是byType还是byName；构造方法有没有参数，参数合不合理；

合理就实例化Bean：new一个 Bean对象到Spring的单例池`SingletonObjects`（Map类型）中。


> 验证代码
>
> **finishBeanFactoryInitialization**
>
> ```Java
> protected void finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory) {
> // 。。。
> // Instantiate all remaining (non-lazy-init) singletons.
> beanFactory.preInstantiateSingletons();
> }
> ```
>
> **preInstantiateSingletons**
>
> ```Java
> public void preInstantiateSingletons() throws BeansException {
> 		//。。。
>     	// 取出beanDefinitionMap存放的beanNames
> 		List<String> beanNames = new ArrayList<>(this.beanDefinitionNames);
> 
> 		// 遍历并验证beanNames,进行实例化
> 		for (String beanName : beanNames) {
>             // 验证是不是单例、懒加载。。。
> 			if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
> 				//。。。
>                 //实例化方法
>                  getBean(beanName);
>                  //。。。
> 			}
> 		}
> 	}
> ```

> 说明
>
> ```Java
> // 实例化所有还剩下的、非懒加载的单例对象
> finishiBeanFactoryInitialization(beanFactory);
> // 内部继续调用preInstantiateSingletons方法
> beanFactort.preInstantiateSingletons();
> // 获取第二步存入List的beanNames，进行遍历验证，例如是不是单例，是否懒加载...等，验证之后开始实例化
> getBean(beanName);
> // 内部调用doGetBean方法
> ```

#### 5.实例化：实例化Bean

doGetBean

1.验证beanName是否合法

```
final String beanName = transformedBeanName(name);
```

2.getSingleton(beanName);Spring从单例池`SingletonObjects`中尝试取出这个bean，判断是否已经实例化

```
Object sharedInstance = getSingleton(beanName);
```

> **getBean**
>
> ```Java
> public Object getBean(String name) throws BeansException {
> 	return doGetBean(name, null, null, false);
> }
> ```
>
> **doGetBean**
>
> ```Java
> protected <T> T doGetBean(final String name, 。。。) throws BeansException {
> 	// 验证beanName合法性
>     final String beanName = transformedBeanName(name);
>     // 检查单例池中此bean是否已经手动生成
>     Object sharedInstance = getSingleton(beanName);
>     // 若单例池中没有此bean，
>     if (sharedInstance != null && args == null) {...}
>     else {
>         // 判断这个类是不是在创建过程中,是则抛出异常
>         if (isPrototypeCurrentlyInCreation(beanName)) {
>             throw new BeanCurrentlyInCreationException(beanName);
>         }
>     }
>     
>     //继续进行一系列验证...
>     
>     // 如果是单例
>     if (mbd.isSingleton()) {
>         //...
>         // bean开始创建
>     	return createBean(beanName, mbd, args);
>     }
>     //...
> }
> ```
>
> **getSingleton**
>
> ```Java
> /** 单例对象缓存/单例池（beanName->bean实例）: bean name --> bean instance */
> private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);
> 
> protected Object getSingleton(String beanName, boolean allowEarlyReference) {
>     Object singletonObject = this.singletonObjects.get(beanName);
>     //。。。
>     return singletonObject;
> }
> ```
>
> **createBean**
>
> ```Java
> protected Object createBean(String beanName, RootBeanDefinition mbd, ...)throws BeanCreationException {
>     //...
>     RootBeanDefinition mbdToUse = mbd;
>     //...
>     // 第一次调用后置处理器，允许用代理对象替换目标对象
>     Object bean = resolveBeforeInstantiation(beanName, mbdToUse);
>     // ...
>     // 进入doCreateBean方法创建bean实例
>     Object beanInstance = doCreateBean(beanName, mbdToUse, args);
> }
> ```
>
> **doCreateBean**
>
> ```Java
> protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final @Nullable Object[] args)throws BeanCreationException {
>     // 实例化bean
>     BeanWrapper instanceWrapper = null;
>     // 单例的，则从bean工厂里移除
>     if (mbd.isSingleton()) {
>         instanceWrapper = this.factoryBeanInstanceCache.remove(beanName);
>     }
>     if (instanceWrapper == null) {
>         // 实例化对象，createBeanInstance里第二次调用后置处理器（通过构造方法反射实例化对象）
>         instanceWrapper = createBeanInstance(beanName, mbd, args);
>         //。。。
>         // 第三次后置处理器，修改合并的bean定义
>         applyMergedBeanDefinitionPostProcessors(mbd, beanType, beanName);
>         //。。。
>         // 是否允许循环依赖（单例的&&允许循环依赖&&正在创建）【allowCircularReferences默认为true】
>         // 可以通过setAllowCircularReferences(false);设置不支持循环依赖
>         boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&isSingletonCurrentlyInCreation(beanName));
>         if (earlySingletonExposure) {
> 			//。。。
>             // 第四次调用后置处理器，判断是否需要AOP
> 			addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
> 		}
>         // 初始化bean实例
>         Object exposedObject = bean;
> 		try {
>             // 填充属性，即自动注入。内部完成第五，第六次后置处理器调用
> 			populateBean(beanName, mbd, instanceWrapper);
>             // 初始化Spring。内部完成第七，第八次后置处理器调用
> 			exposedObject = initializeBean(beanName, exposedObject, mbd);
> 		}
>     }
> }
> ```
>
> 



































































# ============================================

--------------------

- **Spring是如何默认支持循环依赖的？**

Spring在生命周期中的(getBean--doGetBean--createBean--doCreateBean方法中)实例化bean中会判断当前Spring容器是否允许循环依赖，是由属性**allowCircularReferences**（默认为true：开启循环依赖）、是否单例、是否正在创建判断的。如果允许，则会第四次调用后置处理器

可以手动调用refresh()方法，并在此之前使用setAllowCircularReferences(false)关闭循环依赖。

也可以通过扩展功能关闭循环依赖。（如何扩展看标题3）

