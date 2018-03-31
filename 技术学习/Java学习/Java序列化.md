

# Java序列化

###什么是序列化？

序列化是将一个对象的状态，各属性的值序列化保存起来，然后在合适的时候通过反序列化获得。

Java的序列化是将一个对象表示成字节序列，该字节序列包括了对象的数据，有关对象的类型信息和存储在对象中的数据类型。

说白了，就是将对象保存起来，就跟保存字符串数据一样，用到的时候再取出来。

1. 任何实现了Serializable接口的类都可以被序列化。
2. 被transient修饰的属性不能被序列化。

###示例代码：

```java
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.io.Serializable;

public class SerializeDemo {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Employee em = new Employee();
		em.name = "wangjun";
		em.age = 24;
		em.ssh = 123456;
      	//将对象序列化后保存到文件
		try {
			FileOutputStream fo = new FileOutputStream("tem.ser");
			ObjectOutputStream oo = new ObjectOutputStream(fo);
			oo.writeObject(em);
			oo.close();
			fo.close();
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		//反序列化取出对象
		try {
			FileInputStream fi = new FileInputStream("tem.ser");
			ObjectInputStream oi = new ObjectInputStream(fi);
			Employee e2 = (Employee) oi.readObject();
			oi.close();
			fi.close();
			System.out.println(e2.name);
			System.out.println(e2.age);
			System.out.println(e2.ssh);
			e2.test();
		} catch (Exception e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
	
	static class Employee implements Serializable{
		String name;
		int age;
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
this is test method
```

可以看到被transient修饰的属性没有还原成最初的赋值。



