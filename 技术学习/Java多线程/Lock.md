# Lock简介

Lock提供了比synchronized方法和synchronized代码块更广泛的锁定操作，Lock允许更灵活的结构，可以具有差别很大的属性。Lock和ReadWriteLock是java5提供的两个接口，并为Lock提供了ReentrantLock实现类，为ReadWriteLock提供了ReentrantReadWriteLock实现类。

Lock比传统线程模型中的synchronized方式更加面向对象，与生活中的锁类似，锁本身也应该是一个对象。两个线程执行的代码片段要实现同步互斥的效果，它们必须用同一个Lock对象。

以下是ReentrantLock的基本用法：

```java
//服务类
class MyService{
  ReentrantLock lock = new ReentrantLock();
  public void printInt() {
    lock.lock();//加锁
    for(int i = 0; i < 5; i++) {
      System.out.println(Thread.currentThread().getName() + ":" + i);
    }
    lock.unlock();//释放锁
  }
}
```

