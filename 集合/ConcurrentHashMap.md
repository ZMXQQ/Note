# 搞定HashMap线程不安全问题-----ConcurrentHashMap源码解析

**前言**

​		HashMap是线程不安全的集合，如果要保证线程安全该怎么做呢？

​		首先，HashMap为什么会线程不安全？

- ​				jdk1.7中，在多线程环境下，(头插法)扩容时会造成环形链或数据丢失。

- ​				jdk1.8中，在多线程环境下，PUT方法会发生数据覆盖的情况。



​		如何保证线程安全？

```java
//替代HashMap的方式
public static void main(String[] args) {
        /**方式0**/
        HashMap<Object,Object> map = new HashMap<>();
        
        /**方式1**/
        Collections.synchronizedMap(new HashMap<>());

        /**方式2**/
        Hashtable hashtable = new Hashtable();

        /**方式3**/
        ConcurrentHashMap<Object,Object> concurrentHashMap = new ConcurrentHashMap<Object,Object>(32,0.75f,32);

}
```



1. **Collections.synchronizedMap(Map)创建线程安全的map集合**

   ​	看源码可知，synchronizedMap把创建的对象**map**赋给要同步的对象**mutex**，之后每次操作map都会在方法上加上锁。

   ![image-20200827193320243](C:\Users\李佳庆\Desktop\APP\笔记本\image-20200827193320243.png)

2. **HashTable**

   ​	HashTable中对数据操作的大部分方法都会上锁，与方式1原理相似。如图中HashTable源码所示。

   ![image-20200827193941741](C:\Users\李佳庆\Desktop\APP\笔记本\image-20200827193941741.png)

   ​	缺点就是效率低，有些时候没必要加锁，却仍要线程等待。

   ​	hashtable的put方法：

   ![image-20200829164847390](C:\Users\李佳庆\Desktop\APP\笔记本\image-20200829164847390.png)

3. **ConcurrentHashMap**

   ​	HashMap是由Entry<K, V>[]数组（jdk1.7）, Node<K, V>[]数组（jdk1.8）组成，

   ​	而ConcurrentHashMap是由 Segment[HashEntry[]] （jdk1.7），Node<K, V>[]数组（jdk1.8）组成。

   ​	在jdk1.7中ConcurrentHashMap的底层数据结构是数组（Segment）加链表（HashEntry），可以看到内部类Segment的结构

   ```java
   /** Segment类继承于ReentrantLock类，所以Segment本质上是一个可重入的互斥锁 */
   static final class Segment<K,V> extends ReentrantLock implements Serializable {
   
       private static final long serialVersionUID = 2249069246763182397L;
   
       //存放数据的节点，与HashMap不同，用volatile修饰value和next节点
       transient volatile HashEntry<K,V>[] table;
   
       transient int count;
       transient int modCount;
       transient int threshold;
       final float loadFactor;
   
   }
   ```
   
   ​	在来看jdk1.7中ConcurrentHashMap的put方法
   
   ```java
   /** 1 先定位到Segment，然后进行put操作 */
   public V put(K key, V value) {
       Segment<K,V> s;
       //注意不能put空value
       if (value == null)
           throw new NullPointerException();
       int hash = hash(key);
       int j = (hash >>> segmentShift) & segmentMask;
       // 如果找不到该Segment，则新建一个。
       if ((s = (Segment<K,V>)UNSAFE.getObject          
            (segments, (j << SSHIFT) + SBASE)) == null) 
           s = ensureSegment(j);
       return s.put(key, hash, value, false);
   }
   ```
   
   ```java
   /** 2 put操作开始前，线程会获取Segment的互斥锁
   *	  更改或新建key-value键值对后再释放锁
   */
   final V put(K key, int hash, V value, boolean onlyIfAbsent) {
             /** 
             	*tryLock()获取锁，成功返回true，失败返回false。
       		*获取锁失败的话，则通过scanAndLockForPut()“自旋”获取锁，								*并返回”要插入的key-value“对应的”HashEntry链表。 
       		*/
               HashEntry<K,V> node = tryLock() ? null :
                   scanAndLockForPut(key, hash, value);
               V oldValue;
               try {
                   //根据”hash值获取HashEntry数组中对应的HashEntry链表
                   HashEntry<K,V>[] tab = table;
                   int index = (tab.length - 1) & hash;
                   HashEntry<K,V> first = entryAt(tab, index);
                   //遍历该 HashEntry链表
                   for (HashEntry<K,V> e = first;;) {
                       //HashEntry链表中的当前HashEntry节点不为null
                       if (e != null) {
                           K k;
                           //hash相等处的key也相等则覆盖value值
                           if ((k = e.key) == key ||
                               (e.hash == hash && key.equals(k))) {
                               oldValue = e.value;
                               //onlyIfAbsent:要插入的key不存在时才插入
                               if (!onlyIfAbsent) {
                                   e.value = value;
                                   ++modCount;
                               }
                               break;
                           }
                           e = e.next;
                       }
                       //当前HashEntry节点为null
                       else {
                           // 如果node非空，则将first设置为“node的下一个节点”。
                   		// 否则，新建HashEntry链表
                           if (node != null)
                               node.setNext(first);
                           else
                               node = new HashEntry<K,V>(hash, key, value, first);
                           int c = count + 1;
                           if (c > threshold && tab.length < MAXIMUM_CAPACITY)
                               //超出阈值则扩容
                               rehash(node);
                           else
                               setEntryAt(tab, index, node);
                           ++modCount;
                           count = c;
                           oldValue = null;
                           break;
                       }
                   }
               } finally {
                   //释放锁
                   unlock();
               }
               return oldValue;
           }
   ```
   
   ​	综上，**ConcurrentHashMap线程安全的实现原理（jdk1.7）：**对于ConcurrentHashMap的添加，删除操作，在操作开始前，线程都会获取Segment的互斥锁；操作完毕之后，才会释放。而对于读取操作，它是通过volatile去实现的，HashEntry数组是volatile类型的，而volatile能保证“即对一个volatile变量的读，总是能看到（任意线程）对这个volatile变量最后的写入”，即我们总能读到其它线程写入HashEntry之后的值。[参考链接](https://blog.csdn.net/hbtj_1216/article/details/76205903)
   
   
   
   ​	而jdk1.8使用红黑树O（logn）来优化链表O（n）;并取消了segment数组，直接用Node[]保存数据，降低锁的粒度，减少并发冲突的概率。
   
   
   
   ![image-20200827200955228](C:\Users\李佳庆\Desktop\APP\笔记本\image-20200827200955228.png)

```java
/**
	0.key或value不能为空
	
    1.根据key的hash值定位到桶位置

    2.判断if(table==null)，先初始化table。

    3.判断if(table[i]==null),cas添加元素。成功则跳出循环，失败则进入下一轮for循环。

    4.table[i]!=null，判断是否有其他线程在扩容table，有则帮忙扩容，扩容完成再添加元素。进入真正的put步骤

    5.真正的put步骤。桶的位置不为空，synchronized锁锁住头节点，遍历该桶的链表或者红黑树，若key已存在，则覆盖；不存在则将key插入到链表或红黑树的尾部。
*/

final V putVal(K key, V value, boolean onlyIfAbsent) {
        if (key == null || value == null) throw new NullPointerException();// key和value不允许null
        int hash = spread(key.hashCode());//两次hash，减少hash冲突，可以均匀分布
        int binCount = 0;//i处结点标志，0: 未加入新结点, 2: TreeBin或链表结点数, 其它：链表结点数。主要用于每次加入结点后查看是否要由链表转为红黑树
        for (Node<K, V>[] tab = table; ; ) {//CAS经典写法，不成功无限重试
            Node<K, V> f;
            int n, i, fh;
            //检查是否初始化了，如果没有，则初始化
            if (tab == null || (n = tab.length) == 0)
                tab = initTable();
            /**
             * i=(n-1)&hash 等价于i=hash%n(前提是n为2的幂次方).即取出table中位置的节点用f表示。 有如下两种情况：
             * 1、如果table[i]==null(即该位置的节点为空，没有发生碰撞)，则利用CAS操作直接存储在该位置， 如果CAS操作成功则退出死循环。
             * 2、如果table[i]!=null(即该位置已经有其它节点，发生碰撞)
             */
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                if (casTabAt(tab, i, null,
                        new Node<K, V>(hash, key, value, null)))
                    break;                   // no lock when adding to empty bin
            } else if ((fh = f.hash) == MOVED)//检查table[i]的节点的hash是否等于MOVED，如果等于，则检测到正在扩容，则帮助其扩容
                tab = helpTransfer(tab, f);
            else {//table[i]的节点的hash值不等于MOVED。
                V oldVal = null;
                // 针对首个节点进行加锁操作，而不是segment，进一步减少线程冲突
                synchronized (f) {
                    //判断加锁时当前位置节点是否被其他线程操作，被改变则再去循环
                    if (tabAt(tab, i) == f) {
                        if (fh >= 0) {
                            binCount = 1;
                            for (Node<K, V> e = f; ; ++binCount) {
                                K ek;
                                // 如果在链表中找到值为key的节点e，直接设置e.val = value即可
                                if (e.hash == hash &&
                                        ((ek = e.key) == key ||
                                                (ek != null && key.equals(ek)))) {
                                    oldVal = e.val;
                                    if (!onlyIfAbsent)
                                        e.val = value;
                                    break;
                                }
                                // 如果没有找到值为key的节点，直接新建Node并加入链表即可
                                Node<K, V> pred = e;
                                if ((e = e.next) == null) {//插入到链表末尾并跳出循环
                                    pred.next = new Node<K, V>(hash, key,
                                            value, null);
                                    break;
                                }
                            }
                        } else if (f instanceof TreeBin) {// 如果首节点为TreeBin类型，说明为红黑树结构，执行putTreeVal操作
                            Node<K, V> p;
                            binCount = 2;
                            if ((p = ((TreeBin<K, V>) f).putTreeVal(hash, key,
                                    value)) != null) {
                                oldVal = p.val;
                                if (!onlyIfAbsent)
                                    p.val = value;
                            }
                        }
                    }
                }
                if (binCount != 0) {
                    // 如果节点数>＝8，那么转换链表结构为红黑树结构
                    if (binCount >= TREEIFY_THRESHOLD)
                        treeifyBin(tab, i);//若length<64,直接tryPresize,两倍table.length;不转红黑树
                    if (oldVal != null)
                        return oldVal;
                    break;
                }
            }
        }
        // 计数增加1，有可能触发transfer操作(扩容)
        addCount(1L, binCount);
        return null;
    }
```

ConcurrentHashMap（jdk1.8）get操作：

```java
/**
     * 根据key在Map中找出其对应的value，如果不存在key，则返回null，
     * 其中key不允许为null，否则抛异常
     * 对于节点可能在链表或树上的情况，需要分别去查找
     *
     * @throws NullPointerException if the specified key is null
     */
    public V get(Object key) {
        Node<K, V>[] tab;
        Node<K, V> e, p;
        int n, eh;
        K ek;
        int h = spread(key.hashCode());//两次hash计算出hash值
        //根据hash值确定节点位置
        if ((tab = table) != null && (n = tab.length) > 0 &&
                (e = tabAt(tab, (n - 1) & h)) != null) {
            // 搜索到的节点key与传入的key相同且不为null,直接返回这个节点
            if ((eh = e.hash) == h) {
                if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                    return e.val;
            } else if (eh < 0)//如果eh<0 说明这个节点在树上 直接寻找
                return (p = e.find(h, key)) != null ? p.val : null;
            //否则遍历链表 找到对应的值并返回
            while ((e = e.next) != null) {
                if (e.hash == h &&
                        ((ek = e.key) == key || (ek != null && key.equals(ek))))
                    return e.val;
            }
        }
        return null;
    }
```

