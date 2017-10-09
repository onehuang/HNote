---
layout: post
title:  java多线程编程
excerpt: "   线程是操作系统调度的最小单位，它被包含在进程中，是进程实际运行单位。线程自己不拥有系统资源，但它可与同属一个进程的其它线程共享进程所拥有的全部资源。一个线程可以创建和撤消另一个线程，同一进程中的多个线程之间可以并发执行。线程有就绪、阻塞和运行三种基本状态。"
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

***
**********************************
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
##### 32.有三个线程T1，T2，T3，怎么确保它们按顺序执行？    
&ensp;&ensp;&ensp;&ensp;在多线程中有多种方法让线程按特定顺序执行，你可以用线程类的join()方法在一个线程中启动另一个线程，另外一个线程完成该线程继续执行。为了确保三个线程的顺序你应该先启动最后一个(T3调用T2，T2调用T1)，这样T1就会先完成而T3最后完成。join()也是一种阻塞方式，直到挂接已经叫做死线或指定等待时间的线程结束。  

***
##### 33.Thread类中的yield方法有什么作用？  
&ensp;&ensp;&ensp;&ensp;Yield方法可以暂停当前正在执行的线程对象，让其它有相同优先级的线程执行。它是一个静态方法而且只保证当前线程放弃CPU占用而不能保证使其它线程一定能占用CPU，执行yield()的线程有可能在进入到暂停状态后马上又被执行。  
yield和wait的区别：
* (1) wait定义在java.lang.Object中声明，而yield()是在java.lang.Thread类中声明。    
* (2) wait()有两个重载的版本，一个超时时间和无参的版本；yield()没有被重载。     
* (3) wait()是一个实例方法；yield()是一个静态方法。    
* (4) wait()和yield()的另一个区别是，当一个线程调用wait()时，它将释放监视器。    
* (5) wait()必须在同步块或者同步方法中调用，而yield()不用。  
* (6) wait()尽量在循环里面掉，yield()尽量是在循环外面。  

***
##### 34.Java中ConcurrentHashMap的并发度是什么？  
&ensp;&ensp;&ensp;&ensp;ConcurrentHashMap把实际map划分成若干部分来实现它的可扩展性和线程安全。这种划分是使用并发度获得的，它是ConcurrentHashMap类构造函数的一个可选参数，默认值为16，这样在多线程情况下就能避免争用。   

***
##### 35. Java中Semaphore是什么？    
&ensp;&ensp;&ensp;&ensp;Java中的Semaphore是一种新的同步类，它是一个计数信号。从概念上讲，从概念上讲，信号量维护了一个许可集合。如有必要，在许可可用前会阻塞每一个 acquire()，然后再获取该许可。每个 release()添加一个许可，从而可能释放一个正在阻塞的获取者。但是，不使用实际的许可对象，Semaphore只对可用许可的号码进行计数，并采取相应的行动。信号量常常用于多线程的代码中，比如数据库连接池。  

***
##### 36.如果你提交任务时，线程池队列已满。会时发会生什么？    
&ensp;&ensp;&ensp;&ensp;这个问题问得很狡猾，许多程序员会认为该任务会阻塞直到线程池队列有空位。事实上如果一个任务不能被调度执行那么ThreadPoolExecutor’s submit()方法将会抛出一个RejectedExecutionException异常。   

***
##### 37.Java线程池中submit() 和 execute()方法有什么区别？  
&ensp;&ensp;&ensp;&ensp;两个方法都可以向线程池提交任务，execute()方法的返回类型是void，它定义在Executor接口中, 而submit()方法可以返回持有计算结果的Future对象，它定义在ExecutorService接口中，它扩展了Executor接口，其它线程池类像ThreadPoolExecutor和ScheduledThreadPoolExecutor都有这些方法。   

***
##### 38.什么是阻塞式方法？  
&ensp;&ensp;&ensp;&ensp;阻塞式方法是指程序会一直等待该方法完成期间不做其他事情，ServerSocket的accept()方法就是一直等待客户端连接。这里的阻塞是指调用结果返回之前，当前线程会被挂起，直到得到结果之后才会返回。此外，还有异步和非阻塞式方法在任务完成前就返回。    

***
##### 39.Swing是线程安全的吗？ 为什么？    
&ensp;&ensp;&ensp;&ensp;你可以很肯定的给出回答，Swing不是线程安全的，但是你应该解释这么回答的原因即便面试官没有问你为什么。当我们说swing不是线程安全的常常提到它的组件，这些组件不能在多线程中进行修改，所有对GUI组件的更新都要在AWT线程中完成，而Swing提供了同步和异步两种回调方法来进行更新。  

***
##### 40.Java中invokeAndWait 和 invokeLater有什么区别？  
这两个方法是Swing API 提供给Java开发者用来从当前线程而不是事件派发线程更新GUI组件用的。InvokeAndWait()同步更新GUI组件，比如一个进度条，一旦进度更新了，进度条也要做出相应改变。如果进度被多个线程跟踪，那么就调用invokeAndWait()方法请求事件派发线程对组件进行相应更新。而invokeLater()方法是异步调用更新组件的。    

***
##### 41.Swing API中那些方法是线程安全的？  
这个问题又提到了swing和线程安全，虽然组件不是线程安全的但是有一些方法是可以被多线程安全调用的，比如repaint(), revalidate()。 JTextComponent的setText()方法和JTextArea的insert() 和 append() 方法也是线程安全的。  

***
##### 42. 如何在Java中创建Immutable对象？  
这个问题看起来和多线程没什么关系， 但不变性有助于简化已经很复杂的并发程序。Immutable对象可以在没有同步的情况下共享，降低了对该对象进行并发访问时的同步化开销。可是Java没有@Immutable这个注解符，要创建不可变类，要实现下面几个步骤：通过构造方法初始化所有成员、对变量不要提供setter方法、将所有的成员声明为私有的，这样就不允许直接访问这些成员、在getter方法中，不要直接返回对象本身，而是克隆对象，并返回对象的拷贝。  

***  
##### 43.Java中的ReadWriteLock是什么？  
一般而言，读写锁是用来提升并发程序性能的锁分离技术的成果。Java中的ReadWriteLock是Java 5 中新增的一个接口，一个ReadWriteLock维护一对关联的锁，一个用于只读操作一个用于写。在没有写线程的情况下一个读锁可能会同时被多个读线程持有。写锁是独占的，你可以使用JDK中的ReentrantReadWriteLock来实现这个规则，它最多支持65535个写锁和65535个读锁。
 Lock、synchronized和ReadWriteLock的区别和联系  
1. 它可以锁住一个方法或者一段代码块;有多个线程ThreadA、ThreadB…ThreadN等，它们同时去执行一段被synchronized修饰的代码块，如果这里ThreadB抢到了同步锁，那么其他的线程都必须等待，ThreadB只有两种情况下会释放锁：  
  1）ThreadB执行完了这段代码块，这个锁就会被释放；     
  2）当ThreadB执行这段代码块抛出异常的时候，jvm虚拟机也会释放这个锁。      
如果ThreadB执行时间特别长，那么其他的线程就会一直等待，这样会照成很大的资源浪费，并且没有效率。    

2.Lock和synchronized最大的区别就是当使用synchronized，一个线程抢占到锁资源，其他线程必须等待；而使用Lock，一个线程抢占到锁资源，其他的线程可以不等待或者设置等待时间，实在抢不到可以去做其他的业务逻辑。  
lock接口中有4个方法，如下：  
{% highlight java %}  
public interface Lock {
    void lock();
    void lockInterruptibly() throws InterruptedException;
    boolean tryLock();
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
    void unlock();
}
{% endhighlight %} 

3.它可以实现读写锁，当读取的时候线程会获得read锁，其他线程也可以获得read锁同时并发的去读取，但是写程序运行获取到write锁的时候，其他线程是不能进行操作的，因为write是排它锁，而上面介绍的两种不管你是read还是write没有抢到锁的线程都会被阻塞或者中断，它也是个接口，里面定义了两种方法readLock()和readLock()，他的一个实现类是ReentrantReadWriteLock。  

***  
##### 44.多线程中的忙循环是什么?
忙循环就是程序员用循环让一个线程等待，不像传统方法wait(), sleep() 或 yield() 它们都放弃了CPU控制，而忙循环不会放弃CPU，它就是在运行一个空循环。这么做的目的是为了保留CPU缓存，在多核系统中，一个等待线程醒来的时候可能会在另一个内核运行，这样会重建缓存。为了避免重建缓存和减少等待重建的时间就可以使用它了。  

***  
##### 45.volatile 变量和 atomic 变量有什么不同？    
这是个有趣的问题。首先，volatile 变量和 atomic 变量看起来很像，但功能却不一样。Volatile变量可以确保先行关系，即写操作会发生在后续的读操作之前, 但它并不能保证原子性。例如用volatile修饰count变量那么 count++ 操作就不是原子性的。而AtomicInteger类提供的atomic方法可以让这种操作具有原子性如getAndIncrement()方法会原子性的进行增量操作把当前值加一，其它数据类型和引用变量也可以进行相似操作。    

***  
##### 46.如果同步块内的线程抛出异常会发生什么？  
这个问题坑了很多Java程序员，若你能想到锁是否释放这条线索来回答还有点希望答对。无论你的同步块是正常还是异常退出的，里面的线程都会释放锁，所以对比锁接口我更喜欢同步块，因为它不用我花费精力去释放锁，该功能可以在finally block里释放锁实现。  

***  
##### 47.单例模式的双检锁是什么？  
这个问题在Java面试中经常被问到，但是面试官对回答此问题的满意度仅为50%。一半的人写不出双检锁还有一半的人说不出它的隐患和Java1.5是如何对它修正的。它其实是一个用来创建线程安全的单例的老方法，当单例实例第一次被创建时它试图用单个锁进行性能优化，但是由于太过于复杂在JDK1.4中它是失败的，我个人也不喜欢它。无论如何，即便你也不喜欢它但是还是要了解一下，因为它经常被问到。

***  
##### 48.如何在Java中创建线程安全的Singleton？  
这是上面那个问题的后续，如果你不喜欢双检锁而面试官问了创建Singleton类的替代方法，你可以利用JVM的类加载和静态变量初始化特征来创建Singleton实例，或者是利用枚举类型来创建Singleton，我很喜欢用这种方法。  

***  
##### 49.写出3条你遵循的多线程最佳实践    
这种问题我最喜欢了，我相信你在写并发代码来提升性能的时候也会遵循某些最佳实践。以下三条最佳实践我觉得大多数Java程序员都应该遵循：  
* 给你的线程起个有意义的名字。  
这样可以方便找bug或追踪。OrderProcessor, QuoteProcessor or TradeProcessor 这种名字比 Thread-1. Thread-2 and Thread-3 好多了，给线程起一个和它要完成的任务相关的名字，所有的主要框架甚至JDK都遵循这个最佳实践。
* 避免锁定和缩小同步的范围  
锁花费的代价高昂且上下文切换更耗费时间空间，试试最低限度的使用同步和锁，缩小临界区。因此相对于同步方法我更喜欢同步块，它给我拥有对锁的绝对控制权。  
* 多用同步类少用wait 和 notify
首先，CountDownLatch, Semaphore, CyclicBarrier 和 Exchanger 这些同步类简化了编码操作，而用wait和notify很难实现对复杂控制流的控制。其次，这些类是由最好的企业编写和维护在后续的JDK中它们还会不断优化和完善，使用这些更高等级的同步工具你的程序可以不费吹灰之力获得优化。  
* 多用并发集合少用同步集合
这是另外一个容易遵循且受益巨大的最佳实践，并发集合比同步集合的可扩展性更好，所以在并发编程时使用并发集合效果更好。如果下一次你需要用到map，你应该首先想到用ConcurrentHashMap。      

***  
##### 50.如何强制启动一个线程？
这个问题就像是如何强制进行Java垃圾回收，目前还没有这样得方法，虽然你可以使用System.gc()来进行垃圾回收，但是不保证能成功。在Java里面没有办法强制启动一个线程，它是被线程调度器控制着且Java没有公布相关的API。  

***  
##### 51.Java中的fork join框架是什么？  
fork join框架是JDK7中出现的一款高效的工具，Java开发人员可以通过它充分利用现代服务器上的多处理器。它是专门为了那些可以递归划分成许多子模块设计的，目的是将所有可用的处理能力用来提升程序的性能。fork join框架一个巨大的优势是它使用了工作窃取算法，可以完成更多任务的工作线程可以从其它线程中窃取任务来执行。
[http://www.infoq.com/cn/articles/fork-join-introduction] 

***  
##### 52.Java多线程中调用wait() 和 sleep()方法有什么不同？  
Java程序中wait 和 sleep都会造成某种形式的暂停，它们可以满足不同的需要。wait()方法用于线程间通信，如果等待条件为真且其它线程被唤醒时它会释放锁，而sleep()方法仅仅释放CPU资源或者让当前线程停止执行一段时间，但不会释放锁。 


参考资料：
[http://www.importnew.com/12773.html]

[http://www.importnew.com/12773.html]:http://www.importnew.com/12773.html
[http://www.infoq.com/cn/articles/fork-join-introduction]:http://www.infoq.com/cn/articles/fork-join-introduction
[参考资料1]: http://www.importnew.com/12773.html 
[实例代码]:   https://github.com/onehuang/GitHubResource/blob/master/src/com/hzb/javaThread.java






