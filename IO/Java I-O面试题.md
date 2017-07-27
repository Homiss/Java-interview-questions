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


### 结束


文章转载自：
http://blog.csdn.net/u012815721/article/details/25279613





