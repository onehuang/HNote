---
layout: post
title:  java多线程编程
excerpt: java多线程有关的学习与总结。
categories: [thread, java]
tags: [java, spring]
comments: true
---
## 多线程的面试题总结（来自网络）

##### 1.什么是线程？

&ensp;&ensp;&ensp;&ensp;线程是操作系统调度的最小单位，它被包含在进程中，是进程实际运行单位。线程自己不拥有系统资源，但它可与同属一个进程的其它线程共享进程所拥有的全部资源。一个线程可以创建和撤消另一个线程，同一进程中的多个线程之间可以并发执行。线程有就绪、阻塞和运行三种基本状态。

线程是一个动态执行的过程，它也有一个从产生到死亡的过程。  
下图显示了一个线程完整的生命周期。  
![来自网络的截图]({{ site.url }}/img/posts/java-thread.jpg)

> +  **新建状态:**   
使用 new 关键字和 Thread 类或其子类建立一个线程对象后，该线程对象就处于新建状态。它保持这个状态直到程序 start() 这个线程。
> + **就绪状态:**  
当线程对象调用了start()方法之后，该线程就进入就绪状态。就绪状态的线程处于就绪队列中，要等待JVM里线程调度器的调度。
> + **运行状态:**  
如果就绪状态的线程获取 CPU 资源，就可以执行 run()，此时线程便处于运行状态。处于运行状态的线程最为复杂，它可以变为阻塞状态、就绪状态和死亡状态。
> + **阻塞状态:**  
如果一个线程执行了sleep（睡眠）、suspend（挂起）等方法，失去所占用资源之后，该线程就从运行状态进入阻塞状态。在睡眠时间已到或获得设备资源后可以重新进入就绪状态。可以分为三种：  
等待阻塞：运行状态中的线程执行 wait() 方法，使线程进入到等待阻塞状态。  
同步阻塞：线程在获取 synchronized 同步锁失败(因为同步锁被其他线程占用)。  
其他阻塞：通过调用线程的 sleep() 或 join() 发出了 I/O 请求时，线程就会进入到阻塞状态。当sleep() 状态超时，join() 等待线程终止或超时，或者 I/O 处理完毕，线程重新转入就绪状态。
> + **死亡状态:**  
一个运行状态的线程完成任务或者其他终止条件发生时，该线程就切换到终止状态。

附加：  
进程：是指一个内存中运行的应用程序，每个进程都有自己独立的一块内存空间，一个进程中可以启动多个线程。比如在Windows系统中，一个运行的exe就是一个进程。

线程中几个重要的概念：  
* 线程同步
* 线程间通信
* 线程死锁
* 线程控制：挂起、停止和恢复

***

##### 2.线程和进程有什么区别？
&ensp;&ensp;&ensp;&ensp;线程是进程的子集，一个进程可以有很多线程，每个线程并行执行不同的任务。不同的进程使用不同的内存空间，而所有的线程共享一片相同的内存空间（进程的没存空间）。别把它和栈内存搞混，每个线程都拥有单独的栈内存用来存储本地数据。

***
##### 3.如何在Java中实现线程？

* 通过实现 Runnable 接口
* 通过继承 Thread 类本身 
* 通过 Callable 和 Future 创建线程 

**(1)通过实现Runable接口**   
&ensp;&ensp;&ensp;&ensp;一个类需要实现run()方法.  

{% highlight java %}
    public void run()
{% endhighlight %}

run()方法可以像其他方法一样调用，但是这样只是在主线程中执行。  
在创建一个实现 Runnable 接口的类之后，你可以在类中实例化一个线程对象。  
Thread 定义了几个构造方法，下面的是我们经常使用的：  
{% highlight java %}
	public Thread() {
	    init(null, null, "Thread-" + nextThreadNum(), 0);
	}

	public Thread(Runnable target) {
	    init(null, target, "Thread-" + nextThreadNum(), 0);
	}

	Thread(Runnable target, AccessControlContext acc) {
	    init(null, target, "Thread-" + nextThreadNum(), 0, acc);
	}

	public Thread(Runnable target, String name) {
	    init(null, target, name, 0);
	}
	...
{% endhighlight %}

其中，threadOb 是一个实现 Runnable 接口的类的实例，并且 name 指定新线程的名字。  
新线程创建之后，你调用它的 start() 方法它才会运行。  

{% highlight java %}
	public synchronized void start() {
	    /**
	     * This method is not invoked for the main method thread or "system"
	     * group threads created/set up by the VM. Any new functionality added
	     * to this method in the future may have to also be added to the VM.
	     *
	     * A zero status value corresponds to state "NEW".
	     */
	    if (threadStatus != 0)
	        throw new IllegalThreadStateException();

	    /* Notify the group that this thread is about to be started
	     * so that it can be added to the group's list of threads
	     * and the group's unstarted count can be decremented. */
	    group.add(this);

	    boolean started = false;
	    try {
	        start0();
	        started = true;
	    } finally {
	        try {
	            if (!started) {
	                group.threadStartFailed(this);
	            }
	        } catch (Throwable ignore) {
	            /* do nothing. If start0 threw a Throwable then
	              it will be passed up the call stack */
	        }
	    }
	}
{% endhighlight %}

**(2)通过继承Thread来创建线程**   
&ensp;&ensp;&ensp;&ensp;创建一个线程的第二种方法是创建一个新的类，该类继承 Thread 类，然后创建一个该类的实例。  
继承类必须重写 run() 方法，该方法是新线程的入口点。它也必须调用 start() 方法才能执行。  
该方法尽管被列为一种多线程实现方式，但是本质上也是实现了 Runnable 接口的一个实例。

***

**(3)通过 Callable 和 Future 创建线程**   
* 创建 Callable 接口的实现类，并实现 call() 方法，该 call() 方法将作为线程执行体，并且有返回值。  
* 创建 Callable 实现类的实例，使用 FutureTask 类来包装 Callable 对象，该 FutureTask 对象封装了该 Callable 对象的 call() 方法的返回值。  
* 使用 FutureTask 对象作为 Thread 对象的 target 创建并启动新线程。  
* 调用 FutureTask 对象的 get() 方法来获得子线程执行结束后的返回值。  
下面是一个例子
{% highlight java%}
package com.hzb;

import java.util.Random;
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.FutureTask;

/**
 * Created by zibeen on 2017/10/6.
 */
public class javaThread  {
    public  static  void main(String[] args) {
        ThreadCallable tc = new ThreadCallable();
        FutureTask ft = new FutureTask(tc);
        new Thread(ft, "子线程计算：").start();
        try {
            while (!ft.isDone()){
                System.out.println("子线程正在计算，等待...");
                Thread.sleep(1000);
            }
            System.out.println("子线程计算完毕，计算结果："+":"+ ft.get());
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
//        ThreadRunable tr = new ThreadRunable();
//        new Thread(tr, "子线程:"+i).start();
    }
}
class ThreadCallable implements Callable<Integer>{
    private  Integer it;
    public ThreadCallable(){
       System.out.println("ThreadCallable构造函数");
    }
    @Override
    public Integer call() throws Exception {

        System.out.println(Thread.currentThread().getName() );
        int num = new Random().nextInt(100);
        System.out.println("随机数：" + num);
        for(int i = 0; i < 100; i++){
            if( i == num){
                System.out.println(Thread.currentThread().getName() + ":" + i);
                return  i;
            }else{
                Thread.sleep(100);
            }
        }
        return -1;
    }
}

class ThreadRunable implements Runnable{
    private  int i = 0;
    @Override
    public void run() {
        while (true){
            if(i == 10) return;
            i++;
            System.out.println(Thread.currentThread().getName()+":"+ i);
            try {
                Thread.sleep(500);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

{% endhighlight %}
**创建线程的三种方式的对比**  
1. 采用实现 Runnable、Callable 接口的方式创见多线程时，线程类只是实现了 Runnable 接口或 Callable 接口，还可以继承其他类。
2. 使用继承 Thread 类的方式创建多线程时，编写简单，如果需要访问当前线程，则无需使用 Thread.currentThread() 方法，直接使用 this 即可获得当前线程。  

***

##### 4.用Runnable还是Thread？  
&ensp;&ensp;&ensp;&ensp;这个问题是上题的后续，大家都知道我们可以通过继承Thread类或者调用Runnable接口来实现线程，问题是，那个方法更好呢？什么情况下使用它？这个问题很容易回答，如果你知道Java不支持类的多重继承，但允许你调用多个接口。所以如果你要继承其他类，当然是调用Runnable接口好了。

***

##### 5.Thread 类中的start() 和 run() 方法有什么区别？  
&ensp;&ensp;&ensp;&ensp;start()方法被用来启动新创建的线程，而且start()内部调用了run()方法，这和直接调用run()方法的效果不一样。当你调用run()方法的时候，只会是在原来的线程中调用，没有新的线程启动，start()方法才会启动新线程。

***

##### 6.Java中Runnable和Callable有什么不同？  
&ensp;&ensp;&ensp;&ensp;sRunnable和Callable都代表那些要在不同的线程中执行的任务。Runnable从JDK1.0开始就有了，Callable是在JDK1.5增加的。它们的主要区别是Callable的 call() 方法可以返回值和抛出异常，而Runnable的run()方法没有这些功能。Callable可以返回装载有计算结果的Future对象。  

Runnable和Callable的区别：
* (1)Callable规定的方法是call(),Runnable规定的方法是run()。  
* (2)Callable的任务执行后可返回值，而Runnable的任务是不能返回值得。 
* (3)call方法可以抛出异常，run方法不可以。  
* (4)运行Callable任务可以拿到一个Future对象，Future 表示异步计算的结果。

***

##### 7.Java中CyclicBarrier 和 CountDownLatch有什么不同？    
&ensp;&ensp;&ensp;&ensp;CyclicBarrier 和 CountDownLatch 都可以用来让一组线程等待其它线程。与 CyclicBarrier 不同的是，CountdownLatch 不能重新使用,CyclicBarrier可以重复使用。
[实例代码]中类ThreadCountDownLatch，ThreadCyclicBarrier和ThreadSemaphore 具体例子。

* CountDownLatch和CyclicBarrier都能够实现线程之间的等待，只不过它们侧重点不同：  
 CountDownLatch一般用于某个线程A等待若干个其他线程执行完任务之后，它才执行；
 而CyclicBarrier一般用于一组线程互相等待至某个状态，然后这一组线程再同时执行；
 另外，CountDownLatch是不能够重用的，而CyclicBarrier是可以重用的。
* Semaphore其实和锁有点类似，它一般用于控制对某组资源的访问权限。  

***

#### 8.Java内存模型是什么？  
&ensp;&ensp;&ensp;&ensp;Java内存模型规定和指引Java程序在不同的内存架构、CPU和操作系统间有确定性地行为。它在多线程的情况下尤其重要。Java内存模型对一个线程所做的变动能被其它线程可见提供了保证，它们之间是先行发生关系。这个关系定义了一些规则让程序员在并发编程时思路更清晰。比如，先行发生关系确保了：  
* 线程内的代码能够按先后顺序执行，这被称为程序次序规则。
* 对于同一个锁，一个解锁操作一定要发生在时间上后发生的另一个锁定操作之前，也叫做管程锁定规则。
* 前一个对volatile的写操作在后一个volatile的读操作之前，也叫volatile变量规则。
* 一个线程内的任何操作必需在这个线程的start()调用之后，也叫作线程启动规则。
* 一个线程的所有操作都会在线程终止之前，线程终止规则。
* 一个对象的终结操作必需在这个对象构造完成之后，也叫对象终结规则。
* 可传递性。
[参考资料1] 

***

##### 9.Java中的volatile 变量是什么？  
&ensp;&ensp;&ensp;&ensp;volatile是一个特殊的修饰符，只有成员变量才能使用它。在Java并发程序缺少同步类的情况下，多线程对成员变量的操作对其它线程是透明的。volatile变量可以保证下一个读取操作会在前一个写操作之后发生，就是上一题的volatile变量规则。

***

##### 10.什么是线程安全？Vector是一个线程安全类吗？  
&ensp;&ensp;&ensp;&ensp;如果你的代码所在的进程中有多个线程在同时运行，而这些线程可能会同时运行这段代码。如果每次运行结果和单线程运行的结果是一样的，而且其他的变量的值也和预期的是一样的，就是线程安全的。一个线程安全的计数器类的同一个实例对象在被多个线程使用的情况下也不会出现计算失误。很显然你可以将集合类分成两组，线程安全和非线程安全的。Vector 是用同步方法来实现线程安全的, 而和它相似的ArrayList不是线程安全的。 

***

##### 11.Java中什么是竞态条件？ 举个例子说明。  
&ensp;&ensp;&ensp;&ensp;当两个线程竞争同一资源时，如果对资源的访问顺序敏感，就称存在竞态条件。导致竞态条件发生的代码区称作临界区。在临界区中使用适当的同步就可以避免竞态条件。
临界区实现方法有两种，一种是用synchronized，一种是用Lock显式锁实现。   

***

##### 12.Java中如何停止一个线程？
&ensp;&ensp;&ensp;&ensp;Java提供了很丰富的API但没有为停止线程提供API。JDK 1.0本来有一些像stop(), suspend() 和 resume()的控制方法但是由于潜在的死锁威胁因此在后续的JDK版本中他们被弃用了，之后Java API的设计者就没有提供一个兼容且线程安全的方法来停止一个线程。当run() 或者 call() 方法执行完的时候线程会自动结束,如果要手动结束一个线程，你可以用volatile 布尔变量来退出run()方法的循环或者是取消任务来中断线程。  

##### 13.一个线程运行时发生异常会怎样？
&ensp;&ensp;&ensp;&ensp;如果异常没有被捕获该线程将会停止执行。Thread.UncaughtExceptionHandler的uncaughtException()是用于处理未捕获异常造成线程突然中断情况的一个内嵌接口。当一个未捕获异常将造成线程中断的时候JVM会使用Thread.getUncaughtExceptionHandler()来查询线程的UncaughtExceptionHandler并将线程和异常作为参数传递给处理程序的uncaughtException()方法进行处理。 
{% highlight java %}
class ExceptionHandler implements Thread.UncaughtExceptionHandler
{
   public void uncaughtException(Thread t, Throwable e)
   {
      //异常后 重启线程
      new Thread(new Task()).start();
   }
}

class Task implements Runnable
{
   @Override
   public void run()
   {
      Thread.currentThread().setUncaughtExceptionHandler(new ExceptionHandler());
      System.out.println(Integer.parseInt("XYZ")); //This will cause NumberFormatException
   }
}

{% endhighlight %}

***

 ##### 14.如何在两个线程间共享数据？
&ensp;&ensp;&ensp;&ensp;你可以通过共享对象来实现这个目的，或者是使用像阻塞队列这样并发的数据结构。用wait和notify方法实现了生产者消费者模型。 [实例代码] ThreadDataExchange类多个线程对一个值增加，另外一些线程对该变量减小。  

***

##### 15.Java中notify 和 notifyAll有什么区别？    
&ensp;&ensp;&ensp;&ensp;因为多线程可以等待单监控锁，Java API 的设计人员提供了一些方法当等待条件改变的时候通知它们，但是这些方法没有完全实现。notify()方法不能唤醒某个具体的线程，所以只有一个线程在等待的时候它才有用武之地。而notifyAll()唤醒所有线程并允许他们争夺锁确保了至少有一个线程能继续运行。  

***

##### 16.为什么wait, notify 和 notifyAll这些方法不在thread类里面？  
&ensp;&ensp;&ensp;&ensp;这是个设计相关的问题，它考察的是面试者对现有系统和一些普遍存在但看起来不合理的事物的看法。回答这些问题的时候，你要说明为什么把这些方法放在Object类里是有意义的，还有不把它放在Thread类里的原因。一个很明显的原因是JAVA提供的锁是对象级的而不是线程级的，每个对象都有锁，通过线程获得。如果线程需要等待某些锁那么调用对象中的wait()方法就有意义了。如果wait()方法定义在Thread类中，线程正在等待的是哪个锁就不明显了。简单的说，由于wait，notify和notifyAll都是锁级别的操作，所以把他们定义在Object类中因为锁属于对象。  

***

##### 17.什么是ThreadLocal变量？  
&ensp;&ensp;&ensp;&ensp;ThreadLocal是Java里一种特殊的变量，线程局部变量。每个线程都有一个ThreadLocal就是每个线程都拥有了自己独立的一个变量，竞争条件被彻底消除了。它是为创建代价高昂的对象获取线程安全的好方法，比如你可以用ThreadLocal让SimpleDateFormat变成线程安全的，因为那个类创建代价高昂且每次调用都需要创建不同的实例所以不值得在局部范围使用它，如果为每个线程提供一个自己独有的变量拷贝，将大大提高效率。首先，通过复用减少了代价高昂的对象的创建个数。其次，你在没有使用高代价的同步或者不变性的情况下获得了线程安全。线程局部变量的另一个不错的例子是ThreadLocalRandom类，它在多线程环境中减少了创建代价高昂的Random对象的个数。  
概括起来说，对于多线程资源共享的问题，同步机制采用了“以时间换空间”的方式，而ThreadLocal采用了“以空间换时间”的方式。前者仅提供一份变量，让不同的线程排队访问，而后者为每一个线程都提供了一份变量，因此可以同时访问而互不影响。 
    Math.Random() 使用原子变量来保存当前的种子，这样两个线程同时调用序列时得到的是伪随机数，而不是相同数量的两倍。  
    ThreadLocalRandom.current() 不再有从多个线程访问同一个随机数生成器实例的争夺。 
***
##### 18.什么是FutureTask？  
&ensp;&ensp;&ensp;&ensp;在Java并发程序中FutureTask表示一个可以取消的异步运算。它有启动和取消运算、查询运算是否完成和取回运算结果等方法。只有当运算完成的时候结果才能取回，如果运算尚未完成get方法将会阻塞。一个FutureTask对象可以对调用了Callable和Runnable的对象进行包装，由于FutureTask也是调用了Runnable接口所以它可以提交给Executor来执行。 
***
 #### 19.Java中interrupted 和 isInterruptedd方法的区别？  
&ensp;&ensp;&ensp;&ensp;interrupted() 和 isInterrupted()的主要区别是前者会将中断状态清除而后者不会。Java多线程的中断机制是用内部标识来实现的，调用Thread.interrupt()来中断一个线程就会设置中断标识为true。当中断线程调用静态方法Thread.interrupted()来检查中断状态时，中断状态会被清零。而非静态方法isInterrupted()用来查询其它线程的中断状态且不会改变中断状态标识。简单的说就是任何抛出InterruptedException异常的方法都会将中断状态清零。无论如何，一个线程的中断状态有有可能被其它线程调用中断来改变。  

***
##### 20. 为什么wait和notify方法要在同步块中调用？  
&ensp;&ensp;&ensp;&ensp;主要是因为Java API强制要求这样做，如果你不这么做，你的代码会抛出IllegalMonitorStateException异常。还有一个原因是为了避免wait和notify之间产生竞态条件。  
***
##### 21.为什么你应该在循环中检查等待条件?  
&ensp;&ensp;&ensp;&ensp;处于等待状态的线程可能会收到错误警报和伪唤醒，如果不在循环中检查等待条件，程序就会在没有满足结束条件的情况下退出。因此，当一个等待线程醒来时，不能认为它原来的等待状态仍然是有效的，在notify()方法调用之后和等待线程醒来之前这段时间它可能会改变。这就是在循环中使用wait()方法效果更好的原因，你可以在Eclipse中创建模板调用wait和notify试一试。如果你想了解更多关于这个问题的内容，我推荐你阅读《Effective Java》这本书中的线程和同步章节。
***
##### 22.Java中的同步集合与并发集合有什么区别？  
&ensp;&ensp;&ensp;&ensp;虽然同步和并发集合类提供了线程安全性，但它们之间的差异在于性能，可扩展性以及如何实现线程安全性。同步 HashMap， Hashtable， HashSet， Vector和synchronized ArrayList比其并发对象（如 ConcurrentHashMap， CopyOnWriteArrayList和 CopyOnWriteHashSet）慢得多。这个缓慢的主要原因是锁定; 同步集合会锁定整个集合，例如整个Map或List，而并发集合则不会锁定整个Map或List。他们通过使用先进和复杂的技术，如锁剥离来实现线程安全。例如，ConcurrentHashMap将整个映射分成几个段，并且仅锁定相关的段，这允许多个线程访问相同ConcurrentHashMap的其他段而不进行锁定。  
类似地，CopyOnWriteArrayList允许多个读取器线程在不同步的情况下读取，并且当写入发生时，它将复制整个ArrayList并与较新的一起进行交换。  
因此，如果您在其有利条件下使用并发收集类，例如更多阅读和更新更新，则它们比同步集合更具可伸缩性。  
Collections.synchronizedMap（）等包装器类可以实现Map等的同步。
并发集合的并发旨在提升并发能力。  
记录：(待验证)  
线程安全（concurrentHashMap,hashTable）的map不允许空值，线程不安全（hashMap）的允许。

> **线程安全集合**
vector,statck,hashtable,enumeration  
> **并发集合**
java.util.concurrent包下面的同步集合：ConcurrentHashMap，CopyOnWriteArrayList，CopyOnWriteArraySet等。

***
##### 23.Java中堆和栈有什么不同？    
1. 栈主要存储局部变量和函数调用，堆中主要存储对象。
2. Java中的每个线程都有自己的堆栈，可以使用-Xss JVM参数进行指定，同样，您还可以使用JVM选项-Xm s和-Xmx 指定Java程序的堆大小，其中-Xms 是堆的起始大小，-Xmx 是java堆的最大大小。  
3. 如果栈中没有内存用于存储函数调用或本地变量，则JVM将抛出java.lang.StackOverFlowError，而如果没有更多的堆空间来创建对象，则JVM将抛出java.lang.OutOfMemoryError： Java堆空间。在我的帖子中阅读更多关于如何处理java.lang.OutOfMemoryError的方法来解决Java中的OutOfMemoryError。   
4. 如果使用递归，哪个方法调用自己，可以快速填充栈内存。栈和堆之间的另一个区别是栈内存的大小远小于 Java 中堆内存的大小。 

***
##### 24. 什么是线程池？ 为什么要使用它？  
&ensp;&ensp;&ensp;&ensp;创建线程要花费昂贵的资源和时间，如果任务来了才创建线程那么响应时间会变长，而且一个进程能创建的线程数有限。为了避免这些问题，在程序启动的时候就创建若干线程来响应处理，它们被称为线程池，里面的线程叫工作线程。从JDK1.5开始，Java API提供了Executor框架让你可以创建不同的线程池。比如单线程池，每次处理一个任务；数目固定的线程池或者是缓存线程池（一个适合很多生存期短的任务的程序的可扩展线程池）。  

***
##### 25.  如何写代码来解决生产者消费者问题？  
&ensp;&ensp;&ensp;&ensp;在现实中你解决的许多线程问题都属于生产者消费者模型，就是一个线程生产任务供其它线程进行消费，你必须知道怎么进行线程间通信来解决这个问题。比较低级的办法是用wait和notify来解决这个问题，比较赞的办法是用Semaphore （信号量）或者 BlockingQueue （阻塞队列）来实现生产者消费者模型。    

***
##### 26.如何避免死锁？  
&ensp;&ensp;&ensp;&ensp;死锁是指两个或两个以上的进程在执行过程中，因争夺资源而造成的一种互相等待的现象，若无外力作用，它们都将无法推进下去。这是一个严重的问题，因为死锁会让你的程序挂起无法完成任务，死锁的发生必须满足以下四个条件：    
* 互斥条件：一个资源每次只能被一个进程使用。    
* 请求与保持条件：一个进程因请求资源而阻塞时，对已获得的资源保持不放。    
* 不剥夺条件：进程已获得的资源，在末使用完之前，不能强行剥夺。    
* 循环等待条件：若干进程之间形成一种头尾相接的循环等待资源关系。    
避免死锁最简单的方法就是阻止循环等待条件，将系统中所有的资源设置标志位、排序，规定所有的进程申请资源必须以一定的顺序（升序或降序）做操作来避免死锁。    

***
##### 27.如何避免死锁？Java中活锁和死锁有什么区别？    
&ensp;&ensp;&ensp;&ensp;这是上题的扩展，活锁和死锁类似，不同之处在于处于活锁的线程或进程的状态是不断改变的，活锁可以认为是一种特殊的饥饿。一个现实的活锁例子是两个人在狭小的走廊碰到，两个人都试着避让对方好让彼此通过，但是因为避让的方向都一样导致最后谁都不能通过走廊。简单的说就是，活锁和死锁的主要区别是前者进程的状态可以改变但是却不能继续执行。    

##### 28.怎么检测一个线程是否拥有锁？
&ensp;&ensp;&ensp;&ensp;在java.lang.Thread中有一个方法叫holdsLock()，它返回true如果当且仅当当前线程拥有某个具体对象的锁。  

***
##### 29.你如何在Java中获取线程堆栈？
&ensp;&ensp;&ensp;&ensp;可以用jstack这个工具来获取，它对线程id进行操作，你可以用jps这个工具找到id。  

***
##### 30.JVM中哪个参数是用来控制线程的栈的大小？  
&ensp;&ensp;&ensp;&ensp;-Xss参数用来控制线程的堆栈大小。

***
 ##### 31.Java中synchronized 和 ReentrantLock 有什么不同？  
&ensp;&ensp;&ensp;&ensp;ReentrantLock是在java.util.concurrent包中；  
ReentrantLock 提供了相同的可见性和排序保证作为隐式锁定，由Java中的synchronized关键字获取，它提供了更多的功能，在某些方面有所不同。如前所述，synchronized和ReentrantLock之间的主要区别是能够中断锁定和超时的能力。   
（1）ReentrantLock 和synchronized关键字之间的另一个显着差异是公平性。synchronized 关键字不支持公平性。任何线程一旦释放就可以获取锁定，不能指定偏好，另一方面可以通过指定公平属性使ReentrantLock 公平，同时创建ReentrantLock的实例。公平财产提供锁定最长等待线程，以备争用。  
（2）同步和可重入锁之间的第二个区别是tryLock（）方法。ReentrantLock提供方便的tryLock（）方法，只有当其可用或不被任何其他线程持有时才会获取锁。这减少了在Java应用程序中等待锁定的线程阻塞。  
（3）Java中的ReentrantLock 和synchronized关键字之间还有一个值得注意的区别就是在等待Lock时中断 Thread的能力。在synchronized关键字的情况下，一个线程可以被阻塞等待锁定，无限期，无法控制。ReentrantLock 提供了一种名为lockInterruptibly（）的方法，可以在等待锁定时中断线程。同样，如果某个时间段内没有锁定，那么tryLock（）可以用于超时。  
（4）ReentrantLock还提供了方便的方法来获取所有线程列表等待锁定。  

ReentrantLock的特点： 
* (1)能够中断锁定。  
* (2)等待锁定时能够超时。  
* (3)创造公平的锁的力量。  
* (4)获取等待线程列表的A​​PI。  
* (5)灵活性尝试锁定而不阻塞。  

***
*   Red
*   Green
*   Blue

[参考资料1]: http://www.importnew.com/12773.html 
[实例代码]:   https://github.com/onehuang/GitHubResource/blob/master/src/com/hzb/javaThread.java






