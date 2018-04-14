# Java实现线程的三种方式和区别

Java实现线程的三种方式：

1. 继承Thread
2. 实现Runnable接口
3. 实现Callable接口

区别：

1. 第一种方式继承Thread就不能继承其他类了，后面两种可以；
2. 使用后两种方式可以多个线程共享一个target；
3. Callable比Runnable多一个返回值，并且call\(\)方法可以抛出异常；
4. 访问线程名，第一种直接使用this.getName\(\)，后两种使用Thread.currentThread\(\).getName\(\)。

下面我们通过代码来看一下实现和区别：

三种实现：

```java
//1. 继承Thread，重写run()方法
class Thread1 extends Thread {

    private int n = 5;

    @Override
    public void run() {
        while(n > 0) {
            System.out.println("name:" + this.getName() + ", n:" + n);
            n--;
        }
    }
}
//2. 实现Runnable接口，实现run()方法
class Thread2 implements Runnable {

    private int n = 5; 

    @Override
    public void run() {
        while(n > 0) {
            System.out.println("name:" + Thread.currentThread().getName() + ", n:" + n);
            n--;
        }
    }
}
//3. 实现Callable接口，实现call()方法，带有返回值和异常
class Thread3 implements Callable<String> {

    private int n = 5;

    @Override
    public String call() throws Exception {
        while(n > 0) {
            System.out.println("name:" + Thread.currentThread().getName() + ", n:" + n);
            n--;
        }
        return String.valueOf(n);
    }

}
```

如何使用：

```java
//第一种实现方式
Thread1 t11 = new Thread1();
Thread1 t12 = new Thread1();
Thread1 t13 = new Thread1();

t11.start();
t12.start();
t13.start();

//第二种实现方式
Thread2 t21 = new Thread2();
Thread2 t22 = new Thread2();
Thread2 t23 = new Thread2();

Thread t211 = new Thread(t21);
Thread t212 = new Thread(t22);
Thread t213 = new Thread(t23);

t211.start();
t212.start();
t213.start();

//第三种实现
Thread3 t31 = new Thread3();
Thread3 t32 = new Thread3();
Thread3 t33 = new Thread3();

FutureTask<String> f1 = new FutureTask<>(t31);
FutureTask<String> f2 = new FutureTask<>(t32);
FutureTask<String> f3 = new FutureTask<>(t33);

Thread t311 = new Thread(f1);
Thread t312 = new Thread(f2);
Thread t313 = new Thread(f3);

t311.start();
t312.start();
t313.start();
```

从代码可以看出以上提到的区别1，3，4。那么区别2共享一个target是什么意思呢？

首先我们看一下上述代码的运行结果，

第一种：

```java
name:Thread-1, n:5
name:Thread-1, n:4
name:Thread-1, n:3
name:Thread-1, n:2
name:Thread-1, n:1
name:Thread-2, n:5
name:Thread-2, n:4
name:Thread-2, n:3
name:Thread-2, n:2
name:Thread-2, n:1
name:Thread-0, n:5
name:Thread-0, n:4
name:Thread-0, n:3
name:Thread-0, n:2
name:Thread-0, n:1
```

第二种：

```java
name:Thread-4, n:5
name:Thread-4, n:4
name:Thread-4, n:3
name:Thread-3, n:5
name:Thread-5, n:5
name:Thread-3, n:4
name:Thread-4, n:2
name:Thread-4, n:1
name:Thread-3, n:3
name:Thread-3, n:2
name:Thread-3, n:1
name:Thread-5, n:4
name:Thread-5, n:3
name:Thread-5, n:2
name:Thread-5, n:1
```

可以看到，这两种方式的结果一样，都是new了三个线程，每个线程内部循环5次。d第二种方式并没有体现共用同一个target。如果我们将第二种创建线程的方式改为：

```java
//第二种实现方式
Thread2 t21 = new Thread2();
Thread2 t22 = new Thread2();
Thread2 t23 = new Thread2();

Thread t211 = new Thread(t21);
Thread t212 = new Thread(t21);
Thread t213 = new Thread(t21);

t211.start();
t212.start();
t213.start();
```

看一下运行结果：

```java
name:Thread-4, n:5
name:Thread-4, n:4
name:Thread-4, n:3
name:Thread-4, n:2
name:Thread-4, n:1
name:Thread-3, n:5
name:Thread-5, n:5
```

可以看到，虽然也启动了3个线程，但是由于共享一个target，n的值改变了，其他两个线程也会知道，所以因此一共循环了5次。但是这里明明是7次啊，这是由于多线程的同步问题，可以给run方法加上synchronized关键字解决：

```java
@Override
public synchronized void run() {
  while(n > 0) {
    System.out.println("name:" + Thread.currentThread().getName() + ", n:" + n);
    n--;
  }
}
```

运行结果：

```java
name:Thread-3, n:5
name:Thread-3, n:4
name:Thread-3, n:3
name:Thread-3, n:2
name:Thread-3, n:1
```

这里可能有点迷惑，只启动了一个线程啊。其实另外两个线程也启动了，只是这个时候n=0无法进入循环。我们可以加一行打印：

```java
@Override
public synchronized void run() {
  System.out.println("进入" + Thread.currentThread().getName() + "线程");
  while(n > 0) {
    System.out.println("name:" + Thread.currentThread().getName() + ", n:" + n);
    n--;
  }
}
```

可以看到运行结果：

```
进入Thread-3线程
name:Thread-3, n:5
name:Thread-3, n:4
name:Thread-3, n:3
name:Thread-3, n:2
name:Thread-3, n:1
进入Thread-5线程
进入Thread-4线程
```



