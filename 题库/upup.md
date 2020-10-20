# upup

[TOC]



### 1 网络

#### 1.1 COOKIE和SESSION有什么区别？

  [cookie/session详解](https://www.zhihu.com/question/19786827)

**Session**(会话)**是在服务端保存的一种数据结构，用来标记用户、跟踪用户的状态，这个数据可以保存在集群，数据库，文件中。**

**Cookie**(小甜点:记录账号，给用户甜头)**是客户端保存用户信息的一种机制，用来记录用户的一些信息,也是实现Session的一种方式，创建Session时需要在Cookie里面记录一个Session ID。**

```
1. 由于HTTP协议是无状态的协议，所以服务端需要记录用户的状态时，就需要用某种机制来识具体的用户，这个机制就是Session.典型的场景比如购物车，当你点击下单按钮时，由于HTTP协议无状态，所以并不知道是哪个用户操作的，所以服务端要为特定的用户创建了特定的Session，用用于标识这个用户，并且跟踪用户，这样才知道购物车里面有几本书。这个Session是保存在服务端的，有一个唯一标识。在服务端保存Session的方法很多，内存、数据库、文件都有。集群的时候也要考虑Session的转移，在大型的网站，一般会有专门的Session服务器集群，用来保存用户会话，这个时候 Session 信息都是放在内存的，使用一些缓存服务比如Memcached之类的来放 Session。
2. 思考一下服务端如何识别特定的客户？这个时候Cookie就登场了。每次HTTP请求的时候，客户端都会发送相应的Cookie信息到服务端。实际上大多数的应用都是用 Cookie 来实现Session跟踪的，第一次创建Session的时候，服务端会在HTTP协议中告诉客户端，需要在 Cookie 里面记录一个Session ID，以后每次请求把这个会话ID发送到服务器，我就知道你是谁了。有人问，如果客户端的浏览器禁用了 Cookie 怎么办？一般这种情况下，会使用一种叫做URL重写的技术来进行会话跟踪，即每次HTTP交互，URL后面都会被附加上一个诸如 sid=xxxxx 这样的参数，服务端据此来识别用户。
3. Cookie其实还可以用在一些方便用户的场景下，设想你某次登陆过一个网站，下次登录的时候不想再次输入账号了，怎么办？这个信息可以写到Cookie里面，访问网站的时候，网站页面的脚本可以读取这个信息，就自动帮你把用户名给填了，能够方便一下用户。这也是Cookie名称的由来，给用户的一点甜头。

所以，总结一下：Session是在服务端保存的一个数据结构，用来跟踪用户的状态，这个数据可以保存在集群、数据库、文件中；Cookie是客户端保存用户信息的一种机制，用来记录用户的一些信息，也是实现Session的一种方式。
```

#### 1.2 怎么获取Cookie/Session

**获取cookie：**

​	**1.document.cookie**

​	**2.Network---Name---Resquest Headers---Cookie**

**获取session：**

​	**1.request.getSession()**.getAttribute("xxx").toString();

#### 1.3 服务端获取客户端请求的真实IP

**1.JSP中获取客户端IP地址：request.getRemoteAddr()**

```
客户端请求信息都包含在HttpServletRequest中，可以通过方法getRemoteAddr()获得该客户端IP。此时如果未经过代理，直接访问服务器端，可以直接获得该客户端真实IP。而如果是通过代理的形式，此时经过多级反向的代理，通过方法getRemoteAddr()得不到客户端真实IP，可以通过x-forwarded-for获得转发后请求信息。当客户端请求被转发，IP将会追加在其后并以逗号隔开，例如：10.47.103.13,4.2.2.2,10.96.112.230。
```

**2.request.getHeader("x-forwarded-for");**

```
X-Forwarded-For: client1, proxy1, proxy2, proxy3

其中的值通过一个 逗号+空格 把多个IP地址区分开, 最左边(client1)是最原始客户端的IP地址, 代理服务器每成功收到一个请求，就把请求来源IP地址添加到右边。 在上面这个例子中，这个请求成功通过了三台代理服务器：proxy1, proxy2 及 proxy3。请求由client1发出，到达了proxy3(proxy3可能是请求的终点)。请求刚从client1中发出时，XFF是空的，请求被发往proxy1；通过proxy1的时候，client1被添加到XFF中，之后请求被发往proxy2;通过proxy2的时候，proxy1被添加到XFF中，之后请求被发往proxy3；通过proxy3时，proxy2被添加到XFF中，之后请求的的去向不明，如果proxy3不是请求终点，请求会被继续转发。

鉴于伪造这一字段非常容易，应该谨慎使用X-Forwarded-For字段。正常情况下XFF中最后一个IP地址是最后一个代理服务器的IP地址, 这通常是一个比较可靠的信息来源。
```

####  1.4 客户端浏览器一次http完整请求过程

[htpp请求详解](https://blog.csdn.net/qq_40959677/article/details/94873075?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.nonecase&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.nonecase)

![img](https://img-blog.csdnimg.cn/20190706170610596.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwOTU5Njc3,size_16,color_FFFFFF,t_70)**①：DNS解析域名得到IP地址**
**②：客户端与服务器建立连接(TCP三次握手)**
**③：客户端发起请求**
**④：服务器接收到请求根据端口号.路径等找到对应资源文件，响应源代码给客户端**
**⑤：客户端拿到请求到的数据(html页面的源代码)，开始解析页面以及请求资源**
**⑥：客户端渲染页面**
**⑦：web服务器断开连接(四次挥手)**

#### 1.5 URI的组成

URI由URL和URN组成。

**URI：统一资源标识符**

**URL：统一资源定位符**

**URN：统一资源名称**

![img](https://img-blog.csdnimg.cn/20190706171214678.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwOTU5Njc3,size_16,color_FFFFFF,t_70)

#### 1.6 TCP三次握手 

[三次握手](https://blog.csdn.net/Geek_Alex/article/details/105989166?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-3.nonecase&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-3.nonecase)

**第一次握手：客户端发送一个带 SYN(建立联机)=1，Seq(顺序号码)=X 的数据包到服务器端口** 
		**第二次握手：服务器发回一个带 SYN=1，ACK(确认)=1, ack(确认号码)=X+1， Seq=Y 的响应包以示传达确认信息** 
		**第三次握手：客户端再回传一个带 ACK=1,   ack=Y+1， Seq=X+1 的数据包，代表“握手结束”**

<img src="https://img-blog.csdnimg.cn/20190706171514339.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwOTU5Njc3,size_16,color_FFFFFF,t_70" alt="img"  />

<img src="https://tva1.sinaimg.cn/large/007S8ZIlly1gek0vnrigmj30p21000vs.jpg" alt="img" style="zoom:50%;" />



### 2 数据库

------



#### 2.1 MySQL

##### 2.1.1 MySQL 内、左、右、全、交叉、自连接  

[MySQL连接详解](https://www.cnblogs.com/zeroingToOne/p/9575203.html)

**内连接（inner join on）**：返回两个表的交集。

**左连接（left join on / left outer join on）**：返回左表全部记录与右表符合条件的记录，右表不符合条件的均为NULL。当两个表的记录行数不同时，以左表为准。

**全连接（Oracle：join on）**：返回左右表的所有行，哪个表中没有的就用null填充。MySQL不支持全外连接，所以只能采取关键字UNION来联合左、右连接。

**交叉连接（cross join on）**：没有where条件的交叉连接中，产生连接表就是笛卡尔积。将两个表的所有行进行组合，连接后的行数为两个表的行数的乘积数。

**自连接**：把一张表看作两张表用，取两个表别名。常用在地市的上一级地市级别、员工的上一级领导。



##### 2.1.2 select * 和select 全部字段的区别

1. **可读性问题：**时间相差不多，指定全部字段能更清楚返回了哪些列。
2. **网络IO问题： **select * 需要数据库先查询表中列的元数据，稍微影响网络传输的性能。
3. **覆盖索引问题：**select 指定字段有索引：mysql可以不用读data，直接使用index里面的值就返回结果；                                  			               select *：             在读完index以后还需要去读data才会返回结果。



##### 2.1.3 in()和exists()的区别

[https://blog.csdn.net/lick4050312/article/details/4476333](https://blog.csdn.net/lick4050312/article/details/4476333)

**答:  IN适合于外表大而内表小的情况；EXISTS适合于外表小而内表大的情况。**

```
select * from A
where id in(select id from B)

select a.* from A a
where exists(select 1 from B b where a.id=b.id)
```

因为IN是先查询子查询的表，然后将内表和外表做一个笛卡尔积，再按照条件进行筛选，最多可能遍历（外表x内表）次；EXISTS会执行（外表的大小）次，因为不缓存exist的结果集，只判断结果集中是否有记录，返回Boolean值。所以in适合内表比外表小；exist适合外表比内表小。



```
1、适用表的类型不同。

in是子查询为驱动表，外面的表为被驱动表，故适用于子查询结果集小而外面的表结果集大的情况。

exists是外面的表位驱动表，子查询里面的表为被驱动表，故适用于外面的表结果集小而子查询结果集大的情况。

2、子查询关联不同。
exists一般都是关联子查询。对于关联子查询，必须先执行外层查询，接着对所有通过过滤条件的记录，执行内层查询。外层查询和内层查询相互依赖，因为外层查询会把数据传递给内层查询。

in则一般都是非关联子查询，非关联子查询则必须先完成内层查询之后，外层查询才能介入。



3、执行次数不同。

IN 语句：只执行一次，确定给定的值是否与子查询或列表中的值相匹配。in在查询的时候，首先查询子查询的表，然后将内表和外表做一个笛卡尔积，然后按照条件进行筛选。所以相对内表比较小的时候，in的速度较快。

EXISTS语句：执行次数根据表的长度而定。指定一个子查询，检测行的存在。遍历循环外表，然后看外表中的记录有没有和内表的数据一样的。匹配上就将结果放入结果集中。
```



### 3 Java基础

------

| 访问修饰符 | 本类 | 同包 | 子类 | 其他 |
| ---------- | ---- | ---- | ---- | ---- |
| private    | √    |      |      |      |
| 默认       | √    | √    |      |      |
| protect    | √    | √    | √    |      |
| public     | √    | √    | √    | √    |





```
为什么用List list=new ArrayList();?
-----------------------------------------------------------------------------------------
==Program to an interface, not to a concrete implementation
==对接口编程，而不是对具体实现编程

答：
将代码与接口的特定实现解耦。这还可以帮助您将来转移到List接口的另一个实现。例如:List<String> names = new ArrayList<String>();之后你决定使用其他一些列表接口的实现，比如LinkedList你只需将它改为List<String> names = new LinkedList<String>();而对list的调用均无需改变。

list.add();
在编译阶段，只是检查参数的引用类型。(List中有add方法就正确)
然而在运行时，Java 虚拟机(JVM)指定对象的类型并且运行该对象的方法。(对象类型ArrayList有add)
向上转型时，子类单独定义的方法会丢失。

List list = new ArrayList(); 这句创建了一个 ArrayList 的对象后把上溯到了 List。此时它是一个 List 对象了，有些 ArrayList 中拥有，但是 List 没有的属性和方法，它就不能再用了。
或者说： 如果 List 与 ArrayList中有相同的属性(如int i)，有相同的方法(如void f()), 则a.i是调用了List中的i,a.f() 是调用 ArrayList中重写了的 f();


什么是检查性异常与分检查性异常？
答：
非检查性异常（运行时异常）RuntimeException：空指针异常(NullPointerException)、除零异常(ArithmeticException)、数组越界异常(ArrayIndexOutOfBoundsException)等;
检查性异常（继承自Exception除RuntimeException以外的异常）（所有异常都继承自Exception）：输入输出异常(IOException)、文件不存在异常(FileNotFoundException)、SQL语句异常(SQLException)等。
```

##### 3.1 Java重载与重写 

 [重写与重载](https://blog.csdn.net/geekmubai/article/details/81975990)

- (1)方法重载是一个类中定义了多个方法名相同,而他们的参数的数量不同或数量相同而类型和次序不同,则称为方法的重载(Overloading)。
- (2)方法重写是在子类存在方法与父类的方法的名字相同,而且参数的个数与类型一样,返回值也一样的方法,就称为重写(Overriding)。
- (3)方法重载是一个类的多态性表现,而方法重写是子类与父类的一种多态性表现。

|          | 重载       | 重写                                           |
| -------- | ---------- | ---------------------------------------------- |
| 参数列表 | 必须修改   | 不能修改                                       |
| 返回类型 | 任意       | 不能修改（派生类可以）                         |
| 异常     | 任意       | 可以减少或删除，一定不能抛出新的或者更广的异常 |
| 修饰符   | 任意       | 一定不能做更严格的限制（可以降低限制）         |
| 范围     | 同一个类中 | 有继承关系的子类中                             |

```
在Java程序中，类的继承关系可以产生一个子类，子类继承父类，它具备了父类所有的特征，继承了父类所有的方法和变量。

子类可以定义新的特征，当子类需要修改父类的一些方法进行扩展，增大功能，程序设计者常常把这样的一种操作方法称为重写，也叫称为覆写或覆盖。

重写体现了Java优越性，重写是建立在继承关系上，它使语言结构更加丰富。在Java中的继承中，子类既可以隐藏和访问父类的方法，也可以覆盖继承父类的方法。

在Java中覆盖继承父类的方法就是通过方法的重写来实现的。所谓方法的重写是指子类中的方法与父类中继承的方法有完全相同的返回值类型、方法名、参数个数以及参数类型。

这样，就可以实现对父类方法的覆盖。如果子类将父类中的方法重写了，调用的时候肯定是调用被重写过的方法，那么如果现在一定要调用父类中的方法该怎么办呢?

此时，通过使用super关键就可以实现这个功能，super关键字可以从子类访问父类中的内容，如果要访问被重写过的方法，使用“super.方法名(参数列表)”的形式调用。

如果要使用super关键字不一定非要在方法重写之后使用，也可以明确地表示某个方法是从父类中继承而来的。使用super只是更加明确的说，要从父类中查找，就不在子类查找了。

重写的好处在于子类可以根据需要，定义特定于自己的行为。
也就是说子类能够根据需要实现父类的方法。
在面向对象原则里，重写意味着可以重写任何现有方法。
-------------------------------------------------------------------------------------
方法重载是让类以统一的方式处理不同类型数据的一种手段。调用方法时通过传递给它们的不同个数和类型的参数来决定具体使用哪个方法，这就是多态性。

所谓方法重载是指在一个类中，多个方法的方法名相同，但是参数列表不同。参数列表不同指的是参数个数、参数类型或者参数的顺序不同。

方法的重载在实际应用中也会经常用到。不仅是一般的方法，构造方法也可以重载。

在方法重载时，方法之间需要存在一定的联系，因为这样可以提高程序的可读性，一般只重载功能相似的方法。

重载是指我们可以定义一些名称相同的方法，通过定义不同的参数来区分这些方法，然后再调用时，Java虚拟机就会根据不同的参数列表来选择合适的方法执行。也就是说，当一个重载方法被调用时，Java用参数的类型或个数来决定实际调用的重载方法。因此，每个重载方法的参数的类型或个数必须是不同。

虽然每个重载方法可以有不同的返回类型，但返回类型并不足以区分所使用的是哪个方法。

当Java调用一个重载方法是，参数与调用参数匹配的方法被执行。在使用重载要注意以下的几点:
1.在使用重载时只能通过不同的参数列表，必须具有不同的参数列表。
2.不能通过访问权限、返回类型、抛出的异常进行重载。
3.方法的异常类型和数目不会对重载造成影响。
4.可以有不同的返回类型，只要参数列表不同就可以了。
5.可以有不同的访问修饰符。
6.可以抛出不同的异常。

重载(overloading) 是在一个类里面，方法名字相同，而参数不同。返回类型呢？可以相同也可以不同。
每个重载的方法（或者构造函数）都必须有一个独一无二的参数类型列表。
```

##### 3.2 基本类型和包装类的区别

[基本类型和包装类的区别](https://blog.csdn.net/qing_gee/article/details/101670051?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.nonecase&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.nonecase)

![img](https://img-blog.csdnimg.cn/20190226164706730.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0ODIwODAz,size_16,color_FFFFFF,t_70)

**01、包装类型可以为 null，而基本类型不可以**

数据库的查询结果可能是 null，如果使用基本类型的话，因为要自动拆箱（将包装类型转为基本类型，比如说把 Integer 对象转换成 int 值），就会抛出 `NullPointerException` 的异常。

**02、包装类型可用于泛型，而基本类型不可以**

因为泛型在编译时会进行类型擦除，最后只保留原始类型，而原始类型只能是 Object 类及其子类——基本类型是个特例

**03、基本类型在栈中直接存储的具体数值，而包装类型则存储的是堆中的引用。**

相比较于基本类型而言，包装类型需要占用更多的内存空间。假如没有基本类型的话，对于数值这类经常使用到的数据来说，每次都要通过 new 一个包装类型就显得非常笨重

**04、包装类的值可能相同，但指向的地址不同**

将“==”操作符应用于包装类型比较的时候，其结果很可能会和预期的不符

**05、自动装箱和自动拆箱**

当需要进行自动装箱时，如果数字在 -128 至 127 之间时，会直接使用缓存中的对象，而不是重新创建一个对象。

##### 3.3 main方法之前执行函数

```
static{}     // 静态代码块中的方法将在main之前执行
```



### 4 框架

------



#### 4.1 springboot

------



#### 4.2 springcloud

------



#### 4.3 ssm

------



#### 4.4 其他

------



##### 4.4.1 jsp能否用servlet完全代替？

答：JSP：Java Server Pages，java服务器页面。JSP替代Servlet输出HTML。

- Servlet在Java代码中通过HttpServletResponse对象动态输出HTML内容
- JSP在静态HTML内容中嵌入Java代码，Java代码被动态执行后生成HTML内容

- Servlet能够很好地组织业务逻辑代码，但是在Java源文件中通过字符串拼接的方式生成动态HTML内容会导致代码维护困难、可读性差
- JSP虽然规避了Servlet在生成HTML内容方面的劣势，但是在HTML中混入大量、复杂的业务逻辑同样也是不可取的

各有优势，不可替代































### 5 集合

------



#### 5.1 HashMap

[hashmap](https://juejin.im/post/6844903682664824845#heading-9)

- jdk1.7中HashMap默认采用**数组+单链表**方式存储元素，当元素出现哈希冲突时，会存储到该位置的单链表中。1.8中，除了数组和单链表外，当单链表中元素个数超过**8**个时，会进而转化为**红黑树**存储，巧妙地将遍历元素的时间复杂度从O(n)降低到了O(logn)）。1.7扩容时需要重新计算哈希值和索引位置，1.8并不重新计算哈希值，巧妙地采用和扩容后容量进行**&**操作来计算新的索引位置。

- 执行构造函数时，存储元素的数组并不会进行初始化，而是在**第一次放入元素**的时候，才会进行初始化操作。创建HashMap对象时，仅仅计算初始容量tableSizeFor()和新增阈值。

- HashMap在jdk1.7中采用**头插**入法，在扩容时会改变链表中元素原本的顺序，以至于在**并发**场景下导致链表成环的问题。而在jdk1.8中采用**尾插**入法，在扩容时会保持链表元素原本的顺序，就不会出现链表成环的问题了。

##### 5.1.1 **HashMap的底层数据结构？**

JDK1.7：数组+单链表。JDK1.8：数组+单链表（单链表中元素个数超过**8**个时转为红黑树存储）

```java
int hash = hash(key);//根据key计算哈希值
int i = indexFor(hash, table.length);//根据哈希值和数组长度计算在数组中的索引位置
									 //(n - 1) & hash数组长度和hash值相与计算index
```

##### **5.1.2 HashMap的存取原理？**

通过获取key对象的hashcode计算出该对象的哈希值，通过该哈希值与数组长度减去1进行位与运算（n-1 & hash）,得到数组的位置，没有冲突就新建节点，当发生hash冲突时，如果key值一样，则会替换旧的key的value，key不一样则新建next链表结点，当链表的长度超过8且数组容量大于64（树形化最小容量），则转换为红黑树存储。

##### **5.1.3 Java7和Java8的区别？**

JDK1.7：数组+单链表。JDK1.8：数组+单链表（单链表中元素个数超过**8**个时转为红黑树存储）。

JDK1.7扩容时转移原来的数据到newtable中采用头插法，1.8改为尾插法。

JDK1.7创建hashmap对象会创建一个长度为16的Entry[] table用来存储键值对数据。jdk1.8之后不是在构造方法创建了，而是在第一次调用put方法时才进行创建Node[] table。

##### **5.1.4 为啥会线程不安全？**

jdk1.7中，在多线程环境下，(头插法)扩容时会造成环形链或数据丢失。

jdk1.8中，在多线程环境下，PUT方法会发生数据覆盖的情况。

##### **5.1.5 有什么线程安全的类代替么?**

currentHashMap 以及 hashTable

##### **5.1.6 默认初始化大小是多少？为啥是这么多？为啥大小都是2的幂？**

16.

设置太大，就会浪费内存空间；设置太小，放几个元素就会扩容。

与index计算公式有关【index = (n - 1) & hash】，16-1=15的所有二进制位全为1，这种情况下，index的结果等同于hash后几位的值，只要计算出的hash本身分布均匀，Hash算法的结果就是均匀的。

##### **5.1.7 HashMap的扩容方式？负载因子是多少？为什么是这么多？**

当前插入元素大于Capacity*(HashMap当前长度)* **x** LoadFactor*(负载因子，默认值0.75f)*时进行扩容。

**①**：创建一个原数组两倍长度的空数组【确保2的幂】；

**②**：遍历原数组，把所有元素重新计算hash和index到新数组。

**0.75**----当负载因子过大的时候，意味着会出现大量的Hash的冲突，底层的红黑树变得异常复杂。对于查询效率极其不利(时间换空间)。当负载因子过小，数组所能存储的元素就会变少(空间换时间)。负载因子是0.75的时候，存储元素比较多，避免了过多的Hash冲突，使得底层的链表或者是红黑树的高度比较低，提升了空间效率。**threshold = loadFactor * capacity**。【0.75正好是3/4，而capacity又是2的幂。所以，两个数的乘积都是整数】

##### **5.1.8 HashMap的主要参数都有哪些？**

```Java
/**
 * 默认初始化容量，必须是2的次方
 */
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

/**
 * 最大容量。即HashMap的数组容量必须小于等于 1 << 30
 */
static final int MAXIMUM_CAPACITY = 1 << 30;

/**
 * 默认的负载因子
 */
static final float DEFAULT_LOAD_FACTOR = 0.75f;

/**
 * 树形化阈值；即当链表的长度大于8的时候，会将链表转为红黑树，优化查询效率。链表查询的时间复杂度为 
 * o(n) , 红黑树查询的时间复杂度为 o(log n)
 */
static final int TREEIFY_THRESHOLD = 8;

/**
 * 解树形化阈值；其实就是当红黑树的节点的个数小于等于6时，会将红黑树结构转为链表结构。
 */
static final int UNTREEIFY_THRESHOLD = 6;

/**
 * 树形化的最小容量；前面我们看到有一个树形化阈值，就是当链表的长度大于8的时候，会从链表转为红黑	  * 树。其实不一定是这样的。转为红黑树有两个条件：
*	① 链表的长度大于8
*	② HashMap数组的容量大于等于64
*	需要当上述两个条件都成立的情况下，链表结构才会转为红黑树。
 */
static final int MIN_TREEIFY_CAPACITY = 64;

```

##### **5.1.9 HashMap是怎么处理hash碰撞的？**

如果key相同，则会替换key对应的内容，key不相同，则接到后面的链表，如果链表长度到达8且数组的长度大于64(树形化的最小容量)时，则将链表转为红黑树，如果数组长度小于64，则是进行扩容

##### **5.1.10 hash的计算规则？**

将对象的key.hashcode()方法返回的hash值，进行无符号的右移16位，并与原来的hash值进行按位异或操作，目的是将hash的低16bit和高16bit做了一个异或，使返回的值足够散列

```java 
return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
```

##### **5.1.11 如何解决初始化，输入的值不是2的n次幂**

tableSizeFor()方法，将值的二进制数，从左边第一个出现的1开始，右边的所有值都通过位或运算变成1，使得可以找出比当前值大一点点的2的幂的数

```Java
 static final int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= n >>> 1
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
 }
```

##### 5.1.12 Set集合是如何保证元素不重复的？

因为HashSet的底层实现都是创建了一个HashMap，而HashMap的add方法，是将元素作为map的key进行存储的，map的key是不会重复的，所以HashSet中的元素也不会重复。

##### 5.1.13 重写equals方法，还必须要重写hashcode方法

使用hashcode方法将对象定义到一个地址上，hashcode值不同则对象一定不同；hashcode相同之后再用equals比较，减少了equals的使用。

如果重写了equals方法而没重写hashcode方法，会出现相等的对象equals相同，但前一步的hashcode不同，结果导致对象不相等。