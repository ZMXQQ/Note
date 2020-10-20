## Object类中有哪些方法？.md

##### registerNatives方法

```
private static native void registerNatives();
    /**
     * 对象初始化时自动调用此方法
     */
    static {
        registerNatives();
    }
```

作用：类被加载时注册该类所包含的除了registerNatives()方法以外的所有本地方法，例如getClass()、hashCode()、clone()等



##### getClass方法

```
/**
     * 返回此Object的运行时类型
     */
    public final native Class<?> getClass();
```



##### hashCode方法

```
public native int hashCode();
```

作用：返回对象的内存地址，这个方法在一些具有哈希功能的Collection中用到。



##### equals方法

```
public boolean equals(Object obj) {
        return (this == obj);
    }
```

作用：比较的是对象的内存地址。子类一般都要重写这个方法，改为比较对象值是否相等（StringBuffer就没重写）

注意：用equals方法判断之前会调用hashcode方法判断对象地址是否相等，地址相等再用equals进行比较，减少equals比较次数。所以重写equals方法、改用对象值比较时，一定要重写hashcode方法，不然对象值相同的不同对象在hashcode判断这一步就判成了不等。



##### clone方法

```
protected native Object clone() throws CloneNotSupportedException;
```

作用：对象的浅复制，只有实现了Cloneable接口才可以调用该方法，否则抛出CloneNotSupportedException异常。



##### toString方法

```
/**
     * 返回该对象的字符串表示,非常重要的方法
     * getClass().getName();获取字节码文件的对应全路径名例如java.lang.Object
     * Integer.toHexString(hashCode());将哈希值转成16进制数格式的字符串。
     */
    public String toString() {
        return getClass().getName() + "@" + Integer.toHexString(hashCode());
    }
```



##### notify、notifyAll方法

```
public final native void notify();

public final native void notifyAll();
```

作用：唤醒在该对象上等待的某个/所有线程。



##### wait方法

```
public final native void wait(long timeout) throws InterruptedException;
```

作用：使当前线程进入等待状态，当

1）超出timeout

2）其他线程通过调用notify方法或notifyAll方法通知当前等待的线程醒来

3）其他线程调用了interrupt中断该线程

则当前线程退出等待状态。



##### finalize方法

```
protected void finalize() throws Throwable {
    }
```

作用：回收对象时调用。子类若要在对象回收时添加逻辑处理，可重写finalize方法。