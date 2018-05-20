# 各种Lock

Java中锁的种类有很多：公平锁/非公平锁、可重入锁/不可重入锁、独享锁/共享锁、互斥锁/读写锁、乐观锁/悲观锁、分段锁、偏向锁/轻量级锁/重量级锁、自旋锁。

### ReentrantLock类

Jdk1.5新增的ReentrantLock类和synchronized关键字一样可以实现线程间同步互斥，但是它在拓展功能上更加强大，比如**嗅探锁定**、**多路分支**等功能，使用的时候也比synchronized更加灵活。

使用Condition对象可以实现类似synchronized的wait()/notify()/notifyAll()同样的功能。Condition有更好的灵活性，比如可以实现多路通知功能，也就是在一个Lock对象中创建多个Condition（即对象监视器）实例，线程对象可以注册在指定的Condition中，从而可以有选择性的进行线程通知，在调度线程上更加灵活。

在使用notify()/notifyAll()方法进行通知时，被通知的线程是由JVM随机选择的，但是使用ReentrantLock和Condition可以实现“选择性通知”，这个功能是非常重要的，而且在Condition类时默认提供的。

而synchronized就相当于整个Lock对象中只有一个单一的Condition对象，所有的线程都注册在它一个对象身上，线程开始notifyAll时，需要通知所有的waiting线程，没有选择权，会出现相当大的效率问题。

**signal()和signalAll()区别**

signalAll通知所有使用了同一个Condition对象的线程。signal()通知所有使用了Condition对象的某一个线程，通过源码可以看到通知的用时是位于队首的那个。

### 1. 公平锁和非公平锁

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

### 2. 可重入锁和不可重入锁

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

### 3. 独享锁和共享锁

独享锁是该锁只能被一个线程持有，共享锁是该锁可以被多个线程持有；

对于ReentrantLock和Synchronized而言，是独享锁。对于ReadWriteLock而言，读锁是共享锁，写锁是独享锁。读锁的共享锁可以保证并发读是非常高效的，读写、写读、写写的过程是互斥的。独享锁与共享锁也是通过AQS来实现的，通过实现不同的方法，来实现独享或者共享。

**互斥锁和读写锁**

独享锁/共享锁就是一种广义的说法，互斥锁/读写锁就是具体的实现。

互斥锁在Java中的具体实现就是ReentrantLock；

读写锁在Java中的具体实现就是ReadWriteLock。



load…..



### 4. 乐观锁和悲观锁





### 5. 分段锁





### 6. 偏向锁/轻量级锁/重量级锁 



### 7. 自旋锁 

在Java中，自旋锁是指尝试获取锁的线程不会立即阻塞，而是采用循环的方式去尝试获取锁，这样的好处是减少线程上下文切换的消耗，缺点是循环会消耗CPU。





