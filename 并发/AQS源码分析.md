## 继承自AQS的ReentrantLock源码分析



#### ReentrantLock源码分析

ReentrantLock(*可重入互斥锁*)。可重入锁又名递归锁，是指在同一个线程在外层方法获取锁的时候，再进入该线程的内层方法会自动获取锁（前提锁对象得是同一个对象或者class），不会因为之前已经获取过还没释放而阻塞。

我们从构造函数开始逐步分析。

**ReentrantLock**的两个构造函数，默认使用的是非公平sync对象

```java
public ReentrantLock() {
        sync = new NonfairSync();
}
    
public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
}
```

看下继承自Sync的非公平同步类**NonfairSync**

```java
static final class NonfairSync extends Sync {
        private static final long serialVersionUID = 7316153563782823691L;

        /**
         * 非公平锁：线程加锁时直接尝试获取锁，获取不到才会到等待队列的队尾排队等待
         */
        final void lock() {
            if (compareAndSetState(0, 1))//CAS
                setExclusiveOwnerThread(Thread.currentThread());//设置所有者线程为当前线程
            else
                acquire(1);//失败后尝试获取锁。
        }

        protected final boolean tryAcquire(int acquires) {
            return nonfairTryAcquire(acquires);
        }
    }
```

compareAndSetState(0, 1)。0是期望值，1是更新值。只要期待值与当前同步状态state相等，则把state更新为update(更新值)。

这里state就是当前线程获得锁的状态。先看下state的说明。

```java
private volatile int state;//state是Volatile修饰的，用于保证一定的可见性和有序性。
```

1. **State初始化的时候为0，表示没有任何线程持有锁。**
2. **当有线程持有该锁时，值就会在原来的基础上+1，同一个线程多次获得锁时，就会多次+1，这里就是可重入的概念。**
3. **解锁也是对这个字段-1，一直到0，此线程对锁释放。**

所以此处的**compareAndSetState(0, 1)**就是为了判断当前线程state是否为0，是0则获取锁成功，并更新state为1.

```java
protected final boolean compareAndSetState(int expect, int update) {
        return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
    }
```



如果获取锁失败呢？也就是当前线程的state不为0，那就会执行**acquire(1)**;

```java
//arg代表获取锁的次数
public final void acquire(int arg) {
    	//tryAcquire(arg)为true，代表获取锁成功(当前线程为独占线程)。
    	//获取锁失败后，再执行acquireQueued将线程排队。
        if (!tryAcquire(arg) &&
            //addWaiter方法其实就是把对应的线程以Node的数据结构形式加入到双端队列里，返回的是一个包含该线程的Node。而这个Node会作为参数，进入到acquireQueued方法中。acquireQueued方法可以对排队中的线程进行“获锁”操作。
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            //中断当前线程
            selfInterrupt();
    }
```



这几个方法都是由ReentrantLock中的(非)公平同步类调用，而(Non)fairSync继承自Sync类，Sync类继承自AQS，所以这些方法都在AQS中定义。例如上面的tryAcquire方法就被ReentrantLock重写了，直接调用AQS的tryAcquire会抛出UnsupportedOperationException异常。

![image-20200822101223233](C:\Users\李佳庆\AppData\Roaming\Typora\typora-user-images\image-20200822101223233.png)

可以看到ReentrantLock中的公平同步类和非公平同步类都重写了tryAcquire。

所以acquire调用的就是NonfairSync中已经重写了的**tryAcquire**

```java
//NonfairSync类里最后一个方法，当前acquires=1
protected final boolean tryAcquire(int acquires) {
    return nonfairTryAcquire(acquires);
}

final boolean nonfairTryAcquire(int acquires) {
    		//当前线程
            final Thread current = Thread.currentThread();
    		//当前state值
            int c = getState();
    		//如果没有线程获取锁
            if (c == 0) {
                if (compareAndSetState(0, acquires)) {
                    //获取锁成功，并将当前线程设置为独占线程
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
    		//如果当前线程为独占线程（已获取锁），则将state加1。
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0) // overflow
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
```



查看addWaiter方法前，要先来看下**Node**节点，也就是未获取到锁的线程加入到了哪种队列中去。

这就涉及到**AQS核心思想**：如果被请求的共享资源空闲，那么就将当前请求资源的线程设置为有效的工作线程，将共享资源设置为锁定状态；如果共享资源被占用，就需要一定的阻塞等待唤醒机制来保证锁分配。这个机制主要用的是CLH队列的变体---虚拟双向队列(FIFO)---实现的，将暂时获取不到锁的线程加入到队列中。

AQS使用一个Volatile的int类型的state来表示同步状态，通过内置的FIFO队列来完成资源获取的排队工作，通过CAS完成对State值的修改。

AQS中基本的数据结构——**Node**，Node即为虚拟双向队列(FIFO)中的节点。

```java
static final class Node {
    /** 标志线程以共享/独占方式等待锁 */
    static final Node SHARED = new Node();
    static final Node EXCLUSIVE = null;

    /** 表示线程获取锁的请求已经取消 */
    static final int CANCELLED =  1;
    /** 表示线程已经准备好，就等资源释放了(被唤醒了) */
    static final int SIGNAL    = -1;
    /** 表示节点在等待队列中，节点线程等待唤醒 */
    static final int CONDITION = -2;
    /** 当前线程处在SHARED情况下，该字段才会使用(其他操作介入，也要确保传播继续) */
    static final int PROPAGATE = -3;
    
    //当前节点在队列中的状态
    volatile int waitStatus;
    //前驱指针
    volatile Node prev;
    //后继指针
    volatile Node next;
    //表示处于该节点的线程
    volatile Thread thread;
    //指向下一个处于CONDITION状态的节点
    Node nextWaiter;
    
    /** 返回前驱节点，没有则抛出空指针异常 */
    final Node predecessor() throws NullPointerException {
        Node p = prev;
        if (p == null)
            throw new NullPointerException();
        else
            return p;
    }
    
    /** 几个Node构造函数 */
    //用来初始化头节点或SHARED标志
    Node() {    
    }

    //用于addWaiter方法
    Node(Thread thread, Node mode) {    
        this.nextWaiter = mode;
        this.thread = thread;
    }

    //用于Condition
    Node(Thread thread, int waitStatus) { 
        this.waitStatus = waitStatus;
        this.thread = thread;
    }
}
//队列头节点
private transient volatile Node head;
//队列尾节点
private transient volatile Node tail;
```



现在再来看addWaiter方法是如何把线程加入到双端队列的。

```java
//这里的mode就是Node.EXCLUSIVE，表示独占模式
private Node addWaiter(Node mode) {
    //通过当前的线程和锁模式新建一个节点。
    Node node = new Node(Thread.currentThread(), mode);
    //pred指向尾节点tail
    Node pred = tail;
    //如果尾节点不为空，则将当前节点插入到尾节点后面(设为尾节点)，并返回node
    if (pred != null) {
        node.prev = pred;
        if (compareAndSetTail(pred, node)) {
            pred.next = node;
            return node;
        }
    }
    //如果Pred指针是Null（说明等待队列中没有元素），或者当前Pred指针和Tail指向的位置不同（说明已经被别的线程修改），则进入enq初始化head节点，并将node设为尾节点。
    enq(node);
    return node;
}
```

**注意双向链表中，第一个节点(头节点)为虚节点，其实并不存储任何信息，只是占位。真正的第一个有数据的节点，是在第二个节点开始的。**

回到上面的**acquireQueued(addWaiter(Node.EXCLUSIVE), arg))**方法，acquireQueued会把放入队列中的线程不断去获取锁，直到获取成功或者不再需要获取（中断）。

那么队列中的线程什么时候可以获取到锁？一直获取不到又会怎样呢？

```java
final boolean acquireQueued(final Node node, int arg) {
	// 标记是否成功拿到资源
	boolean failed = true;
	try {
		// 标记等待过程中是否中断过
		boolean interrupted = false;
		// 开始自旋，要么获取锁，要么中断
		for (;;) {
			// 获取当前节点的前驱节点
			final Node p = node.predecessor();
			// 如果p是头结点，说明当前节点在真实数据队列的首部，就尝试获取锁（别忘了头结点是虚节点）
			if (p == head && tryAcquire(arg)) {
				// 获取锁成功，头指针移动到当前node
				setHead(node);
				p.next = null; // help GC
				failed = false;
				return interrupted;
			}
			// 说明p为头节点且当前没有获取到锁（可能是非公平锁被抢占了）或者是p不为头结点，这个时候就要判断当前node是否要被阻塞（被阻塞条件：前驱节点的waitStatus为-1），防止无限循环浪费资源。
			if (shouldParkAfterFailedAcquire(p, node) && parkAndCheckInterrupt())
				interrupted = true;
		}
	} finally {
		if (failed)//未成功拿到资源，设置当前节点状态为CANCELLED
			cancelAcquire(node);
	}
}

// 靠前驱节点判断当前线程是否应该被阻塞，true阻塞，false不阻塞
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
	// 获取前驱结点的节点状态
	int ws = pred.waitStatus;
	// 说明前驱结点处于唤醒状态，此时需要阻塞当前节点，返回true
	if (ws == Node.SIGNAL)
		return true; 
	// 通过枚举值我们知道waitStatus>0是取消状态
	if (ws > 0) {
		do {
			// 循环向前查找取消节点，把取消节点从队列中剔除
			node.prev = pred = pred.prev;
		} while (pred.waitStatus > 0);
        //直到前面某一个节点非取消节点，将非取消节点连接当前节点
		pred.next = node;
	} else {//前驱节点waitStatus为0或-3。
		// 设置前驱节点等待状态为SIGNAL
		compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
	}
	return false;
}

//parkAndCheckInterrupt主要用于挂起当前线程，阻塞调用栈，返回当前线程的中断状态。
private final boolean parkAndCheckInterrupt() {
    LockSupport.park(this);
    return Thread.interrupted();
}
```

前面的两个问题得到解决：只要前驱节点是头节点，就尝试获取锁。前驱节点不是头节点，或获取锁失败，则判断是否需要阻塞当前线程。如果前驱节点状态是SIGNAL，表明前驱节点准备获取资源，所以当前节点阻塞；如果前驱节点是取消状态CANCELLED(不再获取资源)，移除所有取消状态的前驱节点，继续自旋尝试获取锁；如果前驱节点waitStatus为0(默认值)或-3，则更改前驱节点状态为SIGNAL，继续自旋。







*参考链接* ： [[从ReentrantLock的实现看AQS的原理及应用](https://tech.meituan.com/2019/12/05/aqs-theory-and-apply.html)](https://tech.meituan.com/2019/12/05/aqs-theory-and-apply.html)