

# Java序列化

###什么是序列化？

序列化是将一个对象的状态，各属性的值序列化保存起来，然后在合适的时候通过反序列化获得。

Java的序列化是将一个对象表示成字节序列，该字节序列包括了对象的数据，有关对象的类型信息和存储在对象中的数据类型。

说白了，就是将对象保存起来，就跟保存字符串数据一样，用到的时候再取出来。任何实现了Serializable接口的类都可以被序列化。

###实现Serializable接口进行序列化

```java
package com.wangjun.othersOfJava;

import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.io.Serializable;

public class SerializeDemo {

	public static void main(String[] args) {
		Employee em = new Employee();
		em.name = "wangjun";
		em.age = 24;
		em.ssh = 123456;
		// 将对象序列化后保存到文件
		try (
				FileOutputStream fo = new FileOutputStream("tem.ser");
				ObjectOutputStream oo = new ObjectOutputStream(fo))
		{
			oo.writeObject(em);
		} catch (IOException e) {
			e.printStackTrace();
		}
		// 反序列化取出对象
		try(
				FileInputStream fi = new FileInputStream("tem.ser");
				ObjectInputStream oi = new ObjectInputStream(fi)) 
		{
			Employee e2 = (Employee) oi.readObject();
			System.out.println(e2.name);
			System.out.println(e2.age);
			System.out.println(e2.ssh);
			System.out.println(Employee.local);
			e2.test();
		} catch (Exception e) {
			e.printStackTrace();
		}
	}

	static class Employee implements Serializable {
		String name;
		int age;
		static String local = "earth";
		transient int ssh;

		public void test() {
			System.out.println("this is test method!");
		}
	}

}
```

**程序的运行结果：**

```
wangjun
24
0
earth
this is test method!
```

如果有一些字段不想被序列化怎么办呢？这时候就可以用**transient**关键字修饰，就像上面代码的`ssh`字段，关于**transient**关键字有以下几个特点：

- 一旦被transient关键字修饰，那变量将不再是对象持久化的一部分，该变量内容在序列化后无法获得访问；
- transient只能修饰变量，不能修饰方法和类，本地变量（局部变量）也不能被transient修饰；
- 一个静态变量不管是否被transient修饰，都不能被序列化。

从上面的例子看到好像与第三条不符，其实反序列化取出的local是JVM里面的值，而不是反序列化出来的。可以加一行代码验证一下，在反序列化之前更改一下local的值：

```Java
// 反序列化取出对象
Employee.local = "earth2";
try(
  ...
```

看一下打印结果

```java
wangjun
24
0
earth2
this is test method!
```

这说明打印出来的是JVM中对应的local的值earth2，而不是序列化的时候的值earth。

### 实现Externalizable接口进行序列化

transient只有对实现了Serializable接口方式的序列化有效，还有一种序列化的方式是实现Externalizable接口，这种实现方式不像实现Serializable接口一样可以帮你自动序列化，它需要在writeExternal方法中手动指定需要序列化的变量并且在readExternal手动取出来，这与是否被transient修饰无关，下面更改一下上面的例子，将Employee类改成：

```java
static class Employee implements Externalizable {
		String name;
		int age;
		static String local = "earth";
		transient int ssh;
		
		//实现Externalizable接口进行序列化必须显式声明无参构造器
		public Employee() {
		}

		public void test() {
			System.out.println("this is test method!");
		}

		@Override
		public void writeExternal(ObjectOutput out) throws IOException {
			out.writeObject(name);
			//out.writeObject(age);
			out.writeObject(ssh);
			out.writeObject(local);
		}

		@Override
		public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException {
			name = (String) in.readObject();
			//age = (int) in.readObject();
			ssh = (int) in.readObject();
			local = (String) in.readObject();
		}
	}
```

重新运行，结果（注意：上述主函数中还存在对local重新赋值的代码`Employee.local = "earth2";`）：

```
wangjun
0
123456
earth
this is test method!
```

可以看到能否被序列化跟transient和static修饰都没有关系，只跟writeExternal和readExternal有关系。

### Serializable和Externalizable的区别

- 对Serializable对象反序列化时，由于Serializable对象完全以它存储的二进制位为基础来构造，因此并不会调用任何构造函数，因此Serializable类无需默认构造函数，但是当Serializable类的父类没有实现Serializable接口时，反序列化过程会调用父类的默认构造函数，因此该父类必需有默认构造函数，否则会抛异常。
- 对Externalizable对象反序列化时，会先调用类的不带参数的构造方法，这是有别于默认反序列方式的。如果把类的不带参数的构造方法删除，或者把该构造方法的访问权限设置为private、默认或protected级别，会抛出`java.io.InvalidException: no valid constructor`异常，因此Externalizable对象必须有默认构造函数，而且必需是public的。
- 如果不是特别坚持实现Externalizable接口，那么还有另一种方法。我们可以实现`Serializable`接口，并添加`writeObject()`和`readObject()`的方法。一旦对象被序列化或者重新装配，就会分别调用那两个方法。也就是说，只要提供了这两个方法，就会优先使用它们，而不考虑默认的序列化机制。

### SerialVersionUID的作用

上述实现Serializable接口的Employee类中，会有一个警告：

```
The serializable class Employee does not declare a static final serialVersionUID field of type long
```

意思是Employee没有声明一个静态final的常量serialVersionUID，那这个serialVersionUID的作用是什么呢？

serialVersionUID是对类进行版本控制的，Java的序列化机制是通过判断类的serialVersionUID来验证版本一致性的。在进行反序列化时，JVM会把传来的字节流中的serialVersionUID与本地相应实体类的serialVersionUID进行比较，如果相同就认为是一致的，可以进行反序列化，否则就会出现序列化版本不一致的异常，即是InvalidCastException。

serialVersionUID有两种生成方式：        

- 一是默认的1L，比如：private static final long serialVersionUID = 1L；
- 二是根据类名、接口名、成员方法及属性等来生成一个64位的哈希字段。

如果程序没有显式的声明serialVersionUID，那么程序将用第二种实现。我们可以做一个实现，还是用上述实现Serializable接口的例子。

我们先运行一下程序，生成序列化文件tem.ser，在把“将对象序列化后保存到文件”这一段逻辑注释掉，对Employee类增加一个test字段：

```java
static class Employee implements Serializable {
  String name;
  int age;
  static String local = "earth";
  transient int ssh;
  String test;

  public void test() {
    System.out.println("this is test method!");
  }
}
```

这时候运行的时候会报错：

```
java.io.InvalidClassException: com.wangjun.othersOfJava.SerializeDemo$Employee; local class incompatible: stream classdesc serialVersionUID = 4506166831890198488, local class serialVersionUID = 785960679919880606
	at java.io.ObjectStreamClass.initNonProxy(ObjectStreamClass.java:616)
	at java.io.ObjectInputStream.readNonProxyDesc(ObjectInputStream.java:1843)
	at java.io.ObjectInputStream.readClassDesc(ObjectInputStream.java:1713)
	at java.io.ObjectInputStream.readOrdinaryObject(ObjectInputStream.java:2000)
	at java.io.ObjectInputStream.readObject0(ObjectInputStream.java:1535)
	at java.io.ObjectInputStream.readObject(ObjectInputStream.java:422)
	at com.wangjun.othersOfJava.SerializeDemo.main(SerializeDemo.java:32)
```

因为程序发现取到的序列化文件的serialVersionUID和当前的serialVersionUID不一样。这个serialVersionUID是根据类名、接口名、成员方法及属性等来生成一个64位的哈希字段，因为增加了test字段，因此生成的serialVersionUID不一样了。

接着，我们显式的声明serialVersionUID

```java
static class Employee implements Serializable {
  private static final long serialVersionUID = 1L;
  String name;
  int age;
  static String local = "earth";
  transient int ssh;

  public void test() {
    System.out.println("this is test method!");
  }
}
```

将刚才注释的代码取消注释，运行一遍再注释掉，并且新增字段test：

```java
static class Employee implements Serializable {
  private static final long serialVersionUID = 1L;
  String name;
  int age;
  static String local = "earth";
  transient int ssh;
  String test;

  public void test() {
    System.out.println("this is test method!");
  }
}
```

再次运行发现没有报错，运行OK。这是因为你显式声明了serialVersionUID，序列化的serialVersionUID和目前的serialVersionUID一样，因此会认为是同一个版本的类。

你也可以将serialVersionUID改成2L，这个时候又会报错了。

> 参考：
>
> https://www.cnblogs.com/duanxz/p/3511695.html
>
> https://blog.csdn.net/fjndwy/article/details/39374231
>
> https://blog.csdn.net/bigtree_3721/article/details/50513295