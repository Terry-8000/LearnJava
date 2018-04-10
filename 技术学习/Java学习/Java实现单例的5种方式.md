# Java实现单例的5种方式

### 1. 什么是单例模式

单例模式指的是在应用整个生命周期内只能存在一个实例。单例模式是一种被广泛使用的设计模式。他有很多好处，能够避免实例对象的重复创建，减少创建实例的系统开销，节省内存。   
单例模式的要求有三点：

* 某个类只能有一个实例
* 它必须自行创建这个实例
* 他必须自行向整个系统提供整个实例

### 2. 单例模式和静态类的区别

首先理解一下什么是静态类，静态类就是一个类里面都是静态方法和静态field，构造器被private修饰，因此不能被实例化。Math类就是一个静态类。

知道了什么是静态类后，来说一下他们两者之间的区别：

1）首先单例模式会提供给你一个全局唯一的对象，静态类只是提供给你很多静态方法，这些方法不用创建对象，通过类就可以直接调用；

2）单例模式的灵活性更高，方法可以被override，因为静态类都是静态方法，所以不能被override；

3）如果是一个非常重的对象，单例模式可以懒加载，静态类就无法做到；

那么时候时候应该用静态类，什么时候应该用单例模式呢？首先如果你只是想使用一些工具方法，那么最好用静态类，静态类比单例类更快，因为静态的绑定是在编译期进行的。如果你要维护状态信息，或者访问资源时，应该选用单例模式。还可以这样说，当你需要面向对象的能力时（比如继承、多态）时，选用单例类，当你仅仅是提供一些方法时选用静态类。

### 3.如何实现单例模式

**1. 饿汉模式**

所谓饿汉模式就是立即加载，一般情况下再调用getInstancef方法之前就已经产生了实例，也就是在类加载的时候已经产生了。这种模式的缺点很明显，就是占用资源，当单例类很大的时候，其实我们是想使用的时候再产生实例。因此这种方式适合占用资源少，在初始化的时候就会被用到的类。

```java
class SingletonHungary {
    private static SingletonHungary singletonHungary = new SingletonHungary();
    //将构造器设置为private禁止通过new进行实例化
    private SingletonHungary() {

    }
    public static SingletonHungary getInstance() {
        return singletonHungary;
    }
}
```

**2. 懒汉模式**

懒汉模式就是延迟加载，也叫懒加载。在程序需要用到的时候再创建实例，这样保证了内存不会被浪费。针对懒汉模式，这里给出了5种实现方式，有些实现方式是线程不安全的，也就是说在多线程并发的环境下可能出现资源同步问题。

首先第一种方式，在单线程下没问题，在多线程下就出现问题了。

```java
// 单例模式的懒汉实现1--线程不安全
class SingletonLazy1 {
    private static SingletonLazy1 singletonLazy;

    private SingletonLazy1() {

    }

    public static SingletonLazy1 getInstance() {
        if (null == singletonLazy) {
            try {
                // 模拟在创建对象之前做一些准备工作
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            singletonLazy = new SingletonLazy1();
        }
        return singletonLazy;
    }
}
```

我们模拟10个异步线程测试一下：

```java
public class SingletonLazyTest {

    public static void main(String[] args) {

        Thread2[] ThreadArr = new Thread2[10];
        for (int i = 0; i < ThreadArr.length; i++) {
            ThreadArr[i] = new Thread2();
            ThreadArr[i].start();
        }
    }

}

// 测试线程
class Thread2 extends Thread {
    @Override
    public void run() {
        System.out.println(SingletonLazy1.getInstance().hashCode());
    }
}
```

运行结果：

```
124191239
124191239
872096466
1603289047
1698032342
1913667618
371739364
124191239
1723650563
367137303
```

可以看到他们的hashCode不都是一样的，说明在多线程环境下，产生了多个对象，不符合单例模式的要求。

那么如何使线程安全呢？第二种方法，我们使用synchronized关键字对getInstance方法进行同步。

```java
// 单例模式的懒汉实现2--线程安全
// 通过设置同步方法，效率太低，整个方法被加锁
class SingletonLazy2 {
    private static SingletonLazy2 singletonLazy;

    private SingletonLazy2() {

    }

    public static synchronized SingletonLazy2 getInstance() {
        try {
            if (null == singletonLazy) {
                // 模拟在创建对象之前做一些准备工作
                Thread.sleep(1000);
                singletonLazy = new SingletonLazy2();
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return singletonLazy;
    }
}
```

使用上面的测试类，测试结果：

```
1210004989
1210004989
1210004989
1210004989
1210004989
1210004989
1210004989
1210004989
1210004989
1210004989
```

可以看到，这种方式达到了线程安全。但是缺点就是效率太低，是同步运行的，下个线程想要取得对象，就必须要等上一个线程释放，才可以继续执行。

那我们可以不对方法加锁，而是将里面的代码加锁，也可以实现线程安全。但这种方式和同步方法一样，也是同步运行的，效率也很低。

```java
// 单例模式的懒汉实现3--线程安全
// 通过设置同步代码块，效率也太低，整个代码块被加锁
class SingletonLazy3 {

    private static SingletonLazy3 singletonLazy;

    private SingletonLazy3() {

    }

    public static SingletonLazy3 getInstance() {
        try {
            synchronized (SingletonLazy3.class) {
                if (null == singletonLazy) {
                    // 模拟在创建对象之前做一些准备工作
                    Thread.sleep(1000);
                    singletonLazy = new SingletonLazy3();
                }
            }
        } catch (InterruptedException e) {
            // TODO: handle exception
        }
        return singletonLazy;
    }
}
```

我们来继续优化代码，我们只给创建对象的代码进行加锁，但是这样能保证线程安全么？

```java
// 单例模式的懒汉实现4--线程不安全
// 通过设置同步代码块，只同步创建实例的代码
// 但是还是有线程安全问题
class SingletonLazy4 {

    private static SingletonLazy4 singletonLazy;

    private SingletonLazy4() {

    }

    public static SingletonLazy4 getInstance() {
        try {
            if (null == singletonLazy) {        //代码1
                // 模拟在创建对象之前做一些准备工作
                Thread.sleep(1000);
                synchronized (SingletonLazy4.class) {
                    singletonLazy = new SingletonLazy4(); //代码2
                }
            }
        } catch (InterruptedException e) {
            // TODO: handle exception
        }
        return singletonLazy;
    }
}
```

我们来看一下运行结果：

```
1210004989
1425839054
1723650563
389001266
1356914048
389001266
1560241484
278778395
124191239
367137303
```

从结果看来，这种方式不能保证线程安全，为什么呢？我们假设有两个线程A和B同时走到了‘代码1’，因为此时对象还是空的，所以都能进到方法里面，线程A首先抢到锁，创建了对象。释放锁后线程B拿到了锁也会走到‘代码2’，也创建了一个对象，因此多线程环境下就不能保证单例了。

让我们来继续优化一下，既然上述方式存在问题，那我们在同步代码块里面再一次做一下null判断不就行了，这种方式就是我们的DCL双重检查锁机制。 

```java
//单例模式的懒汉实现5--线程安全
//通过设置同步代码块，使用DCL双检查锁机制
//使用双检查锁机制成功的解决了单例模式的懒汉实现的线程不安全问题和效率问题
//DCL 也是大多数多线程结合单例模式使用的解决方案
//第一个if判断的作用：是为了提高程序的 效率，当SingletonLazy5对象被创建以后，再获取SingletonLazy5对象时就不用去验证同步代码块的锁及后面的代码，直接返回SingletonLazy5对象
//第二个if判断的作用：是为了解决多线程下的安全性问题，也就是保证对象的唯一。
class SingletonLazy5 {

    private static volatile SingletonLazy5 singletonLazy;

    private SingletonLazy5() {

    }

    public static SingletonLazy5 getInstance() {
        try {
            if (null == singletonLazy) {
                // 模拟在创建对象之前做一些准备工作
                Thread.sleep(1000);
                synchronized (SingletonLazy5.class) {
                    if(null == singletonLazy) {
                        singletonLazy = new SingletonLazy5();
                    }
                }
            }
        } catch (InterruptedException e) {
            // TODO: handle exception
        }
        return singletonLazy;
    }
}
```

运行结果：

```
124191239
124191239
124191239
124191239
124191239
124191239
124191239
124191239
124191239
124191239
```

我们可以看到DCL双重检查锁机制很好的解决了懒加载单例模式的效率问题和线程安全问题。这也是我们最常用到的方式。

**volatile关键字**

这里注意到在定义singletonLazy的时候用到了volatile关键字，这是为了防止指令重排序的，为什么要这么做呢，我们来看一个场景：

代码走到了 singletonLazy = new SingletonLazy5\(\);看起来是一句话，但这并不是一个原子操作（要么全部执行完，要么全部不执行，不能执行一半），这句话被编译成8条汇编指令，大致做了3件事情：

1.给SingletonLazy5的实例分配内存。

2.初始化SingletonLazy5的构造器

3.将singletonLazy对象指向分配的内存空间（注意到这步instance就非null了）。

由于Java编译器允许处理器乱序执行（out-of-order），以及JDK1.5之前JMM（Java Memory Medel）中Cache、寄存器到主内存回写顺序的规定，上面的第二点和第三点的顺序是无法保证的，也就是说，执行顺序可能是1-2-3也可能是1-3-2，如果是后者，并且在3执行完毕、2未执行之前，被切换到线程二上，这时候singletonLazy因为已经在线程一内执行过了第三点，singletonLazy已经是非空了，所以线程二直接拿走singletonLazy，然后使用，然后顺理成章地报错，而且这种难以跟踪难以重现的错误估计调试上一星期都未必能找得出来。

DCL的写法来实现单例是很多技术书、教科书（包括基于JDK1.4以前版本的书籍）上推荐的写法，实际上是不完全正确的。的确在一些语言（譬如C语言）上DCL是可行的，取决于是否能保证2、3步的顺序。在JDK1.5之后，官方已经注意到这种问题，因此调整了JMM、具体化了volatile关键字，因此如果JDK是1.5或之后的版本，只需要将singletonLazy的定义加上volatile关键字，就可以保证每次都去singletonLazy都从主内存读取，并且可以禁止重排序，就可以使用DCL的写法来完成单例模式。当然volatile或多或少也会影响到性能，最重要的是我们还要考虑JDK1.42以及之前的版本，所以单例模式写法的改进还在继续。

**3. 静态内部类**

基于上面的考虑，我们可以使用静态内部类实现单例模式，代码如下：

```java
//使用静态内部类实现单例模式--线程安全
class SingletonStaticInner {
    private SingletonStaticInner() {

    }
    private static class SingletonInner {
        private static SingletonStaticInner singletonStaticInner = new SingletonStaticInner();
    }
    public static SingletonStaticInner getInstance() {
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        return SingletonInner.singletonStaticInner;
    }
}
```

可以看到使用这种方式我们没有显式的进行任何同步操作，那他是如何保证线程安全呢？和饿汉模式一样，是靠JVM保证类的静态成员只能被加载一次的特点，这样就从JVM层面保证了只会有一个实例对象。那么问题来了，这种方式和饿汉模式又有什么区别呢？不也是立即加载么？实则不然，加载一个类时，其内部类不会同时被加载。一个类被加载，当且仅当其某个静态成员（静态域、构造器、静态方法等）被调用时发生。

可以说这种方式是实现单例模式的最优解。

**4. 静态代码块**

这里提供了静态代码块实现单例模式。这种方式和第一种类似，也是一种饿汉模式。

```java
//使用静态代码块实现单例模式
class SingletonStaticBlock {
    private static SingletonStaticBlock singletonStaticBlock;
    static {
        singletonStaticBlock = new SingletonStaticBlock();
    }
    public static SingletonStaticBlock getInstance() {
        return singletonStaticBlock;
    }
}
```

**5. 序列化与反序列化**

LZ为什么要提序列化和反序列化呢？因为单例模式虽然能保证线程安全，但在序列化和反序列化的情况下会出现生成多个对象的情况。运行下面的测试类，

```java
public class SingletonStaticInnerSerializeTest {

    public static void main(String[] args) {
        try {
            SingletonStaticInnerSerialize serialize = SingletonStaticInnerSerialize.getInstance();
            System.out.println(serialize.hashCode());
            //序列化
            FileOutputStream fo = new FileOutputStream("tem");
            ObjectOutputStream oo = new ObjectOutputStream(fo);
            oo.writeObject(serialize);
            oo.close();
            fo.close();
            //反序列化
            FileInputStream fi = new FileInputStream("tem");
            ObjectInputStream oi = new ObjectInputStream(fi);
            SingletonStaticInnerSerialize serialize2 = (SingletonStaticInnerSerialize) oi.readObject();
            oi.close();
            fi.close();
            System.out.println(serialize2.hashCode());
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

}

//使用匿名内部类实现单例模式，在遇见序列化和反序列化的场景，得到的不是同一个实例
//解决这个问题是在序列化的时候使用readResolve方法，即去掉注释的部分
class SingletonStaticInnerSerialize implements Serializable {

    /**
     * 2018年03月28日
     */
    private static final long serialVersionUID = 1L;

    private static class InnerClass {
        private static SingletonStaticInnerSerialize singletonStaticInnerSerialize = new SingletonStaticInnerSerialize();
    }

    public static SingletonStaticInnerSerialize getInstance() {
        return InnerClass.singletonStaticInnerSerialize;
    }

//    protected Object readResolve() {
//        System.out.println("调用了readResolve方法");
//        return InnerClass.singletonStaticInnerSerialize;
//    }
}
```

可以看到：

```
865113938
1078694789
```

结果表明的确是两个不同的对象实例，违背了单例模式，那么如何解决这个问题呢？解决办法就是在反序列化中使用readResolve\(\)方法，将上面的注释代码去掉，再次运行：

```
865113938
调用了readResolve方法
865113938
```

问题来了，readResolve\(\)方法到底是何方神圣，其实当JVM从内存中反序列化地"组装"一个新对象时，就会自动调用这个 readResolve方法来返回我们指定好的对象了, 单例规则也就得到了保证。readResolve\(\)的出现允许程序员自行控制通过反序列化得到的对象。

