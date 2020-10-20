## HashMap源码

##### 		列举并解释了一些JDK1.8版本的HashMap源码，所有方法和重要的细节源码中的注释都解释的非常清楚。我只是按照自己的理解搬运了一遍。文章的代码顺序按照源码中出现的顺序。

源码最开始的内容，几个比较重要的参数。

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
 * 树形化的最小容量；前面我们看到有一个树形化阈值，就是当链表的长度大于8的时候，会从链表转为红黑 *  树。其实不一定是这样的。转为红黑树有两个条件：
*	① 链表的长度大于8
*	② HashMap数组的容量大于等于64
*	需要当上述两个条件都成立的情况下，链表结构才会转为红黑树。
*   若链表长度大于8，但HashMap数组容量小于64，则会进行扩容
 */
static final int MIN_TREEIFY_CAPACITY = 64;
```

```Java
static final int hash(Object key) {//根据传入的key对象计算hash值
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);//key为null时hash值为0，数组下标index也会为0.
}
```

hashCode()计算并返回内存地址。

(h = key.hashCode()) ^ (h >>> 16)       注意这里为什么要异或hash值右移16位。

**因为当前hashcode方法计算的散列值仍会出现较多冲突，由于数组的边界影响，hashcode值的高位几乎不会用到。所以利用右移16位后的hashcode与自身进行异或，提升hash值的散列性，减少系统的损失。**



```Java
/*
tableSizeFor()方法，将值的二进制数，从左边第一个出现的1开始，右边的所有值都通过位或运算变成1，使得可以找出比当前值大一点点的2的幂的数
*/
static final int tableSizeFor(int cap) {
        int n = cap - 1;//这里减1是为了避免初始化数组容量为2的幂时仍然向上取值
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
//例如cap为5，n为4
0000 1000
0000 0100   n>>>1
    0000 1100
    0000 0011  n>>>2(每移动一次，1的位数翻倍，所以下次右移位数也翻倍)
    	0000 1111(此时n=7)
    		..... n>>>4(之后的右移都为0，不影响结果)
    ...
    最后 n + 1 = 8(大于5的最小2次幂)
```

```Java
int threshold;//阈(yu)值，(capacity * load factor)，数组容量*负载因子
//决定hashmap是否扩容
```

```Java
/*
*	几个构造函数，注意创建HashMap对象时，仅仅计算数组初始容量tableSizeFor()和新增阈值，只有第一次放入元素时才进行初始化。
*/
public HashMap(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);
        this.loadFactor = loadFactor;
        this.threshold = tableSizeFor(initialCapacity);
    }
public HashMap(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }
public HashMap() {
        this.loadFactor = DEFAULT_LOAD_FACTOR; // 其他所有字段为默认值
    }
```

```Java
public V put(K key, V value) {	//添加元素方法，最常用。
        return putVal(hash(key), key, value, false, true);
}

```

putVal方法逻辑很简单，代码也比较好理解，主要是要明白代码是如何处理hash碰撞的。

```Java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;//第一次put元素调用扩容函数resize()初始化数组。
        if ((p = tab[i = (n - 1) & hash]) == null)//非第一次put则计算数组下标，当前下标位置为空直接新建节点。
            tab[i] = newNode(hash, key, value, null);//构造节点参数格式(int hash, K key, V value, Node<K,V> next)
        else {//当前下标位置已经有值。
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))//如果传入的key的hash值与当前下标位置的hash值相同 且 key值也相同，则替换value。
                e = p;
            else if (p instanceof TreeNode)//key不同，且当前节点为树节点，则用树节点添加方式插入。
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {//当前下标节点next节点为空，则添加到next节点中。
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) //添加后判断节点数量是否大于等于树形化阈值，超过则将链表转化为红黑树。（注意treeifyBin方法中会根据数组大小是否超过树形化最小容量决定扩容还是转为红黑树）
                            treeifyBin(tab, hash);
                        break;
                    }
                    //当前下标节点next节点不为空，则判断next节点hash和key是否和当前put的相等。相等退出循环，e就是p.next，后面方法会替换value；不等则将next节点赋值给p，继续下一层循环。
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { //前面几种e非空的情况执行这个方法，替换value值
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;//涉及到Fail-Fast机制，在出现其他线程改变map值时抛出异常，这里不做详解。可参考下面链接。
        if (++size > threshold)//超出阈值则扩容。
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```

[modCount](https://blog.csdn.net/u012926924/article/details/50452411)详解。

注意这里计算数组下标的公式：

```Java
i = (n - 1) & hash
```

数组下标 = （数组长度 - 1）& hash。当数组长度即n是2的幂时，n-1的二进制则为0000 1111...，保证了后几位全都是1，此时再和hash值相与，就保证了结果一定是hash值的**后几位**。

①为什么要下标结果和hash值后几位相同？---这样可以保证只要hash值分布足够均匀，数组下标就足够分散。

②这里的后几位是多少位？(为什么用数组长度-1？) ---n-1，当n是8时，8-1的二进制为0000 1111。此时与hash值相与，后四位得到的结果一定小于等于1111，也就保证了数组的结果一定小于8且大于等于0。同时也就保证了数组下标大于等于0，小于数组长度n。不会出现数组下标越界的情况。



最后，重要的函数resize()，负责数组的初始化和扩容。

```Java
final Node<K,V>[] resize() {//返回的是一个Node数组(废话)。
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
        if (oldCap > 0) {//旧数组不为空
            if (oldCap >= MAXIMUM_CAPACITY) {//超过最大容量不扩容，设阈值为最大容量(0x7FFFFFFF)
                threshold = Integer.MAX_VALUE; //0x7FFFFFFF
                return oldTab;
            }
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; //不超过最大容量则容量x2,此时容量仍小于最大容量且大于默认初始容量则阈值x2。
        }
        else if (oldThr > 0) //旧数组为空且阈值大于0，阈值赋为新数组容量。
            newCap = oldThr;
        else {               //否则容量阈值都是默认值
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        if (newThr == 0) {//为前面没给阈值的情况初始化一个阈值。
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;//改变阈值和数组容量后的新数组。
        if (oldTab != null) {//非初始化情况，进行元素从旧数组到新数组的转移。
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    if (e.next == null)
                        newTab[e.hash & (newCap - 1)] = e;
                    else if (e instanceof TreeNode)
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { // preserve order
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            next = e.next;
                            if ((e.hash & oldCap) == 0) {
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        return newTab;
    }
```

注意这句

```Java
newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY)
```

默认阈值为负载因子*默认初始容量。

①为什么负载因子是0.75呢？ ---当负载因子过大的时候，意味着会出现大量的Hash的冲突，底层的红黑树变得异常复杂。对于查询效率极其不利(时间换空间)。当负载因子过小，数组所能存储的元素就会变少(空间换时间)。负载因子是0.75的时候，存储元素比较多，避免了过多的Hash冲突，使得底层的链表或者是红黑树的高度比较低，提升了空间效率。**threshold = loadFactor * capacity**。【0.75正好是3/4，而capacity又是2的幂。所以，两个数的乘积都是整数。



```Java
(e.hash & oldCap) == 0
```

数组元素转移到新数组时有这么个条件。先说结论：

- 当**(e.hash & oldCap) == 0**时满足**(e.hash & *oldCap-1)==(e.hash & 2*oldCap-1)**，即元素新旧数组中的下标一致
- 当**(e.hash & oldCap) != 0**时满足**(e.hash & *oldCap-1) + oldCap==(e.hash & 2*oldCap-1)**，即元素新数组中的下标为旧数组中下标加上旧数组容量。

详细解释请参考为什么用[(e.hash & oldCap) == 0](https://blog.csdn.net/u012926924/article/details/50452411)？



其他方法会相继完善，后面会下载jdk1.7版本，对比一下两个版本的源码差异。

有时间了再完善，欢迎指正批评。

