# Java异常处理面试题

标签（空格分隔）： Java面试题 异常处理

---

### 异常处理完成以后，Exception对象会发生什么变化？
Exception对象会在下一个垃圾回收过程中被回收掉。

### 下面关于java.lang.Exception类的说法正确的是（）
A. 继承自 Throwable
B. Serialable
CD 不记得，反正不正确

答案：A

解析：Java 异常的基类为 java.lang.Throwable，java.lang.Error 和 java.lang.Exception 继承 Throwable，RuntimeException 和其它的 Exception 等继承 Exception，具体的 RuntimeException 继承 RuntimeException。

### 扩展：错误和异常的区别(Error vs Exception)
1) java.lang.Error: Throwable 的子类，用于标记严重错误。合理的应用程序不应该去 try/catch 这种错误。绝大多数的错误都是非正常的，就根本不该出现的。

java.lang.Exception: Throwable 的子类，用于指示一种合理的程序想去 catch 的条件。即它仅仅是一种程序运行条件，而非严重错误，并且鼓励用户程序去 catch 它。

2) Error 和 RuntimeException 及其子类都是未检查的异常（unchecked exceptions），而所有其他的 Exception 类都是检查了的异常（checked exceptions）

checked exceptions: 通常是从一个可以恢复的程序中抛出来的，并且最好能够从这种异常中使用程序恢复。比如 FileNotFoundException, ParseException 等。检查了的异常发生在编译阶段，必须要使用 try…catch（或者 throws ）否则编译不通过。
unchecked exceptions: 通常是如果一切正常的话本不该发生的异常，但是的确发生了。发生在运行期，具有不确定性，主要是由于程序的逻辑问题所引起的。比如 ArrayIndexOutOfBoundException, ClassCastException 等。从语言本身的角度讲，程序不该去 catch 这类异常，虽然能够从诸如 RuntimeException 这样的异常中 catch 并恢复，但是并不鼓励终端程序员这么做，因为完全没要必要。因为这类错误本身就是 bug，应该被修复，出现此类错误时程序就应该立即停止执行。 因此，面对 Errors 和 unchecked exceptions 应该让程序自动终止执行，程序员不该做诸如 try/catch 这样的事情，而是应该查明原因，修改代码逻辑。

RuntimeException：RuntimeException体系包括错误的类型转换、数组越界访问和试图访问空指针等等。处理 RuntimeException 的原则是：如果出现 RuntimeException，那么一定是程序员的错误。例如，可以通过检查数组下标和数组边界来避免数组越界访问异常。其他（IOException等等）checked 异常一般是外部错误，例如试图从文件尾后读取数据等，这并不是程序本身的错误，而是在应用环境中出现的外部错误。

### getCustomerInfo() 方法如下，try 中可以捕获三种类型的异常，如果在该方法运行中产生了一个 IOException，将会输出什么结果?
```java
public void getCustomerInfo() {
    try {
        // do something that may cause an Exception
    } catch (java.io.FileNotFoundException ex) {

        System.out.print("FileNotFoundException!");

    } catch (java.io.IOException ex) {

        System.out.print("IOException!");

    } catch (java.lang.Exception ex) {

        System.out.print("Exception!");

    }

}
```
A. IOException!
B. IOException!Exception!
C. FileNotFoundException!IOException!
D. FileNotFoundException!IOException!Exception!

答案：A

解析：考察多个 catch 语句块的执行顺序。当用多个 catch 语句时，catch 语句块在次序上有先后之分。从最前面的 catch 语句块依次先后进行异常类型匹配，这样如果父异常在子异常类之前，那么首先匹配的将是父异常类，子异常类将不会获得匹配的机会，也即子异常类型所在的 catch 语句块将是不可到达的语句。所以，一般将父类异常类即 Exception 老大放在 catch 语句块的最后一个。

### try{} 里有一个 return 语句，那么紧跟在这个 try 后的 finally{} 里的 code 会不会被执行，什么时候被执行，在 return 前还是后?
答：会执行，在方法返回调用者前执行。Java允许在finally中改变返回值的做法是不好的，因为如果存在finally代码块，try 中的return语句不会立马返回调用者，而是记录下返回值待 finally 代码块执行完毕之后再向调用者返回其值，然后如果在 finally 中修改了返回值，这会对程序造成很大的困扰，C# 中就从语法上规定不能做这样的事。

### Java 语言如何进行异常处理，关键字：throws、throw、try、catch、finally 分别如何使用？
答：Java 通过面向对象的方法进行异常处理，把各种不同的异常进行分类，并提供了良好的接口。在 Java 中，每个异常都是一个对象，它是 Throwable 类或其子类的实例。当一个方法出现异常后便抛出一个异常对象，该对象中包含有异常信息，调用这个对象的方法可以捕获到这个异常并进行处理。

Java 的异常处理是通过5个关键词来实现的：try、catch、throw、throws 和 finally。

一般情况下是用try来执行一段程序，如果出现异常，系统会抛出（throw）一个异常，这时候你可以通过它的类型来捕捉（catch）它，或最后（finally）由缺省处理器来处理；try用来指定一块预防所有“异常”的程序；catch子句紧跟在try块后面，用来指定你想要捕捉的“异常”的类型；throw语句用来明确地抛出一个“异常”；throws用来标明一个成员函数可能抛出的各种“异常”；finally为确保一段代码不管发生什么“异常”都被执行一段代码；可以在一个成员函数调用的外面写一个try语句，在这个成员函数内部写另一个try语句保护其他代码。每当遇到一个try语句，“异常”的框架就放到栈上面，直到所有的try语句都完成。如果下一级的try语句没有对某种“异常”进行处理，栈就会展开，直到遇到有处理这种“异常”的 try 语句。

### 运行时异常与受检异常有何异同？
答：异常表示程序运行过程中可能出现的非正常状态，运行时异常表示虚拟机的通常操作中可能遇到的异常，是一种常见运行错误，只要程序设计得没有问题通常就不会发生。受检异常跟程序运行的上下文环境有关，即使程序设计无误，仍然可能因使用的问题而引发。Java编译器要求方法必须声明抛出可能发生的受检异常，但是并不要求必须声明抛出未被捕获的运行时异常。异常和继承一样，是面向对象程序设计中经常被滥用的东西，神作《Effective Java》中对异常的使用给出了以下指导原则：

不要将异常处理用于正常的控制流（设计良好的API不应该强迫它的调用者为了正常的控制流而使用异常）
对可以恢复的情况使用受检异常，对编程错误使用运行时异常
避免不必要的使用受检异常（可以通过一些状态检测手段来避免异常的发生）
优先使用标准的异常
每个方法抛出的异常都要有文档
保持异常的原子性
不要在 catch 中忽略掉捕获到的异常
### 请写出 5 种常见到的runtime exception。
答：

NullPointerException：当操作一个空引用时会出现此错误。

NumberFormatException：数据格式转换出现问题时出现此异常。

ClassCastException：强制类型转换类型不匹配时出现此异常。

ArrayIndexOutOfBoundsException：数组下标越界，当使用一个不存在的数组下标时出现此异常。

ArithmeticException：数学运行错误时出现此异常

### error 和 exception 有什么区别?
答：

error 表示系统级的错误和程序不必处理的异常，是恢复不是不可能但很困难的情况下的一种严重问题；比如内存溢出，不可能指望程序能处理这样的情况； exception 表示需要捕捉或者需要程序进行处理的异常，是一种设计或实现问题；也就是说，它表示如果程序运行正常，从不会发生的情况。

### 结束


参考：
[异常处理][1]


  [1]: http://wiki.jikexueyuan.com/project/java-interview-bible/exception.html