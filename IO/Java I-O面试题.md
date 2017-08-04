# Java I/O面试题

标签（空格分隔）： I/O Java面试题

---

### 流的概念和作用
流是一种有顺序的，有起点和终点的字节集合，是对数据传输的总成或抽象。即数据在两设备之间的传输称之为流，流的本质是数据传输，根据数据传输的特性讲流抽象为各种类，方便更直观的进行数据操作。

### IO流的分类

根据数据处理类的不同分为：字符流和字节流。

根据数据流向不同分为：输入流和输出流。

### 字符流和字节流

字符流的由来：因为数据编码的不同，而有了对字符进行高效操作的流对象，其本质就是基于字节流读取时，去查了指定的码表。字符流和字节流的区别：

（1）读写单位不同：字节流一字节（8bit）为单位，字符流以字符为单位，根据码表映射字符，一次可能读多个字节。

（2）处理对象不同：字节流能处理所有类型的数据（例如图片，avi），而字符流只能处理字符类型的数据。

（3）字节流操作的时候本身是不会用到缓冲区的，是对文件本身的直接操作。而字符流在操作的时候是会用到缓冲区的，通过缓冲区来操作文件。

结论：优先使用字节流，首先因为在硬盘上所有的文件都是以字节的形式进行传输或保存的，包括图片等内容。但是字符流只是在内存中才会形成，所以在开发中字节流使用广泛。

##输入流和输出流

对输入流只能进行读操作，对输出流只能进行写操作。程序中根据数据传输的不同特性使用不同的流。

##输入字节流InputStream

InputStream是所有输入字节流的父类，它是一个抽象类。

ByteArrayInputStream、StringBufferInputStream、FileInputStream 是三种基本的介质流，它们分别从Byte 数组、StringBuffer、和本地文件中读取数据。PipedInputStream 是从与其它线程共用的管道中读取数据。

ObjectInputStream 和所有FilterInputStream的子类都是装饰流（装饰器模式的主角）。意思是FileInputStream类可以通过一个String路径名创建一个对象，FileInputStream(String name)。而DataInputStream必须装饰一个类才能返回一个对象，DataInputStream(InputStream in)。

ByteArrayInputStream:
```java
/**  
 * 使用内存操作流将一个大写字母转化为小写字母  
 * */  
import java.io.*;  
class hello{  
   public static void main(String[] args) throws IOException {  
       String str="Homiss";  
       ByteArrayInputStream input=new ByteArrayInputStream(str.getBytes());  
       ByteArrayOutputStream output=new ByteArrayOutputStream();  
       int temp=0;  
       while((temp=input.read())!=-1){  
           char ch=(char)temp;  
           output.write(Character.toLowerCase(ch));  
       }  
       String outStr=output.toString();  
       input.close();  
       output.close();  
       System.out.println(outStr);  
    }  
}  
```

字节流FileInputStream读取文件：
```java
import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStream;

/**
 * @author Homiss
 * 逐字节读取文件内容
 */
public class IOFileInputStream {
	
	public static void main(String... args) throws IOException{
		String path = "F:"+File.separator+"test.txt";
		File file = new File(path);
		InputStream in = new FileInputStream(file);
		
		byte[] b = new byte[1024];
		int temp = 0;
		int count = 0;
		while((temp = in.read()) != -1){
			b[count++] = (byte)temp;
		}
		System.out.println(new String(b));
	}
}

```
FileOutputStream添加内容：
```java
import java.io.File;
import java.io.FileOutputStream;
import java.io.IOException;

public class IOFileOutputStream {
	
	public static void main(String... args) throws IOException{
		String path = "f:"+File.separator+"test.txt";
		//true表示在文件末尾追加内容
		FileOutputStream fout = new FileOutputStream(path,true);
		String str = "\r\n半兽人";
		
		fout.write(str.getBytes());
		fout.close();
	}
}

```

PushBackInputStream回退流操作：
```java
import java.io.ByteArrayInputStream;
import java.io.IOException;
import java.io.PushbackInputStream;

/**
 * @author Homiss
 * PushbackInputStream
 */
public class IOPushbackInputStream {
	
	public static void main(String... args) throws IOException{
		String str = "hello , this is my Blog!";
		ByteArrayInputStream in = new ByteArrayInputStream(str.getBytes());
		PushbackInputStream pis = new PushbackInputStream(in);
		
		int temp = 0;
		while((temp = pis.read()) != -1){
			if(temp == ','){
				pis.unread(temp);
				temp = pis.read();
				System.out.println((char)temp);
			}else{
				System.out.print((char)temp);
			}
		}
	}
}
```
PipedInputStream管道流操作：
```java
import java.io.IOException;
import java.io.PipedInputStream;
import java.io.PipedOutputStream;

public class IOPipedOutputStream {
	public static void main(String... args) throws IOException{
		
		Send send=new Send();  
		Recive recive=new Recive();  
		try{  
           send.getOut().connect(recive.getInput());  
		}catch (Exception e) {  
           e.printStackTrace();  
		}  
		new Thread(recive).start();
		new Thread(send).start();  
	}
}

class Send implements Runnable{
	
	private PipedOutputStream out = null;
	public Send(){
		out = new PipedOutputStream();
	}
	
	public PipedOutputStream getOut(){
		return this.out;
	}
	
	@Override
	public void run() {
		// TODO Auto-generated method stub
		String message="hello , Homiss";  
		try{  
           out.write(message.getBytes());  
		}catch (Exception e) {  
           e.printStackTrace();  
		}try{  
            out.close();
		}catch (Exception e) {  
           e.printStackTrace();  
       }
	}
}

class Recive implements Runnable{

	private PipedInputStream input = null;
	
	public Recive() {
		input = new PipedInputStream();
	}

	public PipedInputStream getInput() {
		return input;
	}

	@Override
	public void run() {
		// TODO Auto-generated method stub
		byte[] b = new byte[1024];
		int len = 0;
		try{
			len = this.input.read(b);
		}catch(Exception e){
			e.printStackTrace();
		}
		System.out.println("接受到的内容为:"+(new String(b,0,len)));
	}
}
```
SequenceInputStream:将两个文本文件合并为另外一个文本文件
```java
import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.io.SequenceInputStream;

public class SequenceInputStreamDemo {
	public static void main(String... args) throws IOException{
		File f1 = new File("f:"+File.separator+"test.txt");
		File f2 = new File("f:"+File.separator+"homiss.txt");
		File f3 = new File("f:"+File.separator+"my.txt");
		
		InputStream in1 = new FileInputStream(f1);
		InputStream in2 = new FileInputStream(f2);
		OutputStream out = new FileOutputStream(f3);
		
		//合并流
		SequenceInputStream sis = new SequenceInputStream(in1,in2);
		int temp = 0;
		while((temp = sis.read()) != -1){
			out.write(temp);
		}
		in1.close();
		in2.close();
		out.close();
		sis.close();
	}
}
```
PrintStream:也可以认为是一个辅助工具。主要可以向其他输出流，或者FileInputStream 写入数据，本身内部实现还是带缓冲的。
```java
import java.io.File;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.PrintStream;

public class PrintStreamDemo {
	public static void main(String... args) throws IOException{
		File f = new File("f:"+File.separator+"test.txt");
		PrintStream print = new PrintStream(new FileOutputStream(f));
		
		String name = "Homiss";
		int age = 21;
		print.printf("姓名：%s . 年龄：%d .",name,age);
		print.close();
	}
}

```

ZipOutputStream:
```java
import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.util.zip.ZipEntry;
import java.util.zip.ZipOutputStream;
public class ZipOutputStreamDemo01 {
	public static void main(String... args)throws IOException{
		File file = new File("f:"+File.separator+"test.txt");
		File zipFile = new File("f:"+File.separator+"test.zip");
		
		InputStream in = new FileInputStream(file);
		ZipOutputStream zipOut = new ZipOutputStream(new FileOutputStream(zipFile));
		
		zipOut.putNextEntry(new ZipEntry(file.getName()));
		//设置注释
		zipOut.setComment("Homiss");
		int temp = 0;
		while((temp = in.read()) != -1){
			zipOut.write(temp);
		}
		in.close();
		zipOut.close();
	}
}
```

### 字符输入流Reader
定义和说明：
>Reader 是所有的输入字符流的父类，它是一个抽象类。

CharReader、StringReader是两种基本的介质流，它们分别将Char 数组、String中读取数据。PipedReader 是从与其它线程共用的管道中读取数据。
BufferedReader 很明显就是一个装饰器，它和其子类负责装饰其它Reader 对象。
FilterReader 是所有自定义具体装饰流的父类，其子类PushbackReader 对Reader 对象进行装饰，会增加一个行号。

InputStreamReader 是一个连接字节流和字符流的桥梁，它将字节流转变为字符流。FileReader可以说是一个达到此功能、常用的工具类，在其源代码中明显使用了将FileInputStream 转变为Reader 的方法。我们可以从这个类中得到一定的技巧。Reader 中各个类的用途和使用方法基本和InputStream 中的类使用一致。

BufferedReader:
>注意：BufferedReader只能接受字符流的缓冲区，因为每一个中文需要占据两个字节，所以需要将System.in这个字节输入流变为字符输入流，采用：
BufferedReader buf = new BufferedReader(newInputStreamReader(System.in));

```java
import java.io.BufferedReader;  
import java.io.IOException;  
import java.io.InputStreamReader;  
    
/**  
 * 使用缓冲区从键盘上读入内容  
 * */  
public class BufferedReaderDemo{  
   public static void main(String[] args){  
       BufferedReader buf = new BufferedReader(  
                newInputStreamReader(System.in));  
       String str = null;  
       System.out.println("请输入内容");  
       try{  
           str = buf.readLine();  
       }catch(IOException e){  
           e.printStackTrace();  
       }  
       System.out.println("你输入的内容是：" + str);  
    }  
}  
```

Scanner类从文件中读出内容:
```java
import java.io.File;  
import java.io.FileNotFoundException;  
import java.util.Scanner;  
    
/**  
 *Scanner的小例子，从文件中读内容  
 * */  
public class ScannerDemo{  
   public static void main(String[] args){  
    
       File file = new File("d:" + File.separator +"hello.txt");  
       Scanner sca = null;  
       try{  
           sca = new Scanner(file);  
       }catch(FileNotFoundException e){  
           e.printStackTrace();  
       }  
       String str = sca.next();  
       System.out.println("从文件中读取的内容是：" + str);  
    }  
}  
```

### Java I/O库的两个对称性
Java I/O库有两个对称性，他们分别是：

 1. 输入-输出对称：比如InputStream和OutputStream各自占据byte流的输入和输出的两个平行的等级结构的根部；而Reader和Writer各自占据Char流的输入和输出的两个平行的等级结构的根部。
 2. byte-char对称：InputStream和Reader的子类分别负责byte和插入流的输入；OutputStream和Writer的子类分别负责byte和插入流的输出，它们分别形成平行的等级结构。

### Java I/O库的两个设计模式

Java I/O库的总体设计是符合装饰模式和适配器模式的。如前所述，这个库中处理流的类叫流类。

装饰模式（Decorator）：在由InputStream、OutputStream、Reader和Writer代表的等级结构内部，有一些流处理器可以对另一些流处理器起到装饰作用，形成新的、具有改善了的功能的流处理器。

适配器模式（Adapter）：在由InputStream、OutputStream、Reader和Writer代表的等级结构内部，有一些流处理器是对其他类型的流处理器的适配。这就是适配器的应用。

####装饰模式的应用

InputStream类型中的装饰模式：

　InputStream有七个直接的具体子类，有四个属于FilterInputStream的具体子类，如下图所示：

上图中所有的类都叫做流处理器，这个图就叫做（InputStream类型的）流处理器图。

书中提到根据输入流的源的类型，可以将这些流类分成两种，即原始流类（Original Stream）和链接流处理器（Wrapper Stream）。

####原始流处理器

原始流处理器接收一个Byte数组对象，String对象，FileDiscriptor对象或者不同类型的流源对象，根据上面的图，原始流处理器包括以下四种：

>- ByteArrayInputStream：为多线程的通信提供缓冲区操作功能，接收一个Byte数组作为流的源。
- FileInputStream:建立一个与文件有关的输入流。接收一个File对象作为流的源。
- PipedInputStream：可以与PipedOutputStream配合使用，用于读入一个数据管道的数据，接收一个PipedOutputStream作为源。
- StringBufferInputStream：将一个字符串缓冲区转换为一个输入流。接收一个String对象作为流的源。（ＪＤＫ帮助文档上说明：已过时。此类未能正确地将字符转换为字节。从ＪＤＫ1.1开始，从字符串创建流的首选方法是通过StringReader类进行创建。只有字符串中每个字符的低八位可以由此类使用。）

####链接流处理器

所谓链接流处理器，就是可以接收另一个流对象作为源，并对之进行功能扩展的类。InputStream类型的链接处理器包括以下几种，它们都接收另一个InputStream对象作为流源。

(1)FilterInputStream称为过滤输入流，它将另一个输入流作为流源。这个类的子类包括以下几种：
> - BufferedInputStream：用来从硬盘将数据读入到一个内存缓冲区中，并从缓冲区提供数据。
- DataInputStream：提供基于多字节的读取方法，可以读取原始类型的数据。
- LineNumberInputStream：提供带有行计数功能的过滤输入流。
- PushbackInputStream：提供特殊的功能，可以将已经读取的字节“推回”到输入流中。

(2)ObjectInputStream可以将使用ObjectInputStream串行化的原始数据类型和对象重新并行化。

(3)SeqcueneInputStream可以将两个已有的输入流连接起来，形成一个输入流，从而将多个输入流排列构成一个输入流序列。

### NIO和BIO的区别

- 阻塞IO：使用简单，但随之而来的问题就是会形成阻塞，需要独立线程配合，而这些线程在大多数时候都是没有进行运算的。Java的BIO使用这种方式，问题带来的问题很明显，一个Socket需要一个独立的线程，因此，会造成线程膨胀。
- 非阻塞IO：采用轮询方式，不会形成线程的阻塞。Java的NIO使用这种方式，对比BIO的优势很明显，可以使用一个线程进行所有Socket的监听（select）。大大减少了线程数。

### Linux IO模型

linux下有五种常见的IO模型，其中只有一种异步模型，其余皆为同步模型。如图：
![image_1bk88gscilce1i75jisk9r7n92q.png-26kB][1]

### 阻塞IO模型

阻塞IO模型是最常见的IO模型了，对于所有的“慢速设备”（socket、pipe、fifo、terminal）的IO默认的方式都是阻塞的方式。阻塞就是进程放弃cpu，让给其他进程使用cpu。进程阻塞最显著的表现就是进程睡眠了。阻塞的时间通常取决于数据是否到来。
这种方式使用简单，但随之而来的问题就是会形成阻塞，需要独立线程配合，而这些线程在大多数时候都是没有进行运算的。Java的BIO使用这种方式，问题带来的问题很明显，一个Socket需要一个独立的线程，因此，会造成线程膨胀。

![image_1bk88hgjnqr51p359avht71h3n37.png-20kB][2]

### 非阻塞IO模型

非阻塞IO就是设置IO相关的系统调用为non-blocking，随后进行的IO操作无论有没有可用数据都会立即返回，并设置errno为EWOULDBLOCK或者EAGAIN。我们可以通过主动check的方式（polling，轮询）确保IO有效时，随之进行相关的IO操作。当然这种方式看起来就似乎不太靠谱，浪费了太多的CPU时间，用宝贵的CPU时间做轮询太不靠谱儿了。图示：
![image_1bk88i1t59b11d4dsqb112j1suf3k.png-27.6kB][3]

### 多路复用

为了解决阻塞I/O的问题,就有了I/O多路复用模型,多路复用就是用单独的线程(是内核级的, 可以认为是高效的优化的) 来统一等待所有的socket上的数据, 一当某个socket上有数据后, 就启用用户线程(可能是从线程池中取出, 而不是重新生成), copy socket data, 并且处理message.因为网络延迟的原因, 同时在处理socket data的用户线程往往比实际的socket数量要少很多. 所以实际应用中, 大部分是用线程池, 池中thread数量可随socket的高峰和低谷 而动态调整.

多路复用I/O中内核中统一﻿的wait socket data那部分可以理解成是非阻塞, 也可以理解成阻塞. 可以理解成非阻塞 是因为它不是等到socket数据全部到达再处理, 而是有了一部分数据就会调用用户线程来处理, 理解成阻塞, 是因为它和用户空间(Appliction)层的非阻塞﻿socket的不同是: socket中没有数据时, 内核还是wait(阻塞)的, 而用户空间的非阻塞﻿socket没有数据也会返回, 会造成CPU的浪费.

Linux下的select和poll 就是多路复用模式,poll相对select,没有了句柄数的限制,但他们都是在内核层通过轮询socket句柄的方式来实现的, 没有利用更底层的notify机制. 但就算是这样,相对阻塞socket也已经﻿进步了很多很多了! 毕竟用一个内核线程就解决了,阻塞socket中N多线程都在无谓地wait的局面.

> 多路复用I/O﻿ 还是让用户层来copy socket data. 这个过程是将内核中的socket buffer copy到用户空间的 buffer. 这有两个问题: 一是多了一次内核空间switch到用户空间的过程, 二是用户空间层不便暴露很低层但很高效的copy方式(比如DMA), 所以如果由内核层来做这个动作, 可以更好地提高效率!

![image_1bk88j5gruddk8pv918ptd9q41.png-32.7kB][4]

### 信号驱动IO模型

所谓信号驱动，就是利用信号机制，安装信号SIGIO的处理函数（进行IO相关操作），通过监控文件描述符，当其就绪时，通知目标进程进行IO操作（signal handler）。
![image_1bk88jplt7hjr51et81a06k524e.png-23.2kB][5]

### 异步IO模型

由于异步IO请求只是写入了缓存，从缓存到硬盘是否成功不可知，因此异步IO相当于把一个IO拆成了两部分，一是发起请求，二是获取处理结果。因此，对应用来说增加了复杂性。但是异步IO的性能是所有很好的，而且异步的思想贯穿了IT系统放放面面。
![image_1bk88kd1vlm7d0f142hvcd1rln4r.png-19.6kB][6]

### epoll

epoll是Java NIO在linux上的默认实现。相关的工具可以关注select,poll，关于三者间的区别，参见这里。在Mac上类似的实现是kqueue,Solaris上是/dev/poll。
epoll的优点

- 支持一个进程打开大数目的socket描述符(FD)
- IO效率不随FD数目增加而线性下降

> 传统的select/poll另一个致命弱点就是当你拥有一个很大的socket集合，不过由于网络延时，任一时间只有部分的socket是”活跃”的，但是select/poll每次调用都会线性扫描全部的集合，导致效率呈现线性下降。但是epoll不存在这个问题，它只会对”活跃”的socket进行操作—这是因为在内核实现中epoll是根据每个fd上面的callback函数实现的。那么，只有”活跃”的socket才会主动的去调用 callback函数，其他idle状态socket则不会，在这点上，epoll实现了一个”伪”AIO，因为这时候推动力在os内核。

- 使用mmap加速内核与用户空间的消息传递。

- 内核微调

epoll有2种工作方式:LT和ET:

- LT(level triggered)是缺省的工作方式，并且同时支持block和no-block socket.在这种做法中，内核告诉你一个文件描述符是否就绪了，然后你可以对这个就绪的fd进行IO操作。如果你不作任何操作，内核还是会继续通知你的，所以，这种模式编程出错误可能性要小一点。传统的select/poll都是这种模型的代表．
- ET (edge-triggered)是高速工作方式，只支持no-block socket。在这种模式下，当描述符从未就绪变为就绪时，内核通过epoll告诉你。然后它会假设你知道文件描述符已经就绪，并且不会再为那个文件描述符发送更多的就绪通知，直到你做了某些操作导致那个文件描述符不再为就绪状态了(比如，你在发送，接收或者接收请求，或者发送接收的数据少于一定量时导致了一个EWOULDBLOCK 错误）。但是请注意，如果一直不对这个fd作IO操作(从而导致它再次变成未就绪)，内核不会发送更多的通知(only once),不过在TCP协议中，ET模式的加速效用仍需要更多的benchmark确认。

### Zero Copy

上面多次提到内核空间和用户空间的switch, 在socket read/write这么小的粒度频繁调用, 代价肯定是很大的.
所以可以在网上看到Zero Copy的技术, 说到底Zero Copy的思路就是: 分析你的业务, 看看是否能避免不必要的跨空间copy,比如可以用 sendfile()函数充分利用内核可以调用DMA的优势, 直接在内核空间将文件的内容通过socket发送出去,而不必经过用户空间.显然,sendfile是有很多的前提条件的, 如果你想让文件内容作一些变换再发出去,就必须要经过用户空间的Appliation logic, 也是无法使用sendfile了.还有一种方式就是象epoll所做的,用内存映射. 据我所知，kafka速度快的一个原因就是使用了零拷贝。
关于零拷贝，可以看这篇文章

### C10K 问题
网络服务在处理数以万计的客户端连接时,往往出现效率低下甚至完全瘫痪,这被 称为C10K问题。随着互联网的迅速发展,越来越多的网络服务开始面临 C10K 问题, 作为大型网站的开发人员有必要对C10K问题有一定的了解。
C10K问题的最大特点是:设计不够良好的程序,其性能和连接数及机器性能的关系往往是非线性的。举个例子:如果没有考虑过C10K问题,一个经典的基于select的程序能在旧服务器上很好处理1000并发的吞吐量,它在2倍性能新服务器上往往处理不了并发2000的吞吐量。
这是因为在策略不当时,大量操作的消耗和当前连接数n成线性相关。会导致单个任务的资源消耗和当前连接数的关系会是O(n)。而服务程序需要同时对数以万计的socket进行I/O处理,积累下来的资源消耗会相当可观,这显然会导致系统吞吐量不能 和机器性能匹配。为解决这个问题,必须改变对连接提供服务的策略。
更详细的资料参考：The C10K problem

### java中有几种类型的流？JDK为每种类型的流提供了一些抽象类以供继承，请说出他们分别是哪些类？
字节输入流：InputStream,字节输出流：OutputStream
字符输入流：Reader，字符输出流：Writer

### 什么是java序列化，如何实现java序列化？
Java对象的序列化指将一个java对象写入OI流中，与此对应的是，对象的反序列化则从IO流中恢复该java对象。
如果要让某个对象支持序列化机制，则必须让它的类是可序列化的，为了让某个类是可序列化的，该类必须实现Serializable接口或Externalizable接口


### 解释一下java.io.Serializable接口（面试常考）
类通过实现 Java.io.Serializable 接口以启用其序列化功能。未实现此接口的类将无法使其任何状态序列化或反序列化。
### 读写原始数据，一般采用什么流？（AC ）
A InputStream
B DataInputStream
C OutputStream
D BufferedInputStream
### 为了提高读写性能，可以采用什么流？（DF）
A InputStream
B DataInputStream
C BufferedReader
D BufferedInputStream
E OutputStream
F BufferedOutputStream
### 对各种基本数据类型和String类型的读写，采用什么流？（ AD）
A DataInputStream
B BufferedReader
C PrintWriter
D DataOutputStream
E ObjectInputStream
F ObjectOutputStream
### 能指定字符编码的I/O流类型是：（BH ）
A Reader
B InputStreamReader
C BufferedReader
D Writer
E PrintWriter
F ObjectInputStream
G ObjectOutputStream
H OutputStreamWriter
### File类型中定义了什么方法来判断一个文件是否存在？（ D）
A createNewFile
B renameTo
C delete
D exists
### File类型中定义了什么方法来创建一级目录？（ C）
A createNewFile
B exists
C mkdirs
D mkdir
### 对文本文件操作用什么I/O流？（AD ）
A FileReader
B FileInputStream
C RandomAccessFile
D FileWriter
### 在unix服务器www.openlab.com.cn上提供了基于TCP的时间服务应用，该应用使用port为13。创建连接到此服务器的语句是：（A ）
A Socket s = new Socket(“www.openlab.com.cn”, 13);
B Socket s = new Socket(“www.openlab.com.cn:13”);
C Socket s = accept(“www.openlab.com.cn”, 13);
### 创建一个TCP客户程序的顺序是：（DACBE ）
A 获得I/O流
B 关闭I/O流
C 对I/O流进行读写操作
D 建立socket
E 关闭socket
### 创建一个TCP服务程序的顺序是：（BCADEGF ）
A 创建一个服务线程处理新的连接
B 创建一个服务器socket
C 从服务器socket接受客户连接请求
D 在服务线程中，从socket中获得I/O流
E 对I/O流进行读写操作，完成与客户的交互
F 关闭socket
G 关闭I/O流
### Java UDP编程主要用到的两个类型是：（ BD）
A UDPSocket
B DatagramSocket
C UDPPacket
D DatagramPacket
### TCP/IP是一种：（ B）
A 标准 
B 协议  
C 语言  
D 算法



### 结束


文章转载自：
http://blog.csdn.net/u012815721/article/details/25279613


  [1]: http://static.zybuluo.com/homiss/tq95mnettxwgivyukwwxpph2/image_1bk88gscilce1i75jisk9r7n92q.png
  [2]: http://static.zybuluo.com/homiss/l40bwthl5c00zt1mpi632b5v/image_1bk88hgjnqr51p359avht71h3n37.png
  [3]: http://static.zybuluo.com/homiss/7suuki09g1ymwsv3e1vlgbxa/image_1bk88i1t59b11d4dsqb112j1suf3k.png
  [4]: http://static.zybuluo.com/homiss/yt9wrdd7grujbp1z8qmpw80y/image_1bk88j5gruddk8pv918ptd9q41.png
  [5]: http://static.zybuluo.com/homiss/ou5borc0llzhcfggdk8uek6u/image_1bk88jplt7hjr51et81a06k524e.png
  [6]: http://static.zybuluo.com/homiss/qz90av9jfzjfrqnu0hi6d6xe/image_1bk88kd1vlm7d0f142hvcd1rln4r.png
  
