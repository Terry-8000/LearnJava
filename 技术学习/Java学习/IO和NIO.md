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

BufferedReader有一个特征，就是读取输入流中的数据时，如果没有读到有效数据，程序将在此处阻塞该线程的执行（使用InputStream的read方法从流中读取数据时，也有这样的特性），java.io下面的输入输出流都是阻塞式的。不仅如此，传统的输入输出流都是通过字节的移动来处理的，及时我们不直接去处理字节流，单底层的实现还是依赖于字节处理，也就是说，面向流的输入输出系统一次只能处理一个字节，因此面向流的输入输出系统通常效率都不高。

为了解决上面的问题，NIO就出现了。NIO采用内存映射文件来处理输入输出，NIO将文件或文件的一段区域映射到内存中，这样就可以像访问内存一样来访问文件了。（这种方式模拟了操作系统上虚拟内存的概念），通过这种方式进行输入输出要快得多。

Channel（通道）和Buffer（缓冲）是NIO中两个核心对象。Channel是对传统的输入输出系统的模拟，在NIO系统中所有的数据都需要通过通道传输，Channel与传统的InputStream、OutputStream最大的区别就是提供了一个map()方法，通过它可以直接将“一块数据”映射到内存中。如果说传统IO是面向流的处理，那么NIO是面向块的处理。

Buffer可以被理解为一个容器，它的本质是一个数组，发送到Channel中所有的对象都必须首先放到Buffer中，从而Channel中读取的数据也必须先放到Buffer中。Buffer既可以一次次从Channel取数据，也可以使用Channel直接将文件的某块数据映射成Buffer。

除了Channel和Buffer之外，NIO还提供了用于将Unicode字符串映射成字节序列以及逆映射操作的**Charset类**，也提供了用于支持非阻塞式输入输出的**Selector类**。其中Selector是非阻塞IO的核心。ß

### Buffer介绍

在Buffer中有三个重要的概念：

- 容量（capacity）：表示Buffer的大小，创建后不能呢过改变；
- 界限（limit）：第一个不能被读写的缓冲区的位置，也就是后面的数据不能被读写；
- 位置（position）：用于指明下一个可以被读写的缓冲区位置索引。当从Channel读取数据的时候，position的值就等于读到了多少数据。

Buffer的主要作用就是装入数据，然后输出数据。当装入数据结束时，调用flip()方法，该方法将limit设为position所在位置，将position的值设为0，为输出数据做好准备。当输出数据结束后，调用clear()方法，将position的值设为0，limit设为capacity，为下一次的装入数据做准备。

常用的Buffer是CharBuffer和ByteBuffer。

使用put()和get()方法进行数据的放入和读取，分为相对和绝对两种：

- 相对：从Buffer当前position位置开始读取或者写入数据，然后将position的值按处理元素的个数增加；
- 绝对：直接根据索引向Buffer中读取和写入数据，使用绝对方式访问Buffer里的数据时，不会影响position的值。

**代码示例**

```java
package com.wangjun.othersOfJava;

import java.nio.CharBuffer;

public class NIOBufferTest {

	public static void main(String[] args) {
		//创建CharBuffer
		CharBuffer cb = CharBuffer.allocate(8);
		System.out.println("capacity:" + cb.capacity());
		System.out.println("limit:" + cb.limit());
		System.out.println("position:" + cb.position());
		//放入元素
		cb.put('a');
		cb.put('b');
		cb.put('c');
		System.out.println("加入三个元素后，position:" + cb.position());
		cb.flip();
		System.out.println("执行flip()后，limit:" + cb.limit());
		System.out.println("执行flip()后，position:" + cb.position());
		System.out.println("取出第一个元素： " + cb.get());
		System.out.println("取出第一个元素后，position：" + cb.position());
		//调用clear方法
		cb.clear();
		System.out.println("执行clear()后，limit:" + cb.limit());
		System.out.println("执行clear()后，position:" + cb.position());
		System.out.println("执行clear()后，数据没有清空，第三个值是:" + cb.get(2));
		System.out.println("执行绝对读取后，position：" + cb.position());
	}

}
```

通过allocate方法创建的是普通Buffer，还可以通过allocateDirect方法来创建直接Buffer，虽然创建成本比较高，但是读写快。因此适用于长期生存的Buffer，使用方法和普通Buffer类似。注意，只有ByteBuffer提供了此方法，其他类型的想用，可以将该Buffer转成其他类型的Buffer。

### Channel（通道）介绍

Channel类似传统的流对象，主要区别如下：

- Channel可以直接将指定文件的部分或全部直接映射成Buffer。
- 程序不能直接访问Channel中的数据，只能通过Buffer交互。

所有的Channel都不应该通过构造器来创建，而是通过传统的InputStream、OutputStream的getChannel()方法来返回对应的Channel，不同的节点流获取的Channel不一样，比如FileInputStream返回的是FileChannel。

Channel常用的方法有三类：map()、read()、write()。map方法将Channel对应的部分或全部数据映射成ByteBuffer；read和write方法都有一系列的重载形式，这些方法用于从Buffer中读取/写入数据。

**代码示例**

```java
package com.wangjun.othersOfJava;

import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.nio.CharBuffer;
import java.nio.MappedByteBuffer;
import java.nio.channels.FileChannel;
import java.nio.charset.Charset;
import java.nio.charset.CharsetDecoder;

/*
 * 将FileChannel的全部数据映射成ByteBuffer
 */
public class NIOChannelTest {

	public static void main(String[] args) throws Exception {
		File f = new File("NIOChannelTest.md");
		//java7新特性try括号内的资源会在try语句结束后自动释放，前提是这些可关闭的资源必须实现 java.lang.AutoCloseable 接口。
		try(
			FileChannel inChannel = new FileInputStream(f).getChannel();
			FileChannel outChannel = new FileOutputStream("a.md").getChannel();
				){
			//将FileChannel的全部数据映射成ByteBuffer
			//map方法的三个参数：1映射模式；2,3控制将哪些数据映射成ByteBuffer
			MappedByteBuffer buffer = inChannel.map(FileChannel.MapMode.READ_ONLY, 0, f.length());
			//直接将buffer的数据全部输出，完成文件的复制： NIOChannelTest.md -> a.md
			outChannel.write(buffer);
			
			//以下为了输出文件字符串内容
			//使用GBK字符集来创建解码器
			Charset charset = Charset.forName("GBK");
			//复原limit和position的位置
			buffer.clear();
			//创建解码器
			CharsetDecoder decoder = charset.newDecoder();
			//使用解码器将ByteBuffer转成CharBuffer
			CharBuffer charBuffer = decoder.decode(buffer);
			System.out.println(charBuffer);
		}
	}

}
```



除了上面的一次性取数据，也可以分多次取数据

```java
//如果Channel对应的文件过大，使用map方法一次性将所有文件内容映射到内存中可能会因此性能下降
//这时候我们可以适应Channel和Buffer进行多次重复取值
public static void getDataByArray() throws Exception {
  try(
    //创建文件输入流
    FileInputStream fileInputStream = new FileInputStream("NIOChannelTest.md");
    FileChannel fileChannel = fileInputStream.getChannel();
  ){
    //定义一个ByteBuffer对象，用于重复取水
    ByteBuffer bbuff = ByteBuffer.allocate(64);
    while(fileChannel.read(bbuff) != -1) {
      bbuff.flip();
      Charset charset = Charset.forName("GBK");
      //创建解码器
      CharsetDecoder decoder = charset.newDecoder();
      //使用解码器将ByteBuffer转成CharBuffer
      CharBuffer charBuffer = decoder.decode(bbuff);
      System.out.println(charBuffer);
      //为下一次读取数据做准备
      bbuff.clear();
    }
  }
}
```

### Selector（选择器）介绍

Selector选择器类管理着一个被注册的通道集合的信息和它们的就绪状态。通道是和选择器一起被注册的，并且使用选择器来更新通道的就绪状态。利用 Selector可使一个单独的线程管理多个 Channel，selector 是非阻塞 IO 的核心。

**应用**

当通道使用register（Selector sel, int ops）方法将通道注册选择器时，选择器对通道事件进行监听，通过第二个参数指定监听的事件类型。

其中可监听的事件类型包括以下：

　　读 : SelectionKey.OP_READ

　　写 : SelectionKey.OP_WRITE

　　连接 : SelectionKey.OP_CONNECT

　　接收 : SelectionKey.OP_ACCEPT

如果需要监听多个事件可以使用`位或`操作符：

　　int key = SelectionKey.OP_READ | SelectionKey.OP_WRITE ; //表示同时监听读写操作

> 参考：
>
> https://blog.csdn.net/dd864140130/article/details/50299687
>
> https://www.cnblogs.com/qq-361807535/p/6670529.html
>
> https://blog.csdn.net/dd864140130/article/details/50299687

### 文件锁

在NIO中，Java提供了文件锁的支持，使用FileLock来支持文件锁定功能，在FileChannel中提供lock()/tryLock()方法来获取文件锁FileLock对象，从而锁定文件。lock和tryLock的区别是前者无法得到文件锁的时候会阻塞，后者不会阻塞。也支持锁定部分内容，使用lock(long position, long size, boolean shared)即可，其中shared为true时，表明该锁是一个共享锁，可以允许多个县城读取文件，但阻止其他进程获得该文件的排他锁。当shared为false时，表明是一个排他锁，它将锁住对该文件的读写。

默认获取的是排他锁。

**代码示例**

```java
package com.wangjun.othersOfJava;

import java.io.FileOutputStream;
import java.nio.channels.FileChannel;
import java.nio.channels.FileLock;

public class FileLockTest {
	public static void main(String[] args) throws Exception {
		try(
			FileOutputStream fos = new FileOutputStream("a.md");
			FileChannel fc = fos.getChannel();
				){
			//使用非阻塞方式对指定文件加锁
			FileLock lock = fc.tryLock();
			Thread.sleep(3000);
			lock.release();//释放锁
		}
	}
}

```



## Java的NIO2

Java7对原来的NIO进行了重大改进：

- 提供了全面的文件IO和文件系统访问支持；
- 基于异步Channel的IO。

这里先简单介绍一下对文件系统的支持，后续继续学习。

NIO2提供了一下接口和工具类：

- Path接口：通过和Paths工具类结合使用产生Path对象，可以获取文件根路径、绝对路径、路径数量等；
- Files工具类：提供了很多静态方法，比如复制文件、一次性读取文件所有行、判断是否为隐藏文件、判断文件大小、遍历文件和子目录、访问文件属性等；
- FileVisitor接口：代表一个文件访问器，Files工具类使用walkFileTree方法遍历文件和子目录时，都会触发FileVisitor中相应的方法，比如访问目录之前、之后，访问文件时，访问文件失败时；
- WatchService：监控文件的变化；

## NIO和IO的区别

Java的NIO提供了与标准IO不同的工作方式：

- Channels and Buffers（通道和缓冲区）：标准的IO基于字节流和字符流进行操作的，没有被缓存在任何地方，而NIO是基于通道（Channel）和缓冲区（Buffer）进行操作，数据总是从通道读取到缓冲区中，或者从缓冲区写入到通道中，因此可以前后移动流中的数据；
- Asynchronous IO（异步IO）：Java NIO可以让你异步的使用IO，例如：当线程从通道读取数据到缓冲区时，线程还是可以进行其他事情。当数据被写入到缓冲区时，线程可以继续处理它。从缓冲区写入通道也类似。因为NIO将阻塞交给了后台线程执行，非阻塞IO的空余时间可以用于其他通道上进行IO，因此可以一个线程可以管理多个通道，而IO是阻塞的，阻塞式的IO每个连接必须开一个线程来处理，每个线程都需要珍贵的CPU资源，等待IO的时候，CPU资源没有被释放就被浪费掉了。
- Selectors（选择器）：选择器允许一个单独的线程可以监听多个数据通道（（网络连接或文件），你可以注册多个通道使用一个选择器，然后使用一个单独的线程来“选择”通道：这些通道里已经有可以处理的输入，或者选择已准备写入的通道。这种选择机制，使得一个单独的线程很容易来管理多个通道。但付出的代价是解析数据可能会比从一个阻塞流中读取数据更复杂。 

### 使用场景

**NIO**

- 优势在于一个线程管理多个通道；但是数据的处理将会变得复杂；
- 如果需要管理同时打开的成千上万个连接，这些连接每次只是发送少量的数据，采用这种；

**传统的IO**

- 适用于一个线程管理一个通道的情况；因为其中的流数据的读取是阻塞的；
- 如果需要管理同时打开不太多的连接，这些连接会发送大量的数据；


> 参考：BIO、NIO、AIO
>
> https://blog.csdn.net/anxpp/article/details/51512200 
>
> http://bbym010.iteye.com/blog/2100868

## Netty简介

Netty是一个java开源框架。Netty提供异步的、事件驱动的网络应用程序框架和工具，用以快速开发高性能、高可靠性的网络服务器和客户端程序。

Netty是一个NIO客户端、服务端框架。允许快速简单的开发网络应用程序。例如：服务端和客户端之间的协议。它最牛逼的地方在于简化了网络编程规范。例如:TCP和UDP的Socket服务。



