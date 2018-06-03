# AtomicInteger的原理

java的并发原子包里面提供了很多可以进行原子操作的类，比如：

- AtomicInteger
- AtomicBoolean
- AtomicLong
- AtomicReference

等等，一共分为四类：原子更新基本类型（3个）、原子更新数组、原子更新引用和原子更新属性（字段）。、提供这些原子类的目的就是为了解决基本类型操作的非原子性导致在多线程并发情况下引发的问题。那么非原子性的操作会引发什么问题呢？下面我们通过一个示例来看一下。

## 1. i++引发的问题

我们知道基本类型的赋值操作是原子操作，但是类似这种`i++`的操作并不是原子操作，通过反编译代码我们可以大致了解此操作分为三个阶段：

```java
tp1 = i;  //1
tp2 = tp1 + 1;  //2
i = tp2;  //3
```

如果有两个线程m和n要执行i++操作，因为重排序的影响，代码执行顺序可能会发生改变。如果代码的执行顺序是m1 - m2 - m3 - n1 - n2 - n3，那么结果是没问题的，如果代码的执行顺序是m1 - n1 - m2 - n2 - m3 - n3那么很明显结果就会出错。

#### 测试代码

```java
package com.wangjun.thread;

public class AtomicIntegerTest {
	
	private static int n = 0;

	public static void main(String[] args) throws InterruptedException {
      	//i++引发的线程问题
		Thread t1 = new Thread() {
			public void run() {
				for(int i = 0; i < 1000; i++) {
					n++;
				}
			}; 
		};
		Thread t2 = new Thread() {
			public void run() {
				for(int i = 0; i < 1000; i++) {
					n++;
				}
			};
		};
		t1.start();
		t2.start();
		t1.join();
		t2.join();
		System.out.println("最终n的值为：" + n);
	}
}
```

如果i++是原子操作，那么结果应该就是2000，反复运行几次发现结果大部分情况下都不是2000，这也证明了i++的非原子性在多线程下产生的问题。当然我们可以通过加锁的方式保证操作的原子性，但本文的重点是使用原子类的解决这个问题。

```
最终n的值为：1367
---
最终n的值为：1243
---
最终n的值为：1380
```

## 2. AtomicInteger的原子操作

上面的问题可以使用AtomicInteger来解决，我们更改一下代码如下：

```java
package com.wangjun.thread;

import java.util.concurrent.atomic.AtomicInteger;

public class AtomicIntegerTest {
	
	private static AtomicInteger n2 = new AtomicInteger(0);

	public static void main(String[] args) throws InterruptedException {
      	Thread t1 = new Thread() {
			public void run() {
				for(int i = 0; i < 1000; i++) {
					n2.incrementAndGet();
				}
			}; 
		};
		Thread t2 = new Thread() {
			public void run() {
				for(int i = 0; i< 1000; i++) {
					n2.incrementAndGet();
				}
			}
		};
		t1.start();
		t2.start();
		t1.join();
		t2.join();
		System.out.println("最终n2的值为：" + n2.toString());
	}
}
```

多次运行，发现结果永远是2000，由此可以证明AtomicInteger的操作是原子性的。

```
最终n2的值为：2000
```

那么AtomicInteger是通过什么机制来保证原子性的呢？接下来，我们对源码进行一下分析。

## 3. AtomicInteger源码分析

#### 构造函数

```java
private volatile int value;
/*
 * AtomicInteger内部声明了一个volatile修饰的变量value用来保存实际值
 * 使用带参的构造函数会将入参赋值给value，无参构造器value默认值为0
 */
public AtomicInteger(int initialValue) {
  value = initialValue;
}
```

#### 自增函数

```java
import sun.misc.Unsafe;
private static final Unsafe unsafe = Unsafe.getUnsafe();
private static final long valueOffset;
static {
  try {
    valueOffset = unsafe.objectFieldOffset
      (AtomicInteger.class.getDeclaredField("value"));
  } catch (Exception ex) { throw new Error(ex); }
}
/*
 * 可以看到自增函数中调用了Unsafe函数的getAndAddInt方法
 */
public final int incrementAndGet() {
  return unsafe.getAndAddInt(this, valueOffset, 1) + 1;
}

```

那么这个getAndAddInt方法是干嘛的呢，首先来了解一下Unsafe这个类。

```
Unsafe类是在sun.misc包下，不属于Java标准。但是很多Java的基础类库，包括一些被广泛使用的高性能开发库都是基于Unsafe类开发的，比如Netty、Cassandra、Hadoop、Kafka等。Unsafe类在提升Java运行效率，增强Java语言底层操作能力方面起了很大的作用。
Unsafe类使Java拥有了像C语言的指针一样操作内存空间的能力，同时也带来了指针的问题。过度的使用Unsafe类会使得出错的几率变大，因此Java官方并不建议使用的，官方文档也几乎没有。
通常我们最好也不要使用Unsafe类，除非有明确的目的，并且也要对它有深入的了解才行。
```

再来说Unsafe的getAndAddInt，通过反编译可以看到实现代码：

```java
/*
 * 其中getIntVolatile和compareAndSwapInt都是native方法
 * getIntVolatile是获取当前的期望值
 * compareAndSwapInt就是我们平时说的CAS(compare and swap)，通过比较如果内存区的值没有改变，那么就用新值直接给该内存区赋值
 */
public final int getAndAddInt(Object paramObject, long paramLong, int paramInt)
{
  int i;
  do
  {
    i = getIntVolatile(paramObject, paramLong);
  } while (!compareAndSwapInt(paramObject, paramLong, i, i + paramInt));
  return i;
}
```

`incrementAndGet`是将自增后的值返回，还有一个方法`getAndIncrement`是将自增前的值返回，分别对应`++i`和`i++`操作。同样的`decrementAndGe`t和`getAndDecrement`则对`--i`和`i--`操作。

## 4. CAS中ABA问题的解决

CAS也并非完美的，它会导致ABA问题，就是说，当前内存的值一开始是A，被另外一个线程先改为B然后再改为A，那么当前线程访问的时候发现是A，则认为它没有被其他线程访问过。在某些场景下这样是存在错误风险的。比如在链表中。

那么如何解决这个ABA问题呢，大多数情况下乐观锁的实现都会通过引入一个版本号标记这个对象，每次修改版本号都会变话，比如使用时间戳作为版本号，这样就可以很好的解决ABA问题。

在JDK中提供了AtomicStampedReference类来解决这个问题，思路是一样的。这个类也维护了一个int类型的标记stamp，每次更新数据的时候顺带更新一下stamp。

**下面我们通过代码演示来看一下AtomicStampedReference的使用：**

```java
package com.wangjun.thread;

import java.util.concurrent.atomic.AtomicInteger;
import java.util.concurrent.atomic.AtomicStampedReference;

public class ABA {
	
	// 普通的原子类，存在ABA问题
	AtomicInteger a1 = new AtomicInteger(10);
	// 带有时间戳的原子类，不存在ABA问题，第二个参数就是默认时间戳，这里指定为0
	AtomicStampedReference<Integer> a2 = new AtomicStampedReference<Integer>(10, 0);
	
	public static void main(String[] args) {
		ABA a = new ABA();
		a.test();
	}
	
	public void test() {
		new Thread1().start();
		new Thread2().start();
		new Thread3().start();
		new Thread4().start();
	}
	
	class Thread1 extends Thread {
		@Override
		public void run() {
			a1.compareAndSet(10, 11);
			a1.compareAndSet(11, 10);
		}
	}
	class Thread2 extends Thread {
		@Override
		public void run() {
			try {
				Thread.sleep(200);  // 睡0.2秒，给线程1时间做ABA操作
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			System.out.println("AtomicInteger原子操作：" + a1.compareAndSet(10, 11));
		}
	}
	class Thread3 extends Thread {
		@Override
		public void run() {
			try {
				Thread.sleep(500);  // 睡0.5秒，保证线程4先执行
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			int stamp = a2.getStamp();
			a2.compareAndSet(10, 11, stamp, stamp + 1);
			stamp = a2.getStamp();
			a2.compareAndSet(11, 10, stamp, stamp + 1);
		}
	}
	class Thread4 extends Thread {
		@Override
		public void run() {
			int stamp = a2.getStamp();
			try {
				Thread.sleep(1000);  // 睡一秒，给线程3时间做ABA操作
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			System.out.println("AtomicStampedReference原子操作:" + a2.compareAndSet(10, 11, stamp, stamp + 1));
		}
	}
}
```

可以看到使用AtomicStampedReference进行compareAndSet的时候，除了要验证数据，还要验证时间戳。

如果数据一样，但是时间戳不一样，那么这个数据其实也被修改过了。

> 参考：
>
> java的Unsafe类：https://www.cnblogs.com/pkufork/p/java_unsafe.html
>
> Java CAS 和ABA问题https://www.cnblogs.com/549294286/p/3766717.html