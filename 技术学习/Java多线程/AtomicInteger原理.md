# AtomicInteger的原理

java的并发原子包里面提供了很多可以进行原子操作的类，比如：

- AtomicInteger
- AtomicBoolean
- AtomicLong
- AtomicReference

等等，一共分为四类：原子更新基本类型（3个）、原子更新数组、原子更新引用和原子更新属性（字段）。、提供这些原子类的目的就是为了解决基本类型操作的非原子性导致在多线程并发情况下引发的问题。那么非原子性的操作会引发什么问题呢？下面我们通过一个示例来看一下。

## i++引发的问题

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

## AtomicInteger的原子操作

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

## AtomicInteger源码分析

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

那么这个getAndAddInt方法是干嘛的呢，首先来了解一下Unsafe这个类，