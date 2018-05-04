# Java的IO和NIO

### Java的IO

Java的IO功能在`java.io`包下，包括输入、输出两种IO流，每种输入、输出流又可分为字节流和字符流两大类。字节流以字节（8位）为单位处理输入输出，字符流以字符（16位）为单位来处理输入输出。

**流的分类**

1. **输入输出流**

Java的输入流主要有`InputStream`和`Reader`作为基类，而输出流主要由`OutputStream`和`Writer`作为基类。他们是一些抽象类，无法直接创建实例。

2. **字节流和字符流**

**处理字节流：**InputStream、OutputStream；

**处理字符流：**Reader、Writer。

3. **节点流和处理流**

**节点流：**可以直接从IO设备（如磁盘、网络）读写数据的流称为节点流，也被称为低级流，比如，处理文件的输入流：FileInputStream和FileReader；

**处理流：**对一个已存在的流进行连接和封装，通过封装后的流来实现数据读写功能，也被称为高级流，比如PrintStream。

当使用节点流时，程序可以直接连接到实际的数据源。当使用处理流时，程序并不会直接连接到实际的数据源。

使用处理流的好处就是，只要使用相同的处理流，程序就可以采用完全相同的输入输出代码来访问不同的数据源，随着处理流所包装节点流的变化，程序实际访问的数据源也会相应的发生改变。实际上这是一种装饰者模式，处理流也被称为**包装流**。通过使用处理流，Java程序无须理会输入输出节点是磁盘还是网络，还是其他设备，程序只要将这些节点流包装成处理流，就可以使用相同的输入输出代码来读写不同设备上的数据。

**识别处理流很简单**，主要流的构造参数不是一个物理节点，而是已经存在的流；而节点流都是直接以物理IO节点作为构造器参数的。比如FileInputStream的构造器是`FileInputStream(String name)`和

FileInputStream(File file)，那么它就是一个节点流，再比如PrintStream的构造器是`PrintStream(OutputStream out) `，那么它就是一个处理流。

**使用处理流的优点**

- 对开发人员来讲，使用处理流进行输入输出操作更简单；
- 使用处理流的执行效率更高。

使用节点流的例子：

```java
package com.wangjun.othersOfJava;

import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
/*
 * 节点流演示
 */
public class FileInputStreamTest {

	public static void main(String[] args) throws IOException {
		//读取文件
		FileInputStream fis = new FileInputStream("README.md");
		//写入文件
		FileOutputStream fos = new FileOutputStream(new File("test.md"));
		//定义每次读取的字节数，保存在b中
		byte[] b = new byte[12];
		//read方法返回实际读取的字节数
		int hasRead = 0;
		while((hasRead = fis.read(b)) > 0) {
			System.out.println("hasRead:" + hasRead);
			System.out.println(new String(b, 0, hasRead));
			fos.write(b, 0, hasRead);
		}
		fis.close();
	}

}
```

使用处理流的例子：

```java
package com.wangjun.othersOfJava;

import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.PrintStream;

/*
 * 处理流演示
 * 程序使用了PrintStream包装了FileOutputStream节点流输出流，使用println输入字符和对象
 * PrintStream输出功能非常强大，我们平时常用的System.out.println()就是使用的PrintStream
 */
public class PrintStreamTest {

	public static void main(String[] args) throws FileNotFoundException {
		FileOutputStream fos = new FileOutputStream("print.md");
		PrintStream ps = new PrintStream(fos);
		ps.println("test1");
		ps.println("test2");
		ps.println(new PrintStreamTest());
	}

}
```

**Java的IO体系**

Java的IO体系提供了将近40个类。

下面将常用的一些类做一下分类汇总。其中粗体的为处理流。

| 分类   | 字节输入流                   | 字节输出流                    | 字符输入流                 | 字符输出流                  |
| ---- | ----------------------- | ------------------------ | --------------------- | ---------------------- |
| 基类   | InputStream             | OutputStream             | Reader                | Writer                 |
| 文件   | FileInputStream         | FileOutputStream         | FileReader            | FileWriter             |
| 数组   | ByteArrayInputStream    | ByteArrayOutputStream    | CharArrayReader       | CharArrayWriter        |
| 字符串  |                         |                          | StringReader          | StringWriter           |
| 缓冲流  | **BufferedInputStream** | **BufferedOutputStream** | **BufferedReader**    | **BufferedWriter**     |
| 转换流  |                         |                          | **InputStreamReader** | **OutputStreamWriter** |
| 对象流  | **ObjectInputStream**   | **ObjectOutputStream**   |                       |                        |
| 打印流  |                         | **PrintStream**          |                       | **PrintWriter**        |

 