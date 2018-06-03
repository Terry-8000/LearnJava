# Java中的阻塞队列

### 1. 什么是阻塞队列

阻塞队列是支持两个附加操作的队列。这两个附加操作就是阻塞式的插入和移除方法。

1. **支持阻塞的插入方法：**当队列满时，队列会阻塞插入元素的线程，直到队列不满；
2. **支持阻塞的移除方法：**当队列为空时，队列会阻塞获取元素的线程，直到队列非空。

在阻塞队列不可用时，这两个附加操作提供了4种处理方式：

|      | 抛出异常      | 返回特殊值    | 一直阻塞   | 超时退出                 |
| ---- | --------- | -------- | ------ | -------------------- |
| 插入   | add(e)    | offer(e) | put(e) | offer(e, time, unit) |
| 移除   | remove()  | poll()   | take() | poll(time, unit)     |
| 检查   | element() | peek()   | 不可用    | 不可用                  |

下面，解释一下这四种情况具体如何处理：

- **抛出异常：**队列满时，再次插入会抛出IllegalStateException（Queue full）异常。队列空时，再次获取元素会抛出NoSuchElementException异常；
- **返回特殊值：**当往队列插入元素时，会返回元素是否插入成功，成功返回true。如果是移除方法，则是从队列里取出一个元素，如果没有返回null；
- **一直阻塞：**队列满时，如果生产者线程继续put元素，队列就会一直阻塞生产者线程，直到队列可用或者响应中断退出。当队列为空时，如果消费者线程继续take元素，那么队列会阻塞消费者线程直到队列不为空；
- **超时退出：**阻塞队列满时，如果生产者线程往队列里插入元素，队列会阻塞生产者线程一段时间，如果超出了这个时间，生产者线程就会退出。

## 2. Java里面的阻塞队列

JDK7提供了7个阻塞队列。

### 2.1 ArrayBlockingQueue

一个由数组结构组成的有界阻塞队列。按照FIFO原则对元素排序。默认情况下不保证线程公平的访问队列，即不保证先阻塞的线程先访问队列。可以通过构造器传入参数构建一个公平的阻塞队列。访问者的公平性是通过可重入锁实现的。

### 2.2 LinkedBlockingQueue

一个由链表实现的有界阻塞队列。默认最大长度Integer.MAX_VALUE。按照FIFO对元素进行排序。

### 2.3 PriorityBlockingQueue

一个支持优先级的无界阻塞队列，默认情况下采用自然排序升序排列，也可以自定义排序规则。

### 2.4 DelayQueue

一个支持延时获取元素的无界阻塞队列。队列使用PriorityQueue实现。

### 2.5 SynchronousQueue

不存储元素的阻塞队列。每一个put操作必须等待一个take操作，否则不能添加元素。支持公平访问队列。

### 2.6 LinkedTransferQueue

由链表结构组成的无界阻塞队列，比其他阻塞队列多了一个tryTransfer和transfer方法。

### 2.7 LinkedBlockingDeque

链表结构组成的双向阻塞队列。

## 3. ArrayBlockingQueue实现原理 

阻塞队列是通过**通知模式**实现生产者和消费者之间的通信的，当生产者向一个满的队列put数据的时候会被阻塞，当消费者消费了一个队列元素后，会通知生产者当前队列可用。看一下源码：

#### 构造器：

```java
final Object[] items;  // 存放队列元素的数组
private final Condition notEmpty;  // 等待take的Condition
private final Condition notFull;  // 等待put的Condition
final ReentrantLock lock;  // 可重入锁

public ArrayBlockingQueue(int capacity) {
  this(capacity, false);
}
public ArrayBlockingQueue(int capacity, boolean fair) {
  if (capacity <= 0)
    throw new IllegalArgumentException();
  this.items = new Object[capacity];  // 初始化存放队列的数组
  lock = new ReentrantLock(fair);  // fair:true 公平锁  fair:false 非公平锁(默认)
  notEmpty = lock.newCondition();
  notFull =  lock.newCondition();
}
```

#### 存储元素

```java
public void put(E e) throws InterruptedException {
  checkNotNull(e);
  final ReentrantLock lock = this.lock;
  lock.lockInterruptibly();  // 如果当前线程没有被打断，则获取锁
  try {
    while (count == items.length)  // 如果当前队列已满，则阻塞生产者
      notFull.await();
    enqueue(e);  // 元素入队
  } finally {
    lock.unlock();
  }
}

private void enqueue(E x) {
  final Object[] items = this.items;
  items[putIndex] = x;
  if (++putIndex == items.length)
    putIndex = 0;
  count++;
  notEmpty.signal();  // 队列有元素了，通知消费者你可以来取了
}
```

#### 取出元素

```java
public E take() throws InterruptedException {
  final ReentrantLock lock = this.lock;
  lock.lockInterruptibly();  // 如果当前线程没有被打断，则获取锁
  try {
    while (count == 0)  // 如果当前队列已空，则阻塞消费者
      notEmpty.await();
    return dequeue();  // 元素出队
  } finally {
    lock.unlock();
  }
}

private E dequeue() {
  final Object[] items = this.items;
  @SuppressWarnings("unchecked")
  E x = (E) items[takeIndex];
  items[takeIndex] = null;
  if (++takeIndex == items.length)
    takeIndex = 0;
  count--;
  if (itrs != null)
    itrs.elementDequeued();
  notFull.signal();  // 刚取出了一个元素，队列肯定不为空，通知生产者你可以来放入元素了
  return x;
}
```

从源码可以看到生产者放入元素的时候，如果队列已满，则阻塞生产者放入元素，直到有消费者消费了队列元素就通知生产者可以放入了。同理，当消费者取出元素的时候，如果队列为空，则阻塞消费者，直到生产者放入了元素则通知消费者你可以继续取元素了。这是一个典型的**通知模式**。



> 参考：
>
> 《Java并发编程的艺术》