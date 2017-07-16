# Java面试题之多线程(二)

标签（空格分隔）： Java面试题 多线程

---

### 不使用stop停止线程？
当run() 或者 call() 方法执行完的时候线程会自动结束,如果要手动结束一个线程，你可以用volatile 布尔变量来退出run()方法的循环或者是取消任务来中断线程。

使用自定义的标志位决定线程的执行情况
```java
public class SafeStopThread implements Runnable{  
   private volatile boolean stop=false;//此变量必须加上volatile  
   int a=0;  
   @Override  
    public void run() {  
        // TODO Auto-generated method stub  
        while(!stop){  
               synchronized ("") {  
                    a++;  
                    try {  
                        Thread.sleep(100);  
                    } catch (Exception e) {  
                        // TODO: handle exception  
                    }  
                    a--;  
                    String tn=Thread.currentThread().getName();  
                    System.out.println(tn+":a="+a);  
                }  
        }  
      //线程终止  
     public void terminate(){  
         stop=true;  
      }  
  public static void main(String[] args) {  
       SafeStopThread t=new SafeStopThread();  
       Thread t1=new Thread(t);  
       t1.start();  
       for(int i=0;i<5;i++){   
           new Thread(t).start();  
       }  
     t.terminate();  
   }  
} 
```
### Java中如何实现线程？各有什么优缺点,比较常用的是那种,为什么？
在语言层面有两种方式。java.lang.Thread 类的实例就是一个线程但是它需要调用java.lang.Runnable接口来执行，由于线程类本身就是调用的Runnable接口所以你可以继承java.lang.Thread 类或者直接调用Runnable接口来重写run()方法实现线程。

Java不支持类的多重继承，但允许你调用多个接口。所以如果你要继承其他类，当然是调用Runnable接口好了。
### 如何控制某个方法允许并发访问线程的大小？
Semaphore两个重要的方法就是semaphore.acquire() 请求一个信号量，这时候的信号量个数-1（一旦没有可使用的信号量，也即信号量个数变为负数时，再次请求的时候就会阻塞，直到其他线程释放了信号量）semaphore.release()释放一个信号量，此时信号量个数+1
```java
public class SemaphoreTest {  
    private Semaphore mSemaphore = new Semaphore(5);  
    public void run(){  
        for(int i=0; i< 100; i++){  
            new Thread(new Runnable() {  
                @Override  
                public void run() {  
                    test();  
                }  
            }).start();  
        }  
    }  
  
    private void test(){  
        try {  
            mSemaphore.acquire();  
        } catch (InterruptedException e) {  
            e.printStackTrace();  
        }  
        System.out.println(Thread.currentThread().getName() + " 进来了");  
        try {  
            Thread.sleep(1000);  
        } catch (InterruptedException e) {  
            e.printStackTrace();  
        }  
        System.out.println(Thread.currentThread().getName() + " 出去了");  
        mSemaphore.release();  
    }  
}  
```
### 在Java中什么是线程调度？ 
JVM调度的模式有两种：分时调度和抢占式调度。

分时调度是所有线程轮流获得CPU使用权，并平均分配每个线程占用CPU的时间;

抢占式调度是根据线程的优先级别来获取CPU的使用权。

JVM的线程调度模式采用了抢占式模式。既然是抢占调度，那么我们就能通过设置优先级来“有限”的控制线程的运行顺序，注意“有限”一次。
### Java中用到的线程调度算法是什么？
抢占式。一个线程用完CPU之后，操作系统会根据线程优先级、线程饥饿情况等数据算出一个总的优先级并分配下一个时间片给某个线程执行。

### 线程类的构造方法、静态块是被哪个线程调用的？
线程类的构造方法、静态块是被new这个线程类所在的线程所调用的，而run方法里面的代码才是被线程自身所调用的。

### 用Runnable还是Thread？
Java不支持类的多重继承，但允许你调用多个接口。所以如果你要继承其他类，当然是调用Runnable接口好了。

### 在实现Runnable的接口中怎么样访问当前线程对象,比如拿到当前线程的名字？
```java
Thread t = Thread.currentThread();
String name = t.getName();
System.out.println("name=" + name);
```

### 什么是线程池？为什么要使用它？为什么使用Executor框架比使用应用创建和管理线程好？ 
创建线程要花费昂贵的资源和时间，如果任务来了才创建线程那么响应时间会变长，而且一个进程能创建的线程数有限。

为了避免这些问题，在程序启动的时候就创建若干线程来响应处理，它们被称为线程池，里面的线程叫工作线程。

Executor框架让你可以创建不同的线程池。比如单线程池，每次处理一个任务；数目固定的线程池或者是缓存线程池（一个适合很多生存期短的任务的程序的可扩展线程池）。

### 常用的线程池模式以及不同线程池的使用场景？
以下是Java自带的几种线程池： 
1、newFixedThreadPool 创建一个指定工作线程数量的线程池。
每当提交一个任务就创建一个工作线程，如果工作线程数量达到线程池初始的最大数，则将提交的任务存入到池队列中。 

2、newCachedThreadPool 创建一个可缓存的线程池。
这种类型的线程池特点是： 

- 1).工作线程的创建数量几乎没有限制(其实也有限制的,数目为Interger. MAX_VALUE),这样可灵活的往线程池中添加线程。 

- 2).如果长时间没有往线程池中提交任务，即如果工作线程空闲了指定的时间(默认为1分钟)，则该工作线程将自动终止。终止后，如果你又提交了新的任务，则线程池重新创建一个工作线程。 

3、newSingleThreadExecutor创建一个单线程化的Executor，即只创建唯一的工作者线程来执行任务，如果这个线程异常结束，会有另一个取代它，保证顺序执行(我觉得这点是它的特色)。

单工作线程最大的特点是可保证顺序地执行各个任务，并且在任意给定的时间不会有多个线程是活动的。 

4、newScheduleThreadPool 创建一个定长的线程池，而且支持定时的以及周期性的任务执行，类似于Timer。

总结： 
一.FixedThreadPool是一个典型且优秀的线程池，它具有线程池提高程序效率和节省创建线程时所耗的开销的优点。但在线程池空闲时，即线程池中没有可运行任务时，它不会释放工作线程，还会占用一定的系统资源。 

二．CachedThreadPool的特点就是在线程池空闲时，即线程池中没有可运行任务时，它会释放工作线程，从而释放工作线程所占用的资源。但是，但当出现新任务时，又要创建一新的工作线程，又要一定的系统开销。并且，在使用CachedThreadPool时，一定要注意控制任务的数量，否则，由于大量线程同时运行，很有会造成系统瘫痪。 

### 在Java中Executor、ExecutorService、Executors的区别？
Executor 和 ExecutorService 这两个接口主要的区别是：

- ExecutorService 接口继承了 Executor 接口，是 Executor 的子接口 
- Executor 和 ExecutorService 第二个区别是：Executor 接口定义了 execute()方法用来接收一个Runnable接口的对象，而 ExecutorService 接口中的 submit()方法可以接受Runnable和Callable接口的对象。
- Executor 和 ExecutorService 接口第三个区别是 Executor 中的 execute() 方法不返回任何结果，而 ExecutorService 中的 submit()方法可以通过一个 Future 对象返回运算结果。
- Executor 和 ExecutorService 接口第四个区别是除了允许客户端提交一个任务，ExecutorService 还提供用来控制线程池的方法。比如：调用 shutDown() 方法终止线程池。

Executors 类提供工厂方法用来创建不同类型的线程池。

比如: newSingleThreadExecutor() 创建一个只有一个线程的线程池，newFixedThreadPool(int numOfThreads)来创建固定线程数的线程池，newCachedThreadPool()可以根据需要创建新的线程，但如果已有线程是空闲的会重用已有线程。

### 如何创建一个Java线程池？
Java通过Executors提供四种线程池，分别为：

newCachedThreadPool创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。

newFixedThreadPool 创建一个定长线程池，可控制线程最大并发数，超出的线程会在队列中等待。

newScheduledThreadPool 创建一个定长线程池，支持定时及周期性任务执行。

newSingleThreadExecutor 创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行。

### Thread 类中的start() 和 run() 方法有什么区别？
start()方法被用来启动新创建的线程，而且start()内部调用了run()方法，这和直接调用run()方法的效果不一样。

当你调用run()方法的时候，只会是在原来的线程中调用，没有新的线程启动，start()方法才会启动新线程。

### Java线程池中submit() 和 execute()方法有什么区别？
两个方法都可以向线程池提交任务，execute()方法的返回类型是void，它定义在Executor接口中, 而submit()方法可以返回持有计算结果的Future对象，它定义在ExecutorService接口中，它扩展了Executor接口，其它线程池类像ThreadPoolExecutor和ScheduledThreadPoolExecutor都有这些方法。

### Java中notify 和 notifyAll有什么区别？
notify()方法不能唤醒某个具体的线程，所以只有一个线程在等待的时候它才有用武之地。而notifyAll()唤醒所有线程并允许他们争夺锁确保了至少有一个线程能继续运行。

当有线程调用了对象的 notifyAll()方法（唤醒所有 wait 线程）或 notify()方法（只随机唤醒一个 wait 线程），被唤醒的的线程便会进入该对象的锁池中，锁池中的线程会去竞争该对象锁。也就是说，调用了notify后只要一个线程会由等待池进入锁池，而notifyAll会将该对象等待池内的所有线程移动到锁池中，等待锁竞争

优先级高的线程竞争到对象锁的概率大，假若某线程没有竞争到该对象锁，它还会留在锁池中，唯有线程再次调用 wait()方法，它才会重新回到等待池中。


### 为什么wait, notify 和 notifyAll这些方法不在thread类里面？
一个很明显的原因是JAVA提供的锁是对象级的而不是线程级的，每个对象都有锁，通过线程获得。

如果线程需要等待某些锁那么调用对象中的wait()方法就有意义了。如果wait()方法定义在Thread类中，线程正在等待的是哪个锁就不明显了。

简单的说，由于wait，notify和notifyAll都是锁级别的操作，所以把他们定义在Object类中因为锁属于对象。

### 为什么wait和notify方法要在同步块中调用？
主要是因为Java API强制要求这样做，如果你不这么做，你的代码会抛出IllegalMonitorStateException异常。还有一个原因是为了避免wait和notify之间产生竞态条件。

### 讲下join,yield方法的作用,以及什么场合用它们？
join() 的作用：让“主线程”等待“子线程”结束之后才能继续运行。

yield方法可以暂停当前正在执行的线程对象，让其它有相同优先级的线程执行。它是一个静态方法而且只保证当前线程放弃CPU占用而不能保证使其它线程一定能占用CPU，执行yield()的线程有可能在进入到暂停状态后马上又被执行。

### sleep方法有什么作用,一般用来做什么？
sleep()方法（休眠）是线程类（Thread）的静态方法，调用此方法会让当前线程暂停执行指定的时间，**将执行机会（CPU）让给其他线程**，但是对象的锁依然保持，因此休眠时间结束后会自动恢复。注意这里的恢复并不是恢复到执行的状态，而是恢复到可运行状态中等待CPU的宠幸。

### Java多线程中调用wait() 和 sleep()方法有什么不同？
Java程序中wait和sleep都会造成某种形式的暂停，它们可以满足不同的需要。

- wait存在于Object类中；sleep存在于Thread类中。
- wait会让出CPU资源以及释放锁；sleep只会释放CPU资源。
- wait只能在同步块中使用；sleep没这限制。
- wait需要notify（或 notifyAll）唤醒，进入等锁状态；sleep到指定时间便会自动恢复到运行状态。

### 为什么Thread里面的大部分方法都是final的？
不能被重写，线程的很多方法都是由系统调用的，不能通过子类覆写去改变他们的行为。

### 为什么Thread类的sleep()和yield()方法是静态的？
Thread类的sleep()和yield()方法将在当前正在执行的线程上运行。

该代码只有在某个A线程执行时会被执行，这种情况下通知某个B线程yield是无意义的（因为B线程本来就没在执行）。因此只有当前线程执行yield才是有意义的。通过使该方法为static，你将不会浪费时间尝试yield 其他线程。

> 只能给自己喂安眠药，不能给别人喂安眠药。

### 什么是阻塞式方法？
阻塞式方法是指程序会一直等待该方法完成期间不做其他事情。

ServerSocket的accept()方法就是一直等待客户端连接。这里的阻塞是指调用结果返回之前，当前线程会被挂起，直到得到结果之后才会返回。

此外，还有异步和非阻塞式方法在任务完成前就返回。

### 如何强制启动一个线程？
在Java里面没有办法强制启动一个线程，它是被线程调度器控制着

### 一个线程运行时发生异常会怎样？
简单的说，如果异常没有被捕获该线程将会停止执行。

Thread.UncaughtExceptionHandler是用于处理未捕获异常造成线程突然中断情况的一个内嵌接口。

当一个未捕获异常将造成线程中断的时候JVM会使用Thread.getUncaughtExceptionHandler()来查询线程的UncaughtExceptionHandler并将线程和异常作为参数传递给handler的uncaughtException()方法进行处理。

### 在线程中你怎么处理不可控制异常？ 
在Java中有两种异常。

非运行时异常（Checked Exception）：这种异常必须在方法声明的throws语句指定，或者在方法体内捕获。例如：IOException和ClassNotFoundException。

运行时异常（Unchecked Exception）：这种异常不必在方法声明中指定，也不需要在方法体中捕获。例如，NumberFormatException。

因为run()方法不支持throws语句，所以当线程对象的run()方法抛出非运行异常时，我们必须捕获并且处理它们。当运行时异常从run()方法中抛出时，默认行为是在控制台输出堆栈记录并且退出程序。

好在，java提供给我们一种在线程对象里捕获和处理运行时异常的一种机制。实现用来处理运行时异常的类，这个类实现UncaughtExceptionHandler接口并且实现这个接口的uncaughtException()方法。示例：
```java
package concurrency;

import java.lang.Thread.UncaughtExceptionHandler;

public class Main2 {
    public static void main(String[] args) {
        Task task = new Task();
        Thread thread = new Thread(task);
        thread.setUncaughtExceptionHandler(new ExceptionHandler());
        thread.start();
    }
}

class Task implements Runnable{
    @Override
    public void run() {
        int numero = Integer.parseInt("TTT");
    }
}

class ExceptionHandler implements UncaughtExceptionHandler{
    @Override
    public void uncaughtException(Thread t, Throwable e) {
        System.out.printf("An exception has been captured\n");
        System.out.printf("Thread:  %s\n", t.getId());
        System.out.printf("Exception:  %s:  %s\n", e.getClass().getName(),e.getMessage());
        System.out.printf("Stack Trace:  \n");
        e.printStackTrace(System.out);
        System.out.printf("Thread status:  %s\n",t.getState());
    }
}
```
当一个线程抛出了异常并且没有被捕获时（这种情况只可能是运行时异常），JVM检查这个线程是否被预置了未捕获异常处理器。如果找到，JVM将调用线程对象的这个方法，并将线程对象和异常作为传入参数。

Thread类还有另一个方法可以处理未捕获到的异常，即静态方法setDefaultUncaughtExceptionHandler()。这个方法在应用程序中为所有的线程对象创建了一个异常处理器。

当线程抛出一个未捕获到的异常时，JVM将为异常寻找以下三种可能的处理器。首先，它查找线程对象的未捕获异常处理器。如果找不到，JVM继续查找线程对象所在的线程组（ThreadGroup）的未捕获异常处理器。如果还是找不到，如同本节所讲的，JVM将继续查找默认的未捕获异常处理器。如果没有一个处理器存在，JVM则将堆栈异常记录打印到控制台，并退出程序。

### 如果同步块内的线程抛出异常会发生什么？
无论你的同步块是正常还是异常退出的，里面的线程都会释放锁，所以对比锁接口我更喜欢同步块，因为它不用我花费精力去释放锁，该功能可以在finally block里释放锁实现。

### 为什么你应该在循环中检查等待条件?
处于等待状态的线程可能会收到错误警报和伪唤醒，如果不在循环中检查等待条件，程序就会在没有满足结束条件的情况下退出。

1、一般来说，wait肯定是在某个条件调用的，不是if就是while
2、放在while里面，是防止出于waiting的对象被别的原因调用了唤醒方法，但是while里面的条件并没有满足（也可能当时满足了，但是由于别的线程操作后，又不满足了），就需要再次调用wait将其挂起。
3、其实还有一点，就是while最好也被同步，这样不会导致错失信号。

### 多线程中的忙循环是什么?
忙循环就是程序员用循环让一个线程等待，不像传统方法wait()、 sleep() 或 yield()，它们都放弃了CPU控制，而忙循环不会放弃CPU，它就是在运行一个空循环。

这么做的目的是为了保留CPU缓存，在多核系统中，一个等待线程醒来的时候可能会在另一个内核运行，这样会重建缓存。为了避免重建缓存和减少等待重建的时间就可以使用它了。

### 什么是自旋锁？
没有获得锁的线程一直循环在那里看是否该锁的保持者已经释放了锁，这就是自旋锁。

### 什么是互斥锁？
互斥锁：从等待到解锁过程，线程会从sleep状态变为running状态，过程中有线程上下文的切换，抢占CPU等开销。

### 自旋锁的优缺点？
自旋锁不会引起调用者休眠，如果自旋锁已经被别的线程保持，调用者就一直循环在那里看是否该自旋锁的保持者释放了锁。由于自旋锁不会引起调用者休眠，所以自旋锁的效率远高于互斥锁。

虽然自旋锁效率比互斥锁高，但它会存在下面两个问题：
1、自旋锁一直占用CPU，在未获得锁的情况下，一直运行，如果不能在很短的时间内获得锁，会导致CPU效率降低。
2、试图递归地获得自旋锁会引起死锁。递归程序决不能在持有自旋锁时调用它自己，也决不能在递归调用时试图获得相同的自旋锁。

由此可见，我们要慎重的使用自旋锁，自旋锁适合于锁使用者保持锁时间比较短并且锁竞争不激烈的情况。正是由于自旋锁使用者一般保持锁时间非常短，因此选择自旋而不是睡眠是非常必要的，自旋锁的效率远高于互斥锁。
       
### 如何在两个线程间共享数据？
同一个Runnable，使用全局变量。

第一种：将共享数据封装到一个对象中，把这个共享数据所在的对象传递给不同的Runnable

第二种：将这些Runnable对象作为某一个类的内部类，共享的数据作为外部类的成员变量，对共享数据的操作分配给外部类的方法来完成，以此实现对操作共享数据的互斥和通信，作为内部类的Runnable来操作外部类的方法，实现对数据的操作
```java
class ShareData {
 private int x = 0;

 public synchronized void addx(){
   x++;
   System.out.println("x++ : "+x);
 }
 public synchronized void subx(){
   x--;
   System.out.println("x-- : "+x);
 }
}

public class ThreadsVisitData {
 
 public static ShareData share = new ShareData();
 
 public static void main(String[] args) {
  //final ShareData share = new ShareData();
  new Thread(new Runnable() {
    public void run() {
        for(int i = 0;i<100;i++){
            share.addx();
        }
    }
  }).start();
  new Thread(new Runnable() {
    public void run() {
        for(int i = 0;i<100;i++){
            share.subx();
        }
    }
   }).start(); 
 }
}
```

### Java中Runnable和Callable有什么不同？
Runnable和Callable都是接口, 不同之处：
1.Callable可以返回一个类型V，而Runnable不可以
2.Callable能够抛出checked exception,而Runnable不可以。
3.Runnable是自从java1.1就有了，而Callable是1.5之后才加上去的
4.Callable和Runnable都可以应用于executors。而Thread类只支持Runnable.

```java
import java.util.concurrent.Callable;  
import java.util.concurrent.ExecutionException;  
import java.util.concurrent.ExecutorService;  
import java.util.concurrent.Executors;  
import java.util.concurrent.Future;  
  
public class ThreadTestB {  
    public static void main(String[] args) {  
        ExecutorService e=Executors.newFixedThreadPool(10);  
        Future f1=e.submit(new MyCallableA());  
        Future f2=e.submit(new MyCallableA());  
        Future f3=e.submit(new MyCallableA());        
        System.out.println("--Future.get()....");  
        try {  
            System.out.println(f1.get());  
            System.out.println(f2.get());  
            System.out.println(f3.get());            
        } catch (InterruptedException e1) {  
            e1.printStackTrace();  
        } catch (ExecutionException e1) {  
            e1.printStackTrace();  
        }  
        e.shutdown();  
    }  
}  
  
class MyCallableA implements Callable<String>{  
    public String call() throws Exception {  
        System.out.println("开始执行Callable");  
        String[] ss={"zhangsan","lisi"};  
        long[] num=new long[2];  
        for(int i=0;i<1000000;i++){  
            num[(int)(Math.random()*2)]++;  
        }  
          
        if(num[0]>num[1]){  
            return ss[0];  
        }else if(num[0]<num[1]){  
            throw new Exception("弃权!");  
        }else{  
            return ss[1];  
        }  
    } 
}  
```
### Java中CyclicBarrier 和 CountDownLatch有什么不同？
CountDownLatch和CyclicBarrier都能够实现线程之间的等待，只不过它们侧重点不同：

- CountDownLatch一般用于某个线程A等待若干个其他线程执行完任务之后，它才执行；
- CyclicBarrier一般用于一组线程互相等待至某个状态，然后这一组线程再同时执行；
- 另外，CountDownLatch是不能够重用的，而CyclicBarrier是可以重用的。

CountDownLatch的用法: 
```java
public class Test {
     public static void main(String[] args) {   
         final CountDownLatch latch = new CountDownLatch(2);
          
         new Thread(){
             public void run() {
                 try {
                     System.out.println("子线程"+Thread.currentThread().getName()+"正在执行");
                    Thread.sleep(3000);
                    System.out.println("子线程"+Thread.currentThread().getName()+"执行完毕");
                    latch.countDown();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
             };
         }.start();
          
         new Thread(){
             public void run() {
                 try {
                     System.out.println("子线程"+Thread.currentThread().getName()+"正在执行");
                     Thread.sleep(3000);
                     System.out.println("子线程"+Thread.currentThread().getName()+"执行完毕");
                     latch.countDown();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
             };
         }.start();
          
         try {
             System.out.println("等待2个子线程执行完毕...");
            latch.await();
            System.out.println("2个子线程已经执行完毕");
            System.out.println("继续执行主线程");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
     }
}
```
CyclicBarrier用法:
```java
public class Test {
    public static void main(String[] args) {
        int N = 4;
        CyclicBarrier barrier  = new CyclicBarrier(N,new Runnable() {
            @Override
            public void run() {
                System.out.println("当前线程"+Thread.currentThread().getName());   
            }
        });
         
        for(int i=0;i<N;i++)
            new Writer(barrier).start();
    }
    static class Writer extends Thread{
        private CyclicBarrier cyclicBarrier;
        public Writer(CyclicBarrier cyclicBarrier) {
            this.cyclicBarrier = cyclicBarrier;
        }
 
        @Override
        public void run() {
            System.out.println("线程"+Thread.currentThread().getName()+"正在写入数据...");
            try {
                Thread.sleep(5000);      //以睡眠来模拟写入数据操作
                System.out.println("线程"+Thread.currentThread().getName()+"写入数据完毕，等待其他线程写入完毕");
                cyclicBarrier.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }catch(BrokenBarrierException e){
                e.printStackTrace();
            }
            System.out.println("所有线程写入完毕，继续处理其他任务...");
        }
    }
}
```

### Java中interrupted和isInterruptedd方法的区别？
interrupt方法用于中断线程。调用该方法的线程的状态为将被置为"中断"状态。
> 注意：线程中断仅仅是置线程的中断状态位，不会停止线程。需要用户自己去监视线程的状态为并做处理。支持线程中断的方法（也就是线程中断后会抛出interruptedException的方法）就是在监视线程的中断状态，一旦线程的中断状态被置为“中断状态”，就会抛出中断异常。

isInterrupted 只是简单的查询中断状态，不会对状态进行修改。

### concurrentHashMap的源码理解以及内部实现原理，为什么他是同步的且效率高
#### ConcurrentHashMap 分析
ConcurrentHashMap的结构是比较复杂的，都深究去本质，其实也就是数组和链表而已。我们由浅入深慢慢的分析其结构。

先简单分析一下，ConcurrentHashMap 的成员变量中，包含了一个 Segment 的数组（final Segment<K,V>[] segments;），而 Segment 是 ConcurrentHashMap 的内部类，然后在 Segment 这个类中，包含了一个 HashEntry 的数组（transient volatile HashEntry<K,V>[] table;）。而 HashEntry 也是ConcurrentHashMap 的内部类。HashEntry 中，包含了 key 和 value 以及 next 指针（类似于 HashMap 中 Entry），所以 HashEntry 可以构成一个链表。

所以通俗的讲，ConcurrentHashMap 数据结构为一个 Segment 数组，Segment 的数据结构为 HashEntry 的数组，而 HashEntry 存的是我们的键值对，可以构成链表。

首先，我们看一下 HashEntry 类。

#### HashEntry

HashEntry 用来封装散列映射表中的键值对。在 HashEntry 类中，key，hash 和 next 域都被声明为 final 型，value 域被声明为 volatile 型。其类的定义为：
```java
static final class HashEntry<K,V> {
        final int hash;
        final K key;
        volatile V value;
        volatile HashEntry<K,V> next;

        HashEntry(int hash, K key, V value, HashEntry<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }
        ...
        ...
}
```

HashEntry 的学习可以类比着 HashMap 中的 Entry。我们的存储键值对的过程中，散列的时候如果发生“碰撞”，将采用“分离链表法”来处理碰撞：把碰撞的 HashEntry 对象链接成一个链表。

如下图，我们在一个空桶中插入 A、B、C 两个 HashEntry 对象后的结构图（其实应该为键值对，在这进行了简化以方便更容易理解）：
![image_1bktj3rvj10jfva56vmqhaq1a9.png-5.9kB][1]
图1

#### Segment

Segment 的类定义为static final class Segment<K,V> extends ReentrantLock implements Serializable。其继承于 ReentrantLock 类，从而使得 Segment 对象可以充当锁的角色。Segment 中包含HashEntry 的数组，其可以守护其包含的若干个桶（HashEntry的数组）。Segment 在某些意义上有点类似于 HashMap了，都是包含了一个数组，而数组中的元素可以是一个链表。

table:table 是由 HashEntry 对象组成的数组如果散列时发生碰撞，碰撞的 HashEntry 对象就以链表的形式链接成一个链表table数组的数组成员代表散列映射表的一个桶每个 table 守护整个 ConcurrentHashMap 包含桶总数的一部分如果并发级别为 16，table 则守护 ConcurrentHashMap 包含的桶总数的 1/16。

count 变量是计算器，表示每个 Segment 对象管理的 table 数组（若干个 HashEntry 的链表）包含的HashEntry 对象的个数。之所以在每个Segment对象中包含一个 count 计数器，而不在 ConcurrentHashMap 中使用全局的计数器，是为了避免出现“热点域”而影响并发性。
```java
/**
 * Segments are specialized versions of hash tables.  This
 * subclasses from ReentrantLock opportunistically, just to
 * simplify some locking and avoid separate construction.
 */
static final class Segment<K,V> extends ReentrantLock implements Serializable {
    /**
     * The per-segment table. Elements are accessed via
     * entryAt/setEntryAt providing volatile semantics.
     */
    transient volatile HashEntry<K,V>[] table;

    /**
     * The number of elements. Accessed only either within locks
     * or among other volatile reads that maintain visibility.
     */
    transient int count;
    transient int modCount;
    /**
     * 装载因子
     */
    final float loadFactor;
}
```
我们通过下图来展示一下插入 ABC 三个节点后，Segment 的示意图：
![image_1bktj6g4i13mrotok2t4b5bc1m.png-25.6kB][2]
图2

其实从我个人角度来说，Segment结构是与HashMap很像的。

#### ConcurrentHashMap

ConcurrentHashMap 的结构中包含的 Segment 的数组，在默认的并发级别会创建包含 16 个 Segment 对象的数组。通过我们上面的知识，我们知道每个 Segment 又包含若干个散列表的桶，每个桶是由 HashEntry 链接起来的一个链表。如果 key 能够均匀散列，每个 Segment 大约守护整个散列表桶总数的 1/16。

下面我们还有通过一个图来演示一下 ConcurrentHashMap 的结构：
![image_1bktj72gfef74gsvvlh5f103513.png-47.7kB][3]
图3

#### 并发写操作
在 ConcurrentHashMap 中，当执行 put 方法的时候，会需要加锁来完成。我们通过代码来解释一下具体过程： 当我们 new 一个 ConcurrentHashMap 对象，并且执行put操作的时候，首先会执行 ConcurrentHashMap 类中的 put 方法，该方法源码为：
```java
/**
 * Maps the specified key to the specified value in this table.
 * Neither the key nor the value can be null.
 *
 * <p> The value can be retrieved by calling the <tt>get</tt> method
 * with a key that is equal to the original key.
 *
 * @param key key with which the specified value is to be associated
 * @param value value to be associated with the specified key
 * @return the previous value associated with <tt>key</tt>, or
 *         <tt>null</tt> if there was no mapping for <tt>key</tt>
 * @throws NullPointerException if the specified key or value is null
 */
@SuppressWarnings("unchecked")
public V put(K key, V value) {
    Segment<K,V> s;
    if (value == null)
        throw new NullPointerException();
    int hash = hash(key);
    int j = (hash >>> segmentShift) & segmentMask;
    if ((s = (Segment<K,V>)UNSAFE.getObject          // nonvolatile; recheck
         (segments, (j << SSHIFT) + SBASE)) == null) //  in ensureSegment
        s = ensureSegment(j);
    return s.put(key, hash, value, false);
}
```
我们通过注释可以了解到，ConcurrentHashMap 不允许空值。该方法首先有一个 Segment 的引用 s，然后会通过 hash() 方法对 key 进行计算，得到哈希值；继而通过调用 Segment 的 put(K key, int hash, V value, boolean onlyIfAbsent)方法进行存储操作。该方法源码为：
```java
final V put(K key, int hash, V value, boolean onlyIfAbsent) {
    //加锁，这里是锁定的Segment而不是整个ConcurrentHashMap
    HashEntry<K,V> node = tryLock() ? null :scanAndLockForPut(key, hash, value);
    V oldValue;
    try {
        HashEntry<K,V>[] tab = table;
        //得到hash对应的table中的索引index
        int index = (tab.length - 1) & hash;
        //找到hash对应的是具体的哪个桶，也就是哪个HashEntry链表
        HashEntry<K,V> first = entryAt(tab, index);
        for (HashEntry<K,V> e = first;;) {
            if (e != null) {
                K k;
                if ((k = e.key) == key ||
                    (e.hash == hash && key.equals(k))) {
                    oldValue = e.value;
                    if (!onlyIfAbsent) {
                        e.value = value;
                        ++modCount;
                    }
                    break;
                }
                e = e.next;
            }
            else {
                if (node != null)
                    node.setNext(first);
                else
                    node = new HashEntry<K,V>(hash, key, value, first);
                int c = count + 1;
                if (c > threshold && tab.length < MAXIMUM_CAPACITY)
                    rehash(node);
                else
                    setEntryAt(tab, index, node);
                ++modCount;
                count = c;
                oldValue = null;
                break;
            }
        }
    } finally {
        //解锁
        unlock();
    }
    return oldValue;
}
```
关于该方法的某些关键步骤，在源码上加上了注释。

需要注意的是：加锁操作是针对的 hash 值对应的某个 Segment，而不是整个 ConcurrentHashMap。因为 put 操作只是在这个 Segment 中完成，所以并不需要对整个 ConcurrentHashMap 加锁。所以，此时，其他的线程也可以对另外的 Segment 进行 put 操作，因为虽然该 Segment 被锁住了，但其他的 Segment 并没有加锁。同时，读线程并不会因为本线程的加锁而阻塞。

正是因为其内部的结构以及机制，所以 ConcurrentHashMap 在并发访问的性能上要比Hashtable和同步包装之后的HashMap的性能提高很多。在理想状态下，ConcurrentHashMap 可以支持 16 个线程执行并发写操作（如果并发级别设置为 16），及任意数量线程的读操作。

#### 总结
在实际的应用中，散列表一般的应用场景是：除了少数插入操作和删除操作外，绝大多数都是读取操作，而且读操作在大多数时候都是成功的。正是基于这个前提，ConcurrentHashMap 针对读操作做了大量的优化。通过 HashEntry 对象的不变性和用 volatile 型变量协调线程间的内存可见性，使得 大多数时候，读操作不需要加锁就可以正确获得值。这个特性使得 ConcurrentHashMap 的并发性能在分离锁的基础上又有了近一步的提高。

ConcurrentHashMap 是一个并发散列映射表的实现，它允许完全并发的读取，并且支持给定数量的并发更新。相比于 HashTable 和用同步包装器包装的 HashMap（Collections.synchronizedMap(new HashMap())），ConcurrentHashMap 拥有更高的并发性。在 HashTable 和由同步包装器包装的 HashMap 中，使用一个全局的锁来同步不同线程间的并发访问。同一时间点，只能有一个线程持有锁，也就是说在同一时间点，只能有一个线程能访问容器。这虽然保证多线程间的安全并发访问，但同时也导致对容器的访问变成串行化的了。

ConcurrentHashMap 的高并发性主要来自于三个方面：

- 用分离锁实现多个线程间的更深层次的共享访问。
- 用 HashEntery 对象的不变性来降低执行读操作的线程在遍历链表期间对加锁的需求。
- 通过对同一个 Volatile 变量的写 / 读访问，协调不同线程间读 / 写操作的内存可见性。

使用分离锁，减小了请求 同一个锁的频率。

通过 HashEntery 对象的不变性及对同一个 Volatile 变量的读 / 写来协调内存可见性，使得 读操作大多数时候不需要加锁就能成功获取到需要的值。由于散列映射表在实际应用中大多数操作都是成功的 读操作，所以 2 和 3 既可以减少请求同一个锁的频率，也可以有效减少持有锁的时间。通过减小请求同一个锁的频率和尽量减少持有锁的时间 ，使得 ConcurrentHashMap 的并发性相对于 HashTable 和用同步包装器包装的 HashMap有了质的提高。

### BlockingQueue的使用？
#### BlockingQueue的原理
阻塞队列（BlockingQueue）是一个支持两个附加操作的队列。这两个附加的操作是：在队列为空时，获取元素的线程会等待队列变为非空。当队列满时，存储元素的线程会等待队列可用。阻塞队列常用于生产者和消费者的场景，生产者是往队列里添加元素的线程，消费者是从队列里拿元素的线程。阻塞队列就是生产者存放元素的容器，而消费者也只从容器里拿元素。

#### BlockingQueue的核心方法： 
1)add(E e): 
添加元素,如果BlockingQueue可以容纳,则返回true,否则报异常 

2)offer(E e):
添加元素,如果BlockingQueue可以容纳,则返回true,否则返回false.

3)put(E e):
添加元素,如果BlockQueue没有空间,则调用此方法的线程被阻断直到BlockingQueue里面有空间再继续. 

4)poll(long timeout, TimeUnit timeUnit):
取走BlockingQueue里排在首位的对象,若不能立即取出,则可以等timeout参数规定的时间,取不到时返回null 

5)take():
取走BlockingQueue里排在首位的对象,若BlockingQueue为空,阻断进入等待状态直到Blocking有新的对象被加入为止 

#### BlockingQueue常用实现类
1)ArrayBlockingQueue:
有界的先入先出顺序队列，构造方法确定队列的大小. 

2)LinkedBlockingQueue:
无界的先入先出顺序队列，构造方法提供两种，一种初始化队列大小，队列即有界；第二种默认构造方法，队列无界（有界即Integer.MAX_VALUE）

4)SynchronousQueue:
特殊的BlockingQueue,没有空间的队列，即必须有取的方法阻塞在这里的时候才能放入元素。

3)PriorityBlockingQueue:
支持优先级的阻塞队列 ，存入对象必须实现Comparator接口 （需要注意的是 队列不是在加入元素的时候进行排序，而是取出的时候，根据Comparator来决定优先级最高的）。

#### BlockingQueue<> 队列的作用
BlockingQueue 实现主要用于生产者-使用者队列，BlockingQueue 实现是线程安全的。所有排队方法都可以使用内部锁或其他形式的并发控制来自动达到它们的目的

这是一个生产者-使用者场景的一个用例。注意，BlockingQueue 可以安全地与多个生产者和多个使用者一起使用 
此用例来自jdk文档
```java
//这是一个生产者类
class Producer implements Runnable {
   private final BlockingQueue queue;
   Producer(BlockingQueue q) { 
       queue = q; 
   }
   public void run() {
     try {
       while(true) { 
           queue.put(produce()); 
       }
     } catch (InterruptedException ex) { 
         ... handle ...
         }
   }
   Object produce() { 
       ... 
   }
 }

 //这是一个消费者类
 class Consumer implements Runnable {
   private final BlockingQueue queue;
   Consumer(BlockingQueue q) { queue = q; }
   public void run() {
     try {
       while(true) { 
           consume(queue.take()); 
       }
     } catch (InterruptedException ex) { 
         ... handle ...
     }
   }
   void consume(Object x) { 
       ... 
   }
 }

 //这是实现类
 class Setup {
   void main() {
     //实例一个非阻塞队列
     BlockingQueue q = new SomeQueueImplementation();
     //将队列传入两个消费者和一个生产者中
     Producer p = new Producer(q);
     Consumer c1 = new Consumer(q);
     Consumer c2 = new Consumer(q);
     new Thread(p).start();
     new Thread(c1).start();
     new Thread(c2).start();
   }
 }
```
### ThreadPool的深入考察？
#### 引言

合理利用线程池能够带来三个好处。第一：降低资源消耗。通过重复利用已创建的线程降低线程创建和销毁造成的消耗。第二：提高响应速度。当任务到达时，任务可以不需要等到线程创建就能立即执行。第三：提高线程的可管理性。线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控。但是要做到合理的利用线程池，必须对其原理了如指掌。

#### 线程池的使用
我们可以通过ThreadPoolExecutor来创建一个线程池。
```java
new  ThreadPoolExecutor(corePoolSize, maximumPoolSize, keepAliveTime, milliseconds,runnableTaskQueue, handler);
```
创建一个线程池需要输入几个参数：

- corePoolSize（线程池的基本大小）：当提交一个任务到线程池时，线程池会创建一个线程来执行任务，即使其他空闲的基本线程能够执行新任务也会创建线程，等到需要执行的任务数大于线程池基本大小时就不再创建。如果调用了线程池的prestartAllCoreThreads方法，线程池会提前创建并启动所有基本线程。
- runnableTaskQueue（任务队列）：用于保存等待执行的任务的阻塞队列。 可以选择以下几个阻塞队列。
    - ArrayBlockingQueue：是一个基于数组结构的有界阻塞队列，此队列按 FIFO（先进先出）原则对元素进行排序。
    - LinkedBlockingQueue：一个基于链表结构的阻塞队列，此队列按FIFO （先进先出） 排序元素，吞吐量通常要高于ArrayBlockingQueue。静态工厂方法Executors.newFixedThreadPool()使用了这个队列。
    - SynchronousQueue：一个不存储元素的阻塞队列。每个插入操作必须等到另一个线程调用移除操作，否则插入操作一直处于阻塞状态，吞吐量通常要高于LinkedBlockingQueue，静态工厂方法Executors.newCachedThreadPool使用了这个队列。
    - PriorityBlockingQueue：一个具有优先级的无限阻塞队列。

- maximumPoolSize（线程池最大大小）：线程池允许创建的最大线程数。如果队列满了，并且已创建的线程数小于最大线程数，则线程池会再创建新的线程执行任务。值得注意的是如果使用了无界的任务队列这个参数就没什么效果。
- ThreadFactory：用于设置创建线程的工厂，可以通过线程工厂给每个创建出来的线程设置更有意义的名字。
- RejectedExecutionHandler（饱和策略）：当队列和线程池都满了，说明线程池处于饱和状态，那么必须采取一种策略处理提交的新任务。这个策略默认情况下是AbortPolicy，表示无法处理新任务时抛出异常。以下是JDK1.5提供的四种策略。
    - AbortPolicy：直接抛出异常。
    - CallerRunsPolicy：只用调用者所在线程来运行任务。
    - DiscardOldestPolicy：丢弃队列里最近的一个任务，并执行当前任务。
    - DiscardPolicy：不处理，丢弃掉。
当然也可以根据应用场景需要来实现RejectedExecutionHandler接口自定义策略。如记录日志或持久化不能处理的任务。
- keepAliveTime（线程活动保持时间）：线程池的工作线程空闲后，保持存活的时间。所以如果任务很多，并且每个任务执行的时间比较短，可以调大这个时间，提高线程的利用率。
- TimeUnit（线程活动保持时间的单位）：可选的单位有天（DAYS），小时（HOURS），分钟（MINUTES），毫秒(MILLISECONDS)，微秒(MICROSECONDS, 千分之一毫秒)和毫微秒(NANOSECONDS, 千分之一微秒)。
#### 向线程池提交任务
我们可以使用execute提交的任务，但是execute方法没有返回值，所以无法判断任务是否被线程池执行成功。通过以下代码可知execute方法输入的任务是一个Runnable类的实例。
```java
threadsPool.execute(new Runnable() {
    @Override
    public void run() {
        // TODO Auto-generated method stub
    }
});
```
我们也可以使用submit 方法来提交任务，它会返回一个future,那么我们可以通过这个future来判断任务是否执行成功，通过future的get方法来获取返回值，get方法会阻塞住直到任务完成，而使用get(long timeout, TimeUnit unit)方法则会阻塞一段时间后立即返回，这时有可能任务没有执行完。
```
Future<Object> future = executor.submit(harReturnValuetask);
try {
     Object s = future.get();
} catch (InterruptedException e) {
    // 处理中断异常
} catch (ExecutionException e) {
    // 处理无法执行任务异常
} finally {
    // 关闭线程池
    executor.shutdown();
}
```
#### 线程池的关闭

我们可以通过调用线程池的shutdown或shutdownNow方法来关闭线程池，它们的原理是遍历线程池中的工作线程，然后逐个调用线程的interrupt方法来中断线程，所以无法响应中断的任务可能永远无法终止。但是它们存在一定的区别，shutdownNow首先将线程池的状态设置成STOP，然后尝试停止所有的正在执行或暂停任务的线程，并返回等待执行任务的列表，而shutdown只是将线程池的状态设置成SHUTDOWN状态，然后中断所有没有正在执行任务的线程。

只要调用了这两个关闭方法的其中一个，isShutdown方法就会返回true。当所有的任务都已关闭后,才表示线程池关闭成功，这时调用isTerminaed方法会返回true。至于我们应该调用哪一种方法来关闭线程池，应该由提交到线程池的任务特性决定，通常调用shutdown来关闭线程池，如果任务不一定要执行完，则可以调用shutdownNow。

#### 线程池的分析

流程分析：线程池的主要工作流程如下图：

![image_1bktlddu61e801ovg2jjbh0155020.png-65.2kB][4]

从上图我们可以看出，当提交一个新任务到线程池时，线程池的处理流程如下：

1. 首先线程池判断基本线程池是否已满？没满，创建一个工作线程来执行任务。满了，则进入下个流程。
2. 其次线程池判断工作队列是否已满？没满，则将新提交的任务存储在工作队列里。满了，则进入下个流程。
3. 最后线程池判断整个线程池是否已满？没满，则创建一个新的工作线程来执行任务，满了，则交给饱和策略来处理这个任务。

#### 源码分析
上面的流程分析让我们很直观的了解了线程池的工作原理，让我们再通过源代码来看看是如何实现的。线程池执行任务的方法如下：
```
public void execute(Runnable command) {
    if (command == null)
       throw new NullPointerException();
    //如果线程数小于基本线程数，则创建线程并执行当前任务 
    if (poolSize >= corePoolSize || !addIfUnderCorePoolSize(command)) {
    //如线程数大于等于基本线程数或线程创建失败，则将当前任务放到工作队列中。
        if (runState == RUNNING && workQueue.offer(command)) {
            if (runState != RUNNING || poolSize == 0)
                      ensureQueuedTaskHandled(command);
        }
    //如果线程池不处于运行中或任务无法放入队列，并且当前线程数量小于最大允许的线程数量，
则创建一个线程执行任务。
        else if (!addIfUnderMaximumPoolSize(command))
        //抛出RejectedExecutionException异常
            reject(command); // is shutdown or saturated
    }
}
```
工作线程。线程池创建线程时，会将线程封装成工作线程Worker，Worker在执行完任务后，还会无限循环获取工作队列里的任务来执行。我们可以从Worker的run方法里看到这点：
```
public void run() {
     try {
           Runnable task = firstTask;
           firstTask = null;
            while (task != null || (task = getTask()) != null) {
                    runTask(task);
                    task = null;
            }
      } finally {
             workerDone(this);
      }
} 
```
#### 合理的配置线程池

要想合理的配置线程池，就必须首先分析任务特性，可以从以下几个角度来进行分析：

1. 任务的性质：CPU密集型任务，IO密集型任务和混合型任务。
2. 任务的优先级：高，中和低。
3. 任务的执行时间：长，中和短。
4. 任务的依赖性：是否依赖其他系统资源，如数据库连接。

任务性质不同的任务可以用不同规模的线程池分开处理。CPU密集型任务配置尽可能小的线程，如配置Ncpu+1个线程的线程池。IO密集型任务则由于线程并不是一直在执行任务，则配置尽可能多的线程，如2*Ncpu。混合型的任务，如果可以拆分，则将其拆分成一个CPU密集型任务和一个IO密集型任务，只要这两个任务执行的时间相差不是太大，那么分解后执行的吞吐率要高于串行执行的吞吐率，如果这两个任务执行时间相差太大，则没必要进行分解。我们可以通过Runtime.getRuntime().availableProcessors()方法获得当前设备的CPU个数。

优先级不同的任务可以使用优先级队列PriorityBlockingQueue来处理。它可以让优先级高的任务先得到执行，需要注意的是如果一直有优先级高的任务提交到队列里，那么优先级低的任务可能永远不能执行。

执行时间不同的任务可以交给不同规模的线程池来处理，或者也可以使用优先级队列，让执行时间短的任务先执行。

依赖数据库连接池的任务，因为线程提交SQL后需要等待数据库返回结果，如果等待的时间越长CPU空闲时间就越长，那么线程数应该设置越大，这样才能更好的利用CPU。

建议使用有界队列，有界队列能增加系统的稳定性和预警能力，可以根据需要设大一点，比如几千。有一次我们组使用的后台任务线程池的队列和线程池全满了，不断的抛出抛弃任务的异常，通过排查发现是数据库出现了问题，导致执行SQL变得非常缓慢，因为后台任务线程池里的任务全是需要向数据库查询和插入数据的，所以导致线程池里的工作线程全部阻塞住，任务积压在线程池里。如果当时我们设置成无界队列，线程池的队列就会越来越多，有可能会撑满内存，导致整个系统不可用，而不只是后台任务出现问题。当然我们的系统所有的任务是用的单独的服务器部署的，而我们使用不同规模的线程池跑不同类型的任务，但是出现这样问题时也会影响到其他任务。

#### 线程池的监控

通过线程池提供的参数进行监控。线程池里有一些属性在监控线程池的时候可以使用

- taskCount：线程池需要执行的任务数量。
- completedTaskCount：线程池在运行过程中已完成的任务数量。小于或等于taskCount。
- largestPoolSize：线程池曾经创建过的最大线程数量。通过这个数据可以知道线程池是否满过。如等于线程池的最大大小，则表示线程池曾经满了。
- getPoolSize:线程池的线程数量。如果线程池不销毁的话，池里的线程不会自动销毁，所以这个大小只增不+getActiveCount：获取活动的线程数。

通过扩展线程池进行监控。通过继承线程池并重写线程池的beforeExecute，afterExecute和terminated方法，我们可以在任务执行前，执行后和线程池关闭前干一些事情。如监控任务的平均执行时间，最大执行时间和最小执行时间等。这几个方法在线程池里是空方法。如：
```
protected void beforeExecute(Thread t, Runnable r) { }
```

### Java中Semaphore是什么？
Java中的Semaphore是一种新的同步类，它是一个计数信号。

从概念上讲，信号量维护了一个许可集合。如有必要，在许可可用前会阻塞每一个 acquire()，然后再获取该许可。每个 release()添加一个许可，从而可能释放一个正在阻塞的获取者。

但是，不使用实际的许可对象，Semaphore只对可用许可的号码进行计数，并采取相应的行动。

信号量常常用于多线程的代码中，比如数据库连接池。

### 结束


  [1]: http://static.zybuluo.com/homiss/6binemonffgaggt0h9dsbezk/image_1bktj3rvj10jfva56vmqhaq1a9.png
  [2]: http://static.zybuluo.com/homiss/v0iqya8gxbk66vfb9r98e50h/image_1bktj6g4i13mrotok2t4b5bc1m.png
  [3]: http://static.zybuluo.com/homiss/qyohusvww2z2315hkv7uej2y/image_1bktj72gfef74gsvvlh5f103513.png
  [4]: http://static.zybuluo.com/homiss/uqokjt20jokos5dn7r3chfjd/image_1bktlddu61e801ovg2jjbh0155020.png