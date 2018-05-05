# Java的IO和NIO

## Java的IO

Java的IO功能在`java.io`包下，包括输入、输出两种IO流，每种输入、输出流又可分为字节流和字符流两大类。字节流以字节（8位）为单位处理输入输出，字符流以字符（16位）为单位来处理输入输出。

### **1.1 流的分类**

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

### 1.2 Java的IO体系

Java的IO体系提供了将近40个类。

下面将常用的一些类做一下分类汇总。其中粗体的为处理流。

| 分类    | 字节输入流                   | 字节输出流                    | 字符输入流                 | 字符输出流                  |
| ----- | ----------------------- | ------------------------ | --------------------- | ---------------------- |
| 基类    | InputStream             | OutputStream             | Reader                | Writer                 |
| 文件    | FileInputStream         | FileOutputStream         | FileReader            | FileWriter             |
| 数组    | ByteArrayInputStream    | ByteArrayOutputStream    | CharArrayReader       | CharArrayWriter        |
| 字符串   |                         |                          | StringReader          | StringWriter           |
| 管道    | PipedInputStream        | PipedOutputStream        | PipedReader           | PipedWriter            |
| 缓冲流   | **BufferedInputStream** | **BufferedOutputStream** | **BufferedReader**    | **BufferedWriter**     |
| 转换流   |                         |                          | **InputStreamReader** | **OutputStreamWriter** |
| 对象流   | **ObjectInputStream**   | **ObjectOutputStream**   |                       |                        |
| 打印流   |                         | **PrintStream**          |                       | **PrintWriter**        |
| 推回输入流 | PushbackInputStream     |                          | PushbackReader        |                        |

 通常来说，我们认为字节流的功能比字符流的功能强大，因为计算机里所有的数据都是二进制的，而字节流可以处理所有的二进制文件，但问题就是如果使用字节流处理文本文件，则需要使用合适的方式将这些字节转换成字符，这就增加了编程的复杂度。

所以通常有这么一个规则：如果进行输入输出的内容是文本内容，则应考虑使用字符流；如果进行输入输出的内容是二进制内容，则应该考虑用字节流。

**注意1：**我们一般把计算机的文件分为文本文件和二进制文件，其实计算机所有的文件都是二进制文件，当二进制文件内容恰好能被正常解析成字符时，则该二进制文件就变成了文本文件。

**注意2：**上面的 表格中列举了4中访问管道的流，它们都是用于实现进程之间通信功能的。

### 1.3 转换流

上面的表格中列举了java提供的两个转换流。InputStreamReader和OutputStreamWriter用于将字节流转换成字符流。为什么要这么做呢？前面提到了字节流的功能比较强大，字符流使用起来方便，如果有一个字节流内容都是文本内容，那么将它转换成字符流更方便操作。

**代码实例**

```java
package com.wangjun.othersOfJava;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;

/*
 * 字节流转字符流示例
 * System.in 是键盘输入，是InputStream的实例，因为键盘输入都是字符，所以转换成字符流操作更方便
 * 普通的Reader读取内容时依然不方便，可以包装成BufferedReader，利用BufferedReader的readLine()方法
 * 可以一次读取一行内容
 */
public class InputStreamReaderTest {

	public static void main(String[] args) throws IOException {
		InputStreamReader isr = new InputStreamReader(System.in);
		BufferedReader br = new BufferedReader(isr);
		String buffer = null;
		while((buffer = br.readLine()) != null) {
			if(buffer.equals("exit")) {
				System.out.println("bye bye!");
				System.exit(1);
			}
			System.out.println("输入内容为：" + buffer);
		}
	}
}
```

BufferedReader具有一个readLine方法，可以非常方便的一次读取一行内容，所以经常把读取文本的输出流包装成BufferedReader，用来方便的读取输入流中的文本内容。

### 1.4 推回输入流

在java的IO体系中提供了两个特殊的流，PushbackInputStream和PushbackReader。它们提供了unread方法用于将字节/字符推回到推回缓冲区里面，从而允许读取刚刚读取的内容。

它们带有一个推回缓冲区，程序调用read方法时总是先从推回缓冲区里面读取，只有完全读取了推回缓冲区的内容，并且还没有装满read所需的数组时才会从原始输入流中读取。

**代码实例**

```java
package com.wangjun.othersOfJava;

import java.io.FileReader;
import java.io.IOException;
import java.io.PushbackReader;

/*
 * 推回缓冲流测试
 * 如果文件中包含LeetCode就把LeetCode周边的字符打印出来
 */
public class PushbackReaderTest {

	public static void main(String[] args) throws IOException {
		//推回缓冲区的长度为64
		PushbackReader pr = new PushbackReader(new FileReader("README.md"), 64);
		char[] cbuf = new char[16];
		int hasRead = 0;
		String lastRead = "";
		while((hasRead = pr.read(cbuf)) > 0) {
			String str = new String(cbuf, 0, hasRead);
			if((lastRead + str).contains("LeetCode")) {
				//推回到缓冲区
				pr.unread(cbuf, 0, hasRead);
				//这一次会先读取缓冲区的内容
				hasRead = pr.read(cbuf);
				//打印字符
				System.out.println(new String(cbuf, 0 ,hasRead));
				System.out.println("lastRead:" + lastRead);
				pr.close();
				return;
			}else {
				lastRead = str;
			}
		}
		pr.close();
	}

}
```

### 1.5 重定向标准输入输出

Java的标准输入输出分别通过System.in和System.out来代表，默认情况下他们代表键盘和显示器，在System类里面提供了3个重定向的方法：`setError(PrintStream err)`、`setIn(IputStream in)`、`setOut(PrintStream out)`。

**代码实例**

```java
package com.wangjun.othersOfJava;

import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.PrintStream;
import java.util.Scanner;

/*
 * 将System.out重定向到文件，而不是屏幕
 */
public class SystemInRedirect {

	public static void main(String[] args) throws FileNotFoundException {
		// 重定向标准输出
		PrintStream ps = new PrintStream(new FileOutputStream("out.md"));
		// 将标准输出重定向到ps流
		System.setOut(ps);
		// 后面打印的内容就会打印到ps中，而不是Console
		System.out.println("test1");
		System.out.println(new SystemInRedirect());

		// 重定向标准输入
		FileInputStream fis = new FileInputStream("test.md");
		System.setIn(fis);
		Scanner s = new Scanner(System.in);
		s.useDelimiter("\n");
		while (s.hasNext()) {
			System.out.println("获取到的输入内容是：" + s.next());
		}
		s.close();
	}

}
```

### 1.6 Java虚拟机读写其他进程的数据

Runtime对象的exec()方法可以运行平台上的其他程序，该方法产生一个Process对象，Process对象代表由该Java程序启动的子进程。Process提供了如下3个方法用于程序和其子进程进行通信：

- InputStream getErrorStream()：获取子进程的错误流；
- InputStream getInputStream()：获取子进程的输入流；
- OutputStream getOutputStream()：获取子进程的输出流。

**代码实例**

```java
package com.wangjun.othersOfJava;

import java.io.BufferedReader;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.PrintStream;
import java.util.Scanner;

/*
 * 读取其他进程的输出信息
 */
public class ReadFromProcess {

	public static void main(String[] args) throws IOException {
		// 1. 读取其他进程的数据
		// 运行javac命令，返回该命令的子进程
		Process p = Runtime.getRuntime().exec("javac");
		// 以p进程的错误流创建BufferedReader对象
		// 这个错误流对本程序是输入流，对p进程则是输出流
		BufferedReader br = new BufferedReader(new InputStreamReader(p.getErrorStream()));
		String buff = null;
		while ((buff = br.readLine()) != null) {
			System.out.println(buff);
		}

		// 2. 将数据输出到其他程序
		Process p2 = Runtime.getRuntime().exec("java ReadStandard");
		PrintStream ps = new PrintStream(p2.getOutputStream());
		ps.println("普通字符串");
		ps.println(new ReadFromProcess());
		System.out.println(222);
	}
}

class ReadStandard {
	public static void main(String[] args) throws FileNotFoundException {
		try (Scanner s = new Scanner(System.in); 
			PrintStream ps = new PrintStream(new FileOutputStream("out.md"))) {
			s.useDelimiter("\n");
			while (s.hasNext()) {
				ps.println("输入内容是：" + s.next());
			}
		}
	}
}
```

*输出到其他程序，代码运行没有成功，待分析...*

## Java的NIO

