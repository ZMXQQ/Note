### **ArrayLIst** 

**查询效率高，增删效率低，线程不安全**。底层由transient Object[] elementData对象数组实现，所以当存储基本数据类型时只能存储它们对应的包装类。  HashMap存储元素用的是Node<K, V>节点，包含

```Java
final int hash;
// final:一个键值对的key不可改变
final K key;
V value;
//指向下个节点的引用
Node<K, V> next;
```



**初始化**

通过无参构造方法的方式对ArrayList()初始化，只赋值Object[] elementData为一个默认空数组Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {}，所以数组容量为0，只有真正对数据进行添加add时，才分配默认DEFAULT_CAPACITY = 10的初始容量

例如下面代码会输出0，因为size是数组元素数量，而不是容量。

```Java
ArrayList list = new ArrayList(10);
System.out.println(list.size());
```



**扩容**

add()添加元素前会通过ensureCapacityInternal判断容量，发现容量不够则将数组扩容1.5倍，然后把原数组的数据，原封不动的复制到新数组中，这个时候再把指向原数组的地址换到新数组。



**删除与增加**

如果删除或增在数组中间的位置index，则会复制index后的所有元素右移（留出位置增加）一位或左移（覆盖掉要删除的）一位，所以效率慢。

（ArrayList的增删底层调用的`copyOf()`被优化过，现代CPU对内存可以**块操作**，ArrayList的增删一点儿也不会比LinkedList慢）



**线程不安全**

1. Collections.synchronizedList把一个普通ArrayList包装成一个线程安全版本的数组容器

2. 用CopyOnWriteArrayList集合https://www.jianshu.com/p/fe84c775ab9e

   ​	其中`add()`方法会加`lock`锁，然后复制出一个新的数组，往新的数组里边`add`真正的元素，最后把array的指向改变为新的数组，`get()`方法又或是`size()`方法只是获取array所指向的数组的元素或者大小。读不加锁，写加锁。

###### 		**缺点**

CopyOnWriteArrayList是很耗费内存的，每次`set()/add()`都会复制一个数组出来，另外就是CopyOnWriteArrayList只能保证数据的**最终一致性**，不能保证数据的实时一致性。



**遍历**

```Java
1:如何创建泛型对象
ArrayList<泛型> list=new ArrayList<>();

2:如何添加元素：
一次添加一个元素：
list.add(元素);
一次添加多个元素：
Collections.addAll(集合，元素，元素，...);
3:得到集合元素的个数
list.size();
4:得到某一个元素
list.get(下标);
5：如何判断集合里面是否出现指定元素
list.contains();
6:遍历
for+下标
for(int x=0;x<list.size();x++){
    //x->下标
    //list.get(元素)；
}
foreache
for(集合的泛型 x :list){
    //x->元素
}
迭代器********（重点）
for(得到迭代器对象;判断迭代器上面是否还有下一个元素;){
				取出下一个元素
			  }

for(Iterator<泛型>iterator=list.iterator();iterator.hasNext;){
    iterator.next();->元素
}

```

