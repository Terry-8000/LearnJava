# 各种Lock

Java中锁的种类有很多：公平锁/非公平锁、可重入锁、独享锁/共享锁、互斥锁/读写锁、乐观锁/悲观锁、分段锁、偏向锁/轻量级锁/重量级锁、自旋锁。

### ReentrantLock类

Jdk1.5新增的ReentrantLock类和synchronized关键字一样可以实现线程间同步互斥，但是它在拓展功能上更加强大，比如**嗅探锁定**、**多路分支**等功能，使用的时候也比synchronized更加灵活。

使用Condition对象可以实现类似synchronized的wait()/notify()/notifyAll()同样的功能。Condition有更好的灵活性，比如可以实现多路通知功能，也就是在一个Lock对象中创建多个Condition（即对象监视器）实例，线程对象可以注册在指定的Condition中，从而可以有选择性的进行线程通知，在调度线程上更加灵活。

在使用notify()/notifyAll()方法进行通知时，被通知的线程是由JVM随机选择的，但是使用ReentrantLock和Condition可以实现“选择性通知”，这个功能是非常重要的，而且在Condition类时默认提供的。

而synchronized就相当于整个Lock对象中只有一个单一的Condition对象，所有的线程都注册在它一个对象身上，线程开始notifyAll时，需要通知所有的waiting线程，没有选择权，会出现相当大的效率问题。

**signal()和signalAll()区别**

signalAll通知所有使用了同一个Condition对象的线程。signal()通知所有使用了Condition对象的某一个线程，通过源码可以看到通知的用时是位于队首的那个。

**公平锁和非公平锁**

公平锁表示线程获取锁顺序是按照线程加锁的顺序来分配的，即FIFO顺序。而非公平锁就是一种获取锁的抢占机制，是随机获得锁的。

