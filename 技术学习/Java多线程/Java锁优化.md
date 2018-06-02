# Java锁优化

应用程序在并发环境下会产生很多问题，通常情况下，我们可以通过加锁来解决多线程对临界资源的访问问题。但是加锁往往会成为系统的瓶颈，因为加锁和释放锁会涉及到与操作系统的交互，会有很大的性能问题。那么这个时候基于锁的优化手段就显得很重要了。

一般情况下，可以从两个角度进行锁优化：对单个锁算法的优化和对锁粒度的细分。

## 1. 单个锁的优化

#### 自旋锁：

​	非自旋锁在未获取锁的情况会被阻塞，之后再唤醒尝试获得锁。而JDK的阻塞和唤醒是基于操作系统实现的，会有系统资源的开销。自旋锁就是线程不停地循环尝试获得锁，而不会将自己阻塞，这样不会浪费系统的资源开销，但是会浪费CPU的资源。所有现在的JDK大部分都是先自旋等待，如果自旋等待一段时间之后还没有获取到锁，就会将当前线程阻塞。

#### 锁消除：

​	当JVM分析代码时发现某个方法只被单个线程安全访问，而且这个方法是同步方法，那么JVM就会去掉这个方法的锁。

#### 单个锁优化的瓶颈：

​	对单个锁优化的效果就像提高单个CPU的处理能力一样，最终会由于各个方面的限制而达到一个平衡点，到达这个点之后优化单个锁的对高并发下面锁的优化效果越来越低。所以将一个锁进行粒度细分带来的效果会很明显，如果一个锁保护的代码块被拆分成两个锁来保护，那么程序的效率就大约能够提高到2倍，这个比单个锁的优化带来的效果要明显很多。常见的锁粒度细分技术有：锁分解和锁分段

## 2. 细分锁粒度

细分锁粒度的目的是降低竞争锁的概率。

### 2.1 锁分解

锁分解的核心是将无关的代码块，如果在一个方法中有一部分的代码与锁无关，一部分的代码与锁有关，那么可以缩小这个锁的返回，这样锁操作的代码块就会减少，锁竞争的可能性也会减少

#### 缩小锁的范围

缩小锁的范围是指尽量只在必要的地方加锁，不要扩大加锁的范围，就拿单例模式举例，范围大的锁可能将整个方法都加锁了：

```java
class Singleton {
  private Singleton instance;

  private Singleton() {
  }

  // 将整个方法加锁
  public synchronized Singleton getInstance() {
    try {
      Thread.sleep(1000);  //do something
      if(null == instance)
        instance = new Singleton();
    } catch (InterruptedException e) {
      e.printStackTrace();
    }

    return instance;
  }

}
```

优化后的，只将部分代码加锁：

```java
class Singleton {
  private Singleton instance;

  private Singleton() {
  }

  public Singleton getInstance() {
    try {
      Thread.sleep(1000);  //do something
      // 只对部分代码加锁
      synchronized(this) {
        if(null == instance)
          instance = new Singleton();
      }
    } catch (InterruptedException e) {
      e.printStackTrace();
    }

    return instance;
  }
}
```

#### 减少锁的粒度

减少锁的粒度是指如果一个锁需要保护多个相互独立的变量，那么可以将一个锁分解为多个锁，并且每个锁保护一个变量，这样就可以减少锁冲突。看一下下面的例子：

```java
class Demo{
  private Set<String> allUsers = new HashSet<String>();
  private Set<String> allComputers = new HashSet<String>();

  //公用一把锁
  public synchronized void addUser(String user){ 
    allUsers.add(user);
  }
  
  public synchronized void addComputer(String computer){
    allComputers.add(computer);
  }
}
```

缩小锁的粒度后，将一个锁拆分为多个：

```java
class Demo{
  private Set<String> allUsers = new HashSet<String>();
  private Set<String> allComputers = new HashSet<String>();
  
  //分解为两把锁
  public void addUser(String user){ 
    synchronized (allUsers){
      allUsers.add(user);
    }
  }
  
  public void addComputer(String computer){
    synchronized (allComputers){
      allComputers.add(computer);
    }
  }
}
```

如上的方法把一个锁分解为2个锁时候，采用两个线程时候，大约能够使程序的效率提升一倍。

### 2.2 锁分段

锁分段和缩小锁的粒度类似，就是将锁细分的粒度更多，比如将一个数组的每个位置当做单独的锁。JDK8以前ConcurrentHashMap就使用了锁分段技术，它将散列数组分成多个Segment，每个Segment存储了实际的数据，访问数据的时候只需要对数据所在的Segment加锁就行。




> 参考：
>
> Java锁分解锁分段技术： http://guochenglai.com/2016/06/04/java-concurrent4-java-subsection-decompose/
>
> ConcurrentHashMap的锁分段技术：https://blog.csdn.net/yansong_8686/article/details/50664351