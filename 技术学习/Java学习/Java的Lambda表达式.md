# Java的Lambda表达式

## 1. 什么是Lambda表达式

简单的说，Lambda表达式就是匿名方法。Lambda表达式让程序员能够使用更加简洁的代码，但是同样也使代码的可读性比较差。

Lambda表达式也叫做匿名方法或者闭包。

## 2. 和匿名内部类做对比

Lambda是匿名方法，这个时候我们会想想到匿名内部类，我们来回想一下匿名内部类的用法，比如下面的代码就是使用匿名内部类实现了一个线程。

```java
public class Test {
  public static void main(String[] args) {
    Thread t = new Thread(new Runnable() {
      @Override
      public void run() {
        System.out.println("线程：" + Thread.currentThread().getName());
      }
    });
    t.start();
  }
}
```

我们一般的做法是写一个Runnable接口的实现类，然后new一个实现类再传给Thread的构造器。如下：

```java
public class Test {

  public static void main(String[] args) {
    MyThread myThread = new MyThread();
    Thread t = new Thread(myThread);
    t.start();
  }

  static class MyThread implements Runnable {
    @Override
    public void run() {
      System.out.println("线程：" + Thread.currentThread().getName());
    }
  }
  
}
```

可以看到使用匿名内部类的话就省略了新建Runnable接口的实现类这一步骤。

## 3. 使用Lambda表达式

上面使用匿名内部类的写法，如果使用Lambda表达式可以写成下面这样：

```java
public class Test {
  public static void main(String[] args) {
    Thread t = new Thread(() ->  {
      System.out.println("线程：" + Thread.currentThread().getName());
    });
    t.start();
  }
}
```

这样有一个问题，如果接口里面有多个方法，那么Lambda表达式怎么知道实现的是哪个方法呢？我们通过代码测试一下：

```java
package com.wangjun.othersOfJava;

public class LambdaTest {

	public static void main(String[] args) {
		Animal a = () -> {  // 编译报错：The target type of this expression must be a functional interface
			System.out.println("狗狗吃饭");
		};
		a.eat();
	}
	interface Animal {
		public void eat();
		public void duty();
	}
}
```

可以看到编译报错，这个提到一个functional interface，就是函数式接口。**函数式接口就是只有一个抽象方法的接口**。这样，就不难理解了，原来Lambda表达式只支持函数式接口。

### 4. Lambda表达式使用的几种方式

```java
package com.wangjun.othersOfJava;

public class LambdaTest {
	
	public static void main(String[] args) {
		
		// 带类型
		Animal a1 = (String str) -> {
			System.out.println("狗狗吃饭:" + str);
		};
		// 不带类型
		Animal a2 = (str) -> {
			System.out.println("狗狗吃饭:" + str);
		};
		// 不带括号
		Animal a3 = str -> {
			System.out.println("狗狗吃饭:" + str);
		};
		// 不带大括号
		Animal a4 = str -> System.out.println("狗狗吃饭:" + str);
		a1.eat("火腿肠");
		a2.eat("牛肉");
		a3.eat("面条");
		a4.eat("米饭");
		
		// 使用return返回
		Person p1 = () -> {
			return "老师的职责：教书育人!";
		};
		// 直接返回
		Person p2 = () -> "医生的职责：救死扶伤!";
		System.out.println(p1.duty());
		System.out.println(p2.duty());
	}
	
	// 没有返回值
	interface Animal {
		public void eat(String str);
	}
	// 有返回值
	interface Person {
		public String duty();
	}
}
```

## 4. Java的双冒号表达式

JDK8中有双冒号的用法，就是把方法当做参数传到stream内部，使stream的每个元素都传入到该方法里面执行一下。下面通过遍历一个List来说明一下双冒号和Lambda表达式使用方式的不同。

```java
package com.wangjun.othersOfJava;

import java.util.Arrays;
import java.util.List;
import java.util.function.Consumer;

public class LambdaTest {
	
  public static void printStr(String str) {
    System.out.println(str);
  }

  public static void main(String[] args) {

    List<String> list = Arrays.asList("aaa","bbb","ccc");
    // 1.通常的遍历方式
    for(String str: list) {
      LambdaTest.printStr(str);
    }
    // 2.使用Lambda表达式遍历
    list.forEach(str -> {
      LambdaTest.printStr(str);
    });
    // 3.使用::遍历
    list.forEach(LambdaTest::printStr);
    // 下面的方法和上面等价，使用的是函数式编程
    Consumer<String> methodParam = LambdaTest::printStr; //方法参数
    list.forEach(x -> methodParam.accept(x));//方法执行accept
  }
}
```



