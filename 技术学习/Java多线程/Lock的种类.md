# 各种Lock

Java中锁的种类有很多：公平锁/非公平锁、可重入锁/不可重入锁、共享锁/排他锁、乐观锁/悲观锁、分段锁、偏向锁/轻量级锁/重量级锁、自旋锁。

## ReentrantLock类

Jdk1.5新增的ReentrantLock类和synchronized关键字一样可以实现线程间同步互斥，但是它在拓展功能上更加强大，比如**嗅探锁定**、**多路分支**等功能，使用的时候也比synchronized更加灵活。

使用Condition对象可以实现类似synchronized的wait()/notify()/notifyAll()同样的功能。Condition有更好的灵活性，比如可以实现多路通知功能，也就是在一个Lock对象中创建多个Condition（即对象监视器）实例，线程对象可以注册在指定的Condition中，从而可以有选择性的进行线程通知，在调度线程上更加灵活。

在使用notify()/notifyAll()方法进行通知时，被通知的线程是由JVM随机选择的，但是使用ReentrantLock和Condition可以实现“选择性通知”，这个功能是非常重要的，而且在Condition类时默认提供的。

而synchronized就相当于整个Lock对象中只有一个单一的Condition对象，所有的线程都注册在它一个对象身上，线程开始notifyAll时，需要通知所有的waiting线程，没有选择权，会出现相当大的效率问题。

**signal()和signalAll()区别**

signalAll通知所有使用了同一个Condition对象的线程。signal()通知所有使用了Condition对象的某一个线程，通过源码可以看到通知的用时是位于队首的那个。

## 1. 公平锁和非公平锁

公平锁表示线程获取锁顺序是按照线程加锁的顺序来分配的，即FIFO顺序。而非公平锁就是一种获取锁的抢占机制，是随机获得锁的。有可能后申请的线程比先申请的线程优先获取锁，可能会造成优先级反转或者饥饿现象。

在公平的锁中，如果有另一个线程持有锁或者有其他线程在等待队列中等待这个所，那么新发出的请求的线程将被放入到队列中。而非公平锁上，只有当锁被某个线程持有时，新发出请求的线程才会被放入队列中。

对于**ReentrantLock**而言，通过构造函数指定该锁是否是公平锁，默认是非公平锁。非公平锁的优点在于吞吐量比公平锁大。

```Java
public ReentrantLock(boolean fair) {
	sync = fair ? new FairSync() : new NonfairSync();
}
```

对于Synchronized而言，也是一种非公平锁。由于其并不像ReentrantLock是通过AQS的来实现线程调度，所以并没有任何办法使其变成公平锁。

> 参考：
>
> https://blog.csdn.net/z69183787/article/details/50971317

## 2. 可重入锁和不可重入锁

可重入锁又名递归锁，直指同一个线程在外层方法获得锁之后，在进入内层方法时，会自动获得锁。**ReentrantLock**和**Synchronized**都是可重入锁。可重入锁的好处之一就是在一定程度上避免死锁。下面通过构建可重入锁和不可重入锁来详细的了解一下。

首先看一个类的定义：

```java
package com.wangjun.thread.IsReentrantLock;

public class Test {
	
	Lock1 lock = new Lock1();
	
	public static void main(String[] args) throws InterruptedException {
		Test t = new Test();
		t.test1();
	}
	
	public void test1() throws InterruptedException {
		lock.lock();
		System.out.println("test1方法执行...调用test2方法");
		test2();
		lock.unLock();
	}
	
	public void test2() throws InterruptedException {
		lock.lock();
		System.out.println("test2方法执行...");
		lock.unLock();
	}
}
```

如果Lock1是一个不可重入锁，那么test1执行的时候已经拿到了锁，再调用test2，由于test2一直获取不到锁，因此会进入死锁状态。

我们来看一下Lock1实现的不可重入锁：

```java
package com.wangjun.thread.IsReentrantLock;

/*
 * 不可重入锁设计
 */
public class Lock1 {
	private boolean lock = false;
	
	public synchronized void lock() throws InterruptedException {
		while(lock) {
			wait();
		}
		lock = true;
	}
	
	public synchronized void unLock() {
		lock = false;
		notify();
	}
}
```

不可重入锁的弊端可以清晰的看到，那么如何构造一个可重入锁呢？我们来看一下可重入锁Lock2的设计：

```java
package com.wangjun.thread.IsReentrantLock;

public class Lock2 {

	private boolean lock = false;  //记录是否有线程获得锁
	private Thread curThread = null;  //记录获得锁的线程
	private int lockCount = 0;  //记录加锁次数
	
	public synchronized void lock() throws InterruptedException {
		Thread thread = Thread.currentThread();
		//如果已经加锁并且不是等当前线程，那么就等待
		while(lock && thread != curThread) {
			wait();
		}
		
		lock = true;  //线程获得锁
		lockCount++;  //加锁次数+1
		curThread = thread;  //获得锁的线程等于当前线程

	}
	
	public synchronized void unLock() {
		Thread thread = Thread.currentThread();
		// 如果是获得锁的线程调用unLock，那么加锁次数减一
		if(thread == curThread) {
			lockCount--;
			//所有的加锁都释放，通知其他线程可以获得锁了
			if(lockCount == 0) {
				notify();
			}
		}
	}
}
```

将测试类的：

```java
Lock1 lock = new Lock1();
```

换成

```java
Lock2 lock = new Lock2();
```

可以看到线程运行正常，不再造成死锁。在test1加锁后调用test2方法时，由于是同一个线程，所以test2种也会顺利拿到锁，并继续执行。

可重入锁就是要保证：线程可以进入任何一个它已经拥有锁所同步着的代码块。

> 参考：
>
> https://www.cnblogs.com/dj3839/p/6580765.html

## 3. 共享锁和排他锁

共享锁也叫S锁，读锁，该锁可以被多个线程持有；

排他锁也叫X锁，写锁，独享锁，该锁只能被一个线程持有。

共享锁【S锁】
若事务T对数据对象A加上S锁，则事务T可以读A但不能修改A，其他事务只能再对A加S锁，而不能加X锁，直到T释放A上的S锁。这保证了其他事务可以读A，但在T释放A上的S锁之前不能对A做任何修改。

排他锁【X锁】
又称写锁。若事务T对数据对象A加上X锁，事务T可以读A也可以修改A，其他事务不能再对A加任何锁，直到T释放A上的锁。这保证了其他事务在T释放A上的锁之前不能再读取和修改A。

对于ReentrantLock和Synchronized而言，是独享锁，读读、读写、写写的过程都是互斥的。对于ReadWriteLock而言，读锁是共享锁，写锁是独享锁，读锁的共享锁可以保证并发读是非常高效的，在读写锁中，读读不互斥、读写、写写的过程是互斥的。独享锁与共享锁也是通过AQS来实现的，通过实现不同的方法，来实现独享或者共享。

**读写锁ReentrantReadWriteLock的应用场景：**

读写锁：分为读锁和写锁，多个读锁不互斥，读锁与写锁互斥，这是由jvm自己控制的，我们只要上好相应的锁即可。如果你的代码只读数据，可以很多人同时读，但不能同时写，那就上读锁；如果你的代码修改数据，只能有一个人在写，且不能同时读取，那就上写锁。总之，读的时候上读锁，写的时候上写锁！

ReentrantReadWriteLock会使用两把锁来解决问题，一个读锁，一个写锁。

**线程进入读锁的前提条件：**

1. 没有其他线程的写锁
2. 没有写请求，或者有写请求但调用线程和持有锁的线程是同一个线程

**进入写锁的前提条件：**

       　1. 没有其他线程的读锁
       　2. 没有其他线程的写锁


ReentrantReadWriteLock的javaodoc文档中提供给我们的一个很好的Cache实例代码案例：

```java
class CachedData {
  Object data;  //缓存的数据
  volatile boolean cacheValid;  //缓存是否有效
  final ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();

  public void processCachedData() {
    rwl.readLock().lock();
    if (!cacheValid) {
      // 再加写锁之前必须先释放读锁，因为进入写锁的条件是没有其他线程的读锁和写锁
      rwl.readLock().unlock();
      rwl.writeLock().lock();
      try {
        // 类似单例模式的DCL双重检查，防止其他线程先拿到写锁对数据进行了缓存，因此要再判断一次
        if (!cacheValid) {
          data = ...
          cacheValid = true;
        }
        // 在释放写锁之前通过获取读锁降级写锁，防止释放写锁后立即被其他线程加上写锁，导致读取脏数据
        rwl.readLock().lock();
      } finally {
        rwl.writeLock().unlock(); // 释放写锁而此时已经持有读锁
      }
    }

    try {
      use(data);
    } finally {
      rwl.readLock().unlock();
    }
  }
}
```


> 参考：
>
> https://www.cnblogs.com/liang1101/p/6475555.html?utm_source=itdadao&utm_medium=referral

## 4. 乐观锁和悲观锁

#### 悲观锁：

总是假设最坏的情况，每次拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想要拿到它的数据就会被一直阻塞直到它拿到锁，传统的关系型数据库里面就用到了很多这种锁机制，比如行锁、表锁、读锁、写锁等，都是在操作之前先上锁。**再比如java里面的synchronized关键字的实现也是悲观锁。**

#### 乐观锁：

顾名思义，很乐观，每次拿数据的时候都认为别人不会修改，**所以不会上锁**，但是在更新的时候会去判断一下别人有没有修改这个数据，可以使用版本号等机制。乐观锁适用于多读的应用场景，这样可以提高吞吐量，像数据库提供的类似于write_condition机制就是提供的乐观锁。**在Java中java.util.concurrent.atomic包下面的原子变量类就是使用了乐观锁的一种实现方式CAS实现的。**

#### 4.1 悲观锁的缺点

悲观锁通过加锁的方式限制其他人对数据的操作，而乐观锁不会加锁，也就放宽了别人对数据的访问。使用悲观锁会引发一些问题：

- 在多线程竞争下，加锁、释放锁会造成比较多的上下文切换和调度延时，引起性能问题；
- 一个线程持有锁，会导致其他所有需要此锁的线程挂起；
- 如果一个优先级高的线程等待一个优先级底的线程的锁，会导致优先级倒置，引起性能风险。

对比于悲观锁的这些问题，一个有效的方式就是乐观锁。其实乐观锁就是：每次不加锁而是假设没有并发冲突而去完成某项操作，如果因为并发冲突失败就重试，直到成功为止。

#### 4.2 乐观锁的一种实现方式：CAS

CAS的全称是Compare And Swap，比较和替换。CAS操作包括三个操作数内存位置（V）、进行比较的预期原值（A）和拟写入的新值（B）。如果内存位置V的值与预期原值A相匹配，那么处理器会自动将该位置值更新为新值B。否则处理器不做任何操作。一般配合死循环来不断尝试更新值，直到成功。

相对于synchronized这种阻塞算法，CAS是一种非阻塞算法的常用实现。

**CAS的缺点**

1. **ABA问题：**意思是说当一个线程获取当前的值是A，此时另一个线程先将A变成B，再变成A，之前的线程继续执行，发现值没变还是A，就继续执行更新操作。这样可能会引发一些潜在问题，问题实例可以参考引用2。JDK的atomic包里提供了一个类AtomicStampedReference来解决ABA问题。
2. **循环时间开销大：**不成功就会一直循环直到成功，如果长时间不成功会给CPU带来非常大的执行开销。如果JVM支持pause指令那么可以一定程度上减少开销。
3. **只能保证一个共享变量的原子操作：**当对一个共享变量执行操作时，我们可以使用循环CAS的方式来保证原子操作，但是对多个共享变量操作时，循环CAS就无法保证操作的原子性，这个时候就可以用锁，或者有一个取巧的办法，就是把多个共享变量合并成一个共享变量来操作。比如有两个共享变量i＝2,j=a，合并一下ij=2a，然后用CAS来操作ij。从Java1.5开始JDK提供了**AtomicReference类来保证引用对象之间的原子性，你可以把多个变量放在一个对象里来进行CAS操作。**

**CAS和synchronized的使用场景：**　　　

- 对于资源竞争较少（线程冲突较轻）的情况，使用synchronized同步锁进行线程阻塞和唤醒切换以及用户态内核态间的切换操作额外浪费消耗cpu资源；而CAS基于硬件实现，不需要进入内核，不需要切换线程，操作自旋几率较少，因此可以获得更高的性能。
- 对于资源竞争严重（线程冲突严重）的情况，CAS自旋的概率会比较大，从而浪费更多的CPU资源，效率低于synchronized。

补充： synchronized在jdk1.6之后，已经改进优化。synchronized的底层实现主要依靠Lock-Free的队列，基本思路是自旋后阻塞，竞争切换后继续竞争锁，稍微牺牲了公平性，但获得了高吞吐量。在线程冲突较少的情况下，可以获得和CAS类似的性能；而线程冲突严重的情况下，性能远高于CAS。

**总结一下就是：**线程冲突小的情况下使用CAS，线程冲突多的情况下使用synchronized。

> 参考：
>
> 乐观锁和悲观锁：https://www.cnblogs.com/qjjazry/p/6581568.html

## 5. 分段锁





## 6. 偏向锁/轻量级锁/重量级锁 



## 7. 自旋锁 

在Java中，自旋锁是指尝试获取锁的线程不会立即阻塞，而是采用循环的方式去尝试获取锁，这样的好处是减少线程上下文切换的消耗，缺点是循环会消耗CPU。





