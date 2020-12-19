# 目录

[TOC]



# 1. 并发编程

线程之间的通信，对于程序员是完全透明的。

并发编程要解决多线程同时操作共享数据所引发的数据不安全问题，同时也要保证并发效率。

## 1.1 并发安全的措施

使程序并发安全的运行，方法策略不止一种，通常有以下三种策略：

1. 避免使用共享变量，每个线程在自己的私有变量上工作；
2. 如果有可能，把共享数据设计成常量，不可变状态的对象一定是线程安全的；
3. 使用多线程同步策略保证共享数据的安全；



​	本文大部分内容都在讲解第三种情况该如何处理，这也是最常见的一种情况，因为很多时候，我们需要让多个线程共同操作一份数据，又或者是因为业务需求不能把数据设置成常量，所以掌握多线程环境下并发编程的技巧显得尤为重要。

​	尽管使用同步策略保证线程安全是很有必要的，但也请不要忘记，这不是唯一的方法，在条件允许的情况下，方法一和方法二也是既简单又高效的解决方式。

## 1.2 同步机制



### 使用信号量实现互斥

**信号量和PV操作** 

​	为了解决并发编程中的安全问题，荷兰科学家 Dijkstra 最先提出**信号量和PV操作**的概念。信号量是一个非负整数的全局变量，并且只能由PV操作来处理；

​	PV操作是两个操作，即 P操作和 V操作，P操作和 V操作是在信号量上进行操作的两个过程。

​	信号量提供了一种非常方便的方式实现对共享变量的互斥访问。基本思想就是**将共享变量和一个信号量（非负整数）联系起来，然后用 两种特殊操作将相应的临界区包围起来。**

- P操作：如果信号量 s 是非零的，那么将s 减1，并立即返回。如果 s 为0，那么就阻塞这个线程，直到 s 变为非零。
- V操作：将信号量 s 加1。如果有任何线程阻塞等待 s 变为非零，那么该操作会唤醒这些线程中的一个。

以提供互斥为目的的信号量也叫**互斥锁**，P操作就是对互斥锁**加锁**，V操作是对互斥锁**解锁**。



上面的两种操作注意两点：

​	加锁和解锁过程中对信号量的加载、加1和存储信号是不能中断的，必须保证读-改-写操作的**原子性**。

​	而信号量的读和写也必须具有内存**可见性**，即线程总是能读到任意线程对该变量的最后一次写入，而线程对该变量的写入也必须立即刷新到主存中，并使其他线程对该变量的缓存失效。

注：

​	信号量不仅可以实现互斥，还能解决生产者-消费者问题（阻塞队列）、读者-写者问题（读写锁）。前提是这个信号量一定要保证内存可见性，对信号量更新的CAS操作也必须保证原子性。



### 使用对象监视器实现互斥

​	Java中的 `synchronized` 是通过**对象监视器**的机制来实现同步的，它其实是对信号量互斥和调度能力的更高级别的抽象；实际上，监控器可以用信号量来实现。



## 1.3 Dijkstra （迪科斯彻 ）

​	这里说的 Dijkstra 不是最短路径算法，而是一位非常牛的计算机科学家，大部分中国程序员记住这个名字是因为**最短路径算法【Dijkstra 算法】**，而这只是 Dijkstra 的发明之一。

​	**信号量、互斥**等并发概念最初都是在他的论文中出现的（并发编程的先锋人物）。

​	Dijkstra 是计算机领域最具有影响力的人物之一，也是计算机科学的奠基人之一。



<img src="https://gitee.com/Jackpotsss/pic_go/raw/master/img/image-20200312103619508.png" alt="image-20200312103619508"  />



## 1.4 Doug Lea（道格·利）

​	Java并发包（java.util.concurrent）的作者，被称为这个世界上对Java影响力最大的一个人。

​	Doug是一个无私的人，他深知分享知识和分享苹果是不一样的，苹果会越分越少，而自己的知识并不会因为给了别人就减少了，**知识的分享更能激荡出不一样的火花。**



<img src="https://gitee.com/Jackpotsss/pic_go/raw/master/img/image-20200312103517177.png" alt="image-20200312103517177"  />



​	在 《Effective Java 》第一版中，作者特别感谢了一个人- Doug Lea；在第二版中，作者特别感谢了两个人，其中一位是 Doug Lea；在第三版中，作者特别感谢了三个人，其中一位是 Doug Lea。

​	作者 Joshua Bloch 特别感谢 Doug Lea 无私的将其时间和知识都毫不保留地**分享**给他，在JDK 1.5 、1.6、 1.7 、1.8 版本的源码中，均有 Doug Lea  的参与，Joshua Bloch 和 Doug Lea 无论是在知识交流、撰写书籍还是JDK源码设计方面，都有非常密切的合作。JDK 很多重要的类也是由二人共同设计的（如 ThreadLocal，PriorityQueue等）。



# 2. 线程 Thread

- ​	线程（Thread）是运行在进程上下文中的一个单一顺序的逻辑控制流。
- ​	一个进程中可以同时运行多个线程。
- ​	线程和进程都是由操作系统内核自动调度。
- ​	每个线程都有自己的线程上下文：包括线程ID、栈、栈指针、程序计数器、寄存器和条件码。
- ​	所有运行在一个进程里的线程共享该进程的整个虚拟内存地址空间，都能读写相同的共享数据。



**线程和进程的区别**

- 一个线程的上下文要比一个进程的上下文小的多，所以线程的上下文切换要比进程的上下文切换快得多。

- 线程不像进程那样，并不是按照严格的父子层次来组织的。

  每个线程都是对等的，一个线程可以创建其他线程，也可以杀死其他任意线程，或者被其他任意线程给杀死。



## 2.1 线程状态

### Java中的线程

Java线程的生命周期有6种不同的状态，在给定的一个时刻，线程只能处于其中一种状态。

| 线程状态     | 描述                                                         |
| ------------ | ------------------------------------------------------------ |
| NEW          | 初始状态：线程被构建，但是还没有调用start()方法；            |
| RUNNABLE     | 可运行状态：Java线程将操作系统中的就绪和运行中两个状态统称为"可运行状态"； |
| BLOCKED      | 阻塞状态：等待对象监视器锁的线程处于该状态；                 |
| WAITING      | 等待状态：进入该状态的线程需要等待其他线程做出一些特定动作（通知或中断）； |
| TIME_WAITING | 超时等待状态：该状态不同于WAITING，它是可以在指定的时间自行返回的； |
| TERMINATED   | 终止状态：表示当前线程已经执行完毕；                         |



<img src="https://gitee.com/Jackpotsss/pic_go/raw/master/img/image-20200301111955908.png" alt="image-20200301111955908" style="zoom: 33%;" />

State 类是Thread类的内部枚举类:



<img src="https://gitee.com/Jackpotsss/pic_go/raw/master/img/image-20200307102910868.png" alt="image-20200307102910868" style="zoom: 80%;" />

### 操作系统中的线程

<img src="https://gitee.com/Jackpotsss/pic_go/raw/master/img/image-20200412171751115.png" alt="image-20200412171751115" style="zoom: 67%;" />

​	Java 将操作系统中的**运行和就绪**两个状态合并称为 RUNNABLE 状态；将操作系统中的**等待状态**分为 BLOCKED、WAITING、TIMED_WAITING 三种状态；

​	线程创建、调度和销毁的具体实现在操作系统内核中，JVM操作线程也只是调用操作系统的相关接口。



## 2.2 线程分类

Java中的线程分为用户线程和守护线程。

### 用户线程

Java创建的线程默认为用户线程（非守护线程），当Java虚拟机中不存在用户线程时，JVM将会退出。



### 守护线程

当Java虚拟机中不存在用户线程时，JVM中的所有Deamon线程都需要立即终止，JVM退出。

```java
Thread.setDaemon(true);
```

注：Daemon属性必须在启动线程之前设置，不能在启动线程之后设置。

## 2.3 操作线程

### 创建线程对象

```java
public void main() {
	Thread thread = new Thread(new MyTask(), "thread-1");
}
	
class MyTask implements Runnable{

	public void run() {
		doWork();	
	}
}
```

注：创建一个Thread对象不会创建一个新的线程，调用 start（）方法才会创建并启动一个线程。

### 启动线程

```
thread.start();
```

调用start() 方法做两件事：

1. 在JVM中创建一个线程并分配内存；
2. 线程调用run（）方法，此时线程进入`RUNNABLE` 状态。



注：

- start() 方法是线程安全的同步方法；
- start() 方法只能调用一次，重复调用有异常；



​	一个新构造的线程对象是由其parent 线程来进行空间分配的。

​	child线程继承了parent是否为Daemon、优先级、加载资源的contextClassLoader 以及ThreadLocal ，同时分配一个唯一ID来标识这个child线程。至此，一个能够运行的线程对象就初始化好了。

### 中断线程

​	创建线程和执行任务是简单的，大多数情况下，我们都会执行完任务，但有时我们需要提交结束任务。Java提供了中断这种协作机制，能够使一个线程中断另一个线程当前的工作。

**打上中断标识**

​	Thread 线程使用 `interrupt()` 方法“中断线程”，调用该方法并不会真正的中断线程，只是给该线程设置中断标识，仅此而已。线程可以中断自己，也可以中断别的线程。

```java
//仅仅是打上中断标识而已;
thread.interrupt();
```

**判断中断标识** 

​	使用线程的 `isInterrupted()` 和 `interrupted()`  方法判断线程是否被打上中断标记，被中断返回true，没有被中断返回false。

​	不同的是，`isInterrupted()` 方法调用之后不会清除中断标记，而 `interrupted()`  方法会清除中断标记。

```java
//判断线程是否有中断标记
public boolean isInterrupted() {
   return isInterrupted(false);
}
//判断线程是否有中断标记(同时清除中断标记)
public static boolean interrupted() {
   return currentThread().isInterrupted(true);
}

//中断标记在C++代码中维护,通过native方法设置
private native boolean isInterrupted(boolean ClearInterrupted);
```

### 终止线程

#### 过期的suspend()、resume()和stop()

​	看视频时，视频播放器下面都会有暂停、恢复和停止操作，对应到线程的API上就是suspend()、resume()和stop()；不过这三个方法已经被打上**废弃**标记了，表示已经不再建议使用。

​	suspend() 和 resume() 用于暂停和恢复线程，这是一对操作，并且suspend() 和sleep() 一样在休眠期间并不会释放锁资源；可以使用等待、通知机制来代替暂停恢复操作。

​	stop() 是暴力的停止一个线程，可能会导致线程来不及清理资源，工作结果不确定。

#### 安全的中断线程

​	**可以通过判断中断标识或自定义标识的方式来安全的终止一个线程。**

​	当然这需要你的任务本身就有判断是否中断的动作，即该任务是否响应中断。如果被标记为已中断，那么就结束run方法，线程也就结束了本次任务的执行。

```java
	//可响应中断的任务
	static class MyTask implements Runnable {

		@Override
		public void run() {
			Thread currentThread = Thread.currentThread();
			for (int i = 10; i > 0; i--) {
				if (currentThread.isInterrupted()) {
					doSomeThingAtInterrupt();//任务中断,做收尾工作;
					return;
				}
				doWork();
			}
		}
	}
```

​	**通常，中断是实现取消的最合理方式。** 

​	Java内置的 wait， sleep ，join 方法都是可以响应中断的，当发生中断时，他们会抛出 InterruptedException 异常，提供给开发者捕获处理；另外，并发包的很多工具类和组件也都支持着中断，如Future接口的get 方法等，线程中断以及中断处理会一直贯穿整个并发编程。



## 2.4 线程之间的通信



## 2.5 等待/通知机制

### 内置等待/通知机制

Object对象监视器模式的等待/通知机制

| 方法           | 描述                                                         |
| -------------- | ------------------------------------------------------------ |
| wait()         | 调用该方法的线程进入WAITING 状态，只有等待另外线程的通知或中断才会返回，需要注意，调用wait()方法后，会释放对象锁； |
| wait(long)     | 调用该方法的线程进入TIMED_WAITING 状态，超时等待指定的时间(毫秒)，如果没有通知就超时返回； |
| wait(long,int) | 对于超时时间更细粒度的控制，可以达到纳秒；                   |
| notify()       | 通知一个在该对象上等待的线程，使其从wait()方法返回，而返回的前提是该线程获取到了对象的锁； |
| notifyAll()    | 通知所有在该对象上等待的线程；                               |

注：

​	Object的 wait 方式是可以响应中断的，如果线程被中断，将会放弃等待，并立即抛出 InterruptedException 异常。

### 示例

```java
/**
 * 等待/通知机制
 */
public class WaitNotify {
	
	static boolean flag = false;
	static Object lock = new Object(); 

	public static void main(String[] args) {
		Thread thread1 = new Thread(new WaitTask(), "Thread-wait");
		thread1.start();
		sleep(1000);
		Thread thread2 = new Thread(new NotifyTask(), "Thread-notify");
		thread2.start();
	}
	
	static class WaitTask implements Runnable{

		public void run() {
			
			//加对象锁,获取lock的Monitor
			synchronized (lock) {
				//条件不满足时,一直等待,同时释放锁
				while(!flag) {
					System.out.println("flag is true,wait...");
					try {
						lock.wait();
					} catch (InterruptedException e) {
					}
				}
				//条件满足,继续工作...
				System.out.println("flag is false,running...");
			}
		}
	}
	
	static class NotifyTask implements Runnable{
		
		public void run() {
			
			//加对象锁,获取lock的Monitor
			synchronized (lock) {
				System.out.println("NotifyTask running...");
				lock.notifyAll();
				sleep(2000);
				flag = true;
				System.out.println("NotifyTask ending...");
			}
		}
	}
}
```

输出:

```
flag is true,wait...
NotifyTask running...
NotifyTask ending...
flag is false,running...
```

### 总结

1. wait() 和 notify() 方法调用的前提是当前线程已经获得了对象的监视器锁。
2. 调用wait() 方法后，A线程状态由 RUNNABLE 变为 WAITING ，并释放对象锁。
3. B线程调用 notify() 方法后，等待锁的A线程并不会立即从wait()方法返回，需要**B线程释放锁并且A线程再次成功获得锁之后**，A线程才能从wait()方法返回。



**Object.wait() 和 Thread.sleep(long) 方法的区别:** 

- sleep(long) 方法与锁没有关系，它只是单纯的定时阻塞线程；wait()方法调用的前提是必须获得锁，调用之后再释放锁；而sleep(long) 方法有没有锁都可以调用，并且在持有锁的情况下调用也不会释放锁（抱着锁睡）。
- 唤醒的方式也不一样，sleep(long) 方法和wait(long) 方法都会使线程变为超时等待状态（TIMED_WAITING ）。进入超时等待状态的线程等待指定时间就可以自行返回了，或者线程被中断，这也是sleep(long) 方法的唯一唤醒方式；而wait(long)方法除了等待指定时间自行返回外，期间调用notify() 方法进行通知也会返回；而wait() 方法则只能是一直等待（WAITING 等待状态），直到其他线程调用notify() 方法进行通知返回。



## 2.6 thread.join()

​	如果线程 A 执行threadB.join() 语句，意思是当前线程A释放锁进入等待状态，直到 threadB 线程终止之后才从threadB.join() 返回。除了join() 方法，还提供了 join(long) 具有超时特性的方法，表示如果在指定时间里没有终止，那么也会返回。

### 示例

```java
	public static void main(String[] args) {
		Thread thread1 = new Thread(new Task1(Thread.currentThread()), "Thread-1");
		thread1.start();
		sleep(2000);
		Thread thread2 = new Thread(new Task2(thread1), "Thread-2");
		thread2.start();
	}
	
	static class Task1 implements Runnable{
		
		private Thread thread;	
		public Task1(Thread thread) {
			super();
			this.thread = thread;
		}

		public void run() {	
			System.out.println("Task1 running...");
			try {
				thread.join();	//thread线程终止了,当前线程才可以返回,继续执行
			} catch (InterruptedException e) {
			}
			sleep(1000);
			System.out.println("Task1 ending...");
		}
	}
	
	static class Task2 implements Runnable{
		
		private Thread thread;
		public Task2(Thread thread) {
			super();
			this.thread = thread;
		}
		
		public void run() {		
			System.out.println("Task2 running...");
			try {
				thread.join();
			} catch (InterruptedException e) {
			}
			System.out.println("Task2 ending...");
		}
	}
```

输出:

```
Task1 running...
Task2 running...
Task1 ending...
Task2 ending...
```

​	上面的示例，每个任务都拥有其他线程的引用，在线程A中调用 thread.join() 的意思是当前线程A进入 `WAITING` 等待状态，等待线程B终止之后才能从 thread.join()  返回。

​	Thread除了提供 join() 方法外，还提供了 join(long millis) 和 join(long millis, int nanos) 两个具备超时特性的方法，这两个超时方法表示，如果线程thread 在给定的时间里没有终止，那么将从该超时方法返回。

​	join() 方法支持响应中断。

### join() 源码

下面的源码取自部分核心源码，JDK1.7版本。

```java
//加锁当前线程对象
public final synchronized void join(long millis) throws InterruptedException {
	//线程还活着(条件不满足),继续等待
    while (isAlive()) {
         wait(0);
     }
    //线程已终止(条件满足),方法返回
}
```

​	从源码可以看到，join() 方法也是通过wait() 方法来实现的，join() 方法是线程安全方法，调用该方法要先获取调用对象的锁。

​	join() 设计的精髓在于4个地方：**synchronized、isAlive()、wait(0)和线程终止的处理**；就拿当前线程A调用线程B.join() 为例：synchronized意味着先获取线程B这个对象的监视器锁，isAlive() 判断线程B是否还活着，如果活着，就让当前线程A一直等待，并且join() 方法无法返回；当线程B终止时，会调用自身的notifyAll() 方法，通知所有等待在该线程对象上的线程。此时isAlive() 判断线程B已经终止，join() 方法返回，同时释放线程B这个对象的监视器锁，线程A继续执行。



## 2.7 ThreadLocal 线程本地存储

​	ThreadLocal 即线程本地存储，是一个以 ThreadLocal 对象为键，任意对象为值的存储结构。

​	同一进程的线程共享进程的数据，这种数据共享是多线程编程的优势。然而某些时候，每个线程需要它自己的某些数据，我们称这种数据为线程本地存储（Thread Local Storage）。

​	线程本地存储与局部变量容易混淆，局部变量只是在单个函数体内有效，而线程本地存储在多个函数调用时都可见；线程本地存储的数据是每个线程独有的，互不影响。

​	线程本地存储不是Java线程独有的概念，大多数线程库，如Windows 和 Pthreads 对线程本地存储都有支持。

### 使用

通常将 ThreadLocal 声明为类的全局静态常量：

```java
public class ThreadLocalTest {

	private static final ThreadLocal<Long> threadLocal = new ThreadLocal<Long>();
	
	public void begin() {
		threadLocal.set(System.currentTimeMillis());
	}
	
	public long end() {
		return System.currentTimeMillis() - threadLocal.get();
	}
}
```


​	上面的方式创建出来的 ThreadLocal，如果不set值，get 的返回结果是 null；如果想有一个初始化值，则可以复写 initialValue() 方法；

```java
 public class UserId {
    	
     private static final AtomicInteger nextId = new AtomicInteger(0);

     private static final ThreadLocal<Integer> userId =
         new ThreadLocal<Integer>() {
             @Override protected Integer initialValue() {
                 return nextId.getAndIncrement();
         }
     };

     public static int get() {
         return userId.get();
     }
 }
```



### 实现

​	ThreadLocal是由 Doug Lea 和 Josh Bloch 一起设计的。

​	可以看到 ThreadLocal 的设计和Thread是密不可分的，value 最终保存在 Thread  线程的变量 `ThreadLocal.ThreadLocalMap threadLocals`中，ThreadLocal 获取数据，同样是从当前线程内部变量中获取数据。所以ThreadLocal 实现了将数据以线程为维度进行区分，各线程在本类中各取所需，互不干扰。

```java
    
public class ThreadLocal<T> {
    
	public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);	//获取当前线程,并获取线程内部的 ThreadLocalMap;
        if (map != null)	
            map.set(this, value);	//1.如果线程内部的ThreadLocalMap不为空,直接保存键值对;
        else
            createMap(t, value);	//2.如果为空,创建Map,并保存键值对;
    }
    
    //内部的Map结构
    static class ThreadLocalMap {
       	
    }
 }
```



```java
public class Thread implements Runnable {
    
	ThreadLocal.ThreadLocalMap threadLocals = null;
}
```



# 3. synchronized 内置锁

​	Java提供synchronized 关键字实现同步锁，这些锁被称为**内置锁或对象监视器锁**。线程在进入同步代码块之前会自动获取锁，并且在退出代码块时自动释放锁（无论是正常退出还是抛异常退出）。

​	Java 的内置锁是一个可重进入的互斥锁。这意味着同一时刻最多只有一个线程可以持有这把锁，进入同步代码块；持有该对象内置锁的线程可以反复进入同步代码块；当A线程持有内置锁进入同步代码块时，此时要进入同步代码块的线程B只能阻塞等待，直到线程A释放锁，线程B才有机会去争夺这把锁，只有拿到了内置锁，才能进入同步代码块；如果一直获取不到锁，那么就一直等待。

## 3.1 使用及说明

synchronized 关键字有两种使用方式:

- 同步方法
- 同步代码块

```java
public class SyncDemo {

	private Object lock = new Object();
    
    //同步的实例方法,需要获取调用当前实例方法的实例对象锁
	public synchronized void exec0() {
		doWork();
	}
	
	public void exec1() {
        //同步代码块,需要获取括号内引用对象的对象锁，这里是this，也就是SyncDemo当前的实例对象
		synchronized (this) {
			doWork();
		}
	}
    
	public void exec2() {
        //同步代码块,需要获取括号内引用对象的对象锁，这里是lock的实例对象锁
		synchronized (lock) {
			doWork();
		}
	}
    //同步的静态方法,需要获取调用当前静态方法的Class对象锁
	public static synchronized void exec3() {
		doWork();
	}
	
	public void exec4() {
		//同步代码块,需要获取SyncDemo类的Class对象锁
		synchronized (SyncDemo.class) {
			doWork();
		}
	}
}
```

​	对于 SyncDemo 的同一个实例对象，线程A调用exec0()方法的同时，线程B调用exec1() 会使线程B阻塞，因为两个方法获取的是同一把实例对象锁；而调用exec3()方法的同时，调用exec4() 会使线程阻塞，因为两个方法获取的是同一把Class对象锁；对于同一个实例对象，调用exec1()方法的同时，调用exec2() 不会使线程阻塞，因为两个方法获取的不是同一把对象锁；对于SyncDemo 的两个实例对象，两个线程分别同时调用exec0()方法，不会造成线程阻塞，因为两次调用获取的是两把不同的实例对象锁。



Java中的每一个对象都可以作为锁，叫做**对象锁**或**监视器锁**。具体表现有3种形式：

1. 对于普通实例方法，锁的是当前**实例对象**。
2. 对于类的静态方法，锁的是当前**Class对象**。
3. 对于同步代码块，锁的是 synchronize 括号里引用的实例/Class对象。



**注：阻塞在同步代码块的线程不响应中断，也就是说内置锁不响应中断，即便中断阻塞的线程，也只是打上中断标记，没有任何作用。**

下面举一个栗子说明synchronized的具体用法：

```java
public class A {

	private static boolean isTrue;

	public static synchronized void staticWrite(boolean b) { isTrue = b;}

	public static synchronized boolean staticRead() { return isTrue;}

	public synchronized void write(boolean b) { isTrue = b;}

	public synchronized boolean read() { return isTrue;}
}
```

有下面几种场景：

1. 线程1访问 A.staticWrite(true) 时，线程2能否访问A.staticRead()方法？
2. 线程1访问 A.staticWrite(true) 时，线程2能否访问a.staticRead()方法？
3. 线程1访问 new A().staticWrite(true)时，线程2能否访问A.staticRead()方法？
4. A a= new A()，线程1访问a .staticWrite(true)时，线程2能否访问A.staticRead()方法？
5. A a= new A()，A a1 = new A()，线程1访问a.write(true)时，线程2能否访问a1.read()？
6. A a= new A()，A a1 = new A()，线程1访问a.write(true)时，线程2能否访问a1.write(true)？
7. A a= new A()，线程1访问a.write(true)时，线程2能否访问a.read()？



答案：1 不能  2 不能  3 不能  4 不能  5 能 6 能 7 不能



​	调用`synchronized` 修饰的方法时，需要获取相应的锁，如果是`类.静态安全方法` 或 `对象.静态安全方法`，则尝试获取Class对象这把锁，而 `对象.普通安全方法`，则尝试获取实例对象锁。

1. 线程1访问 A.staticWrite(true) 时，线程2不能够访问该类的任何静态安全方法，因为调用静态安全方法需要获取类这把锁，造成所竞争；但是线程2可以访问对象的任何普通方法（安全、不安全），因为调用普通安全方法需要获取对象锁，和类锁是不同的锁，不会冲突；
2. A a= new A()，线程1访问a.write(true)时，线程2不能否访问a.read()；因为是同一个对象，都尝试获取同一个对象的锁，造成锁竞争；
3. A a= new A()，A a1 = new A()，线程1访问a.write(true)时，线程2可以访问a1.write(true)；这是两个不同的对象，各自获取不同对象的锁，不会冲突；



**内置锁的调试信息** 

Java对象的内置锁信息会在线程的栈帧中保存一份，因此在调试程序的时候可以在栈帧中查看到。

第一个是Class对象锁，第二个是实例对象锁，这是两把不同的锁，两个线程同时分别获取这两把锁时执行不阻塞。

<img src="https://gitee.com/Jackpotsss/pic_go/raw/master/img/image-20200227173342011.png" alt="image-20200227173342011" style="zoom:80%;" />

第一个是实例对象A锁，第二个是实例对象B锁；虽然两个对象是同一个类型，但这是两把不同的对象锁，多个线程同时执行不阻塞。

<img src="https://gitee.com/Jackpotsss/pic_go/raw/master/img/image-20200227171645259.png" alt="image-20200227171645259" style="zoom: 80%;" />

下面是Java线程的栈帧里保存的锁信息：

```java
//调试信息：Class对象锁
owns: Class<T> (com.beecode.bms.http.core.service.A) (id=16)
//调试信息：实例对象锁
owns: A  (id=25)	
```



**升级**

​	在Java的多线程并发编程中，synchronized 一直是元老级角色，很多人都称呼它为重量级锁，这是因为在JDK1.5及之前的版本中，synchronized 是一个“纯粹”的互斥锁，它严格的按照只有一个线程可以持有锁这种规则运行，只要存在锁竞争，它就会立马阻塞其他线程，在锁竞争不激烈的场景下，造成很多不必要的线程上下文切换开销。

​	在JDK1.6 中，对 synchronized 进行了优化，加锁方式发生了变化，性能有了很大的提升。JDK6 引入了偏向锁和轻量级锁，自此，synchronized 具有 无锁，偏向锁，轻量级锁和互斥锁 四种状态了。



## 3.2 偏向锁

​	当不存在锁竞争的情况下，即始终只有一个线程在操作临界区，synchronized 会在对象头设置标记，并且记录当前线程ID（使用原子的CAS操作），偏向于该线程，下次该线程再进入临界区，synchronized 会对比当前线程和对象头中标记的线程ID是否是同一个，如果是同一个线程，那么直接进入临界区。

## 3.3 轻量级锁

​	在锁竞争不激烈的情况下，即只有两个线程同时操作临界区，而且临界区代码的执行时间非常短，synchronized 会采用短时间轮询的方式获取同步锁，即自旋锁。线程B进入同步代码块之前对比一下对象头的偏向锁标记与当前线程ID是否相同，如果不同，线程B会自旋式的获取锁，如果线程A短时间内可以释放锁，那么自旋的线程B就有机会获取同步锁，而不用阻塞自己，造成线程的上下文切换。



## 3.4 互斥锁 

​	在锁竞争激烈的情况下，即持有锁的线程执行临界区代码的时间比较长，或者是有更多的线程同时获取同一把锁，这时 synchronized 便会放弃自旋式获取锁，短时间内的自旋比阻塞线程效率要高一些，但自旋时间过长，就会占用过多的CPU资源，效率便会下降。这种情况下，轻量级锁会升级为重量级锁，即阻塞要进入临界区的其他所有线程。



注：**锁只能升级不能降级！**   

​	分析锁竞争是否激烈对并发编程是非常有必要的。总览Java内置锁的整体设计，synchronized 其实将**非阻塞算法与阻塞算法相结合使用**，在锁竞争不激烈的情况下，使用非阻塞算法效率更高，在锁竞争激烈的情况下，使用阻塞算法则更适合一些。



## 3.5 对象与锁

​	前面说到Java的内置锁也叫对象监视器锁，Java中的每一个对象都可以作为锁。为什么这么说呢，因为内置锁的标记被设计在**对象头**里。这里简单说一下JVM中Java对象的组成元素：

- 对象头
- 实例数据
- 填充部分



​	其中实例数据是类的实例对象的数据；JVM规范中规定Java对象的内存大小必须是8的整数倍，所以设计了填充部分来补齐这部分数据；对象头专门存储一些特殊的信息，如hashCode值和锁信息等。

​	可以看到，Java对象和内置锁设计的基本思想也是将共享数据（实例数据）和信号量（对象头中的锁标记）关联起来，然后通过相应的原子操作维护信号量的值，判断信号量的值然后阻塞、唤醒线程，从而实现互斥锁。



# 4. volatile 关键字

​	volatile 变量是Java语言提供的一种稍弱的同步机制，用来确保将共享变量的更新操作通知到其他线程。

​	volatile 的英文释义是不稳定的、易挥发的，在计算科学中的意思就是**易失性**的，volatile 关键字修饰的变量在线程私有内存中具有易失性。

volatile 关键字的内存语义如下：

- 保证对单个volatile 变量读的可见性，线程总是能读到最新的变量值（即任意线程对该共享变量的最后写入）。
- 保证对单个volatile 变量写的可见性，线程对volatile 变量的写会立即被刷新到主内存中，并且使其他线程所持有该变量的缓存值失效，其他线程必须重新读取最新的volatile 变量值。



当且仅当满足以下条件时，才应该使用volatile 变量：

1. 多写多读，前提是对变量的写入操作不能依赖原值！
2. 一写多读，确保只有单个线程更新变量的值；
3. 访问变量不需要加锁；



注：

- 加锁机制可以保证原子性和可见性，但 volatile 只能保证可见性，不能保证原子性。

- volatile 变量对内存可见性的影响比volatile 变量本身的意义更为重要。




特别注意：

​	volatile 变量在一写多读的场景下，可以保证线程安全；对于多写多读，并且写操作依赖原值，就无法保证线程安全了（比如对volatile 变量进行i++操作）。



# 5. CAS

比较并交换（Conmpare And Swap）是用于实现多线程同步的**原子指令**。

## 5.1 比较并交换

Java中CAS是由 sun.misc.Unsafe 类的 native 方法提供的，它提供了int、long和对象引用三种数据类型的CAS原子更新操作：

```java
public final class Unsafe {

	public final native boolean compareAndSwapInt
        (Object var1, long var2, int var4, int var5);

    public final native boolean compareAndSwapLong
        (Object var1, long var2, long var4, long var6);  
    
    public final native boolean compareAndSwapObject
        (Object var1, long var2, Object var4, Object var5);
}
```

​	CAS的基本原理很简单，输入期望值expect 和要更新的值update，如果目标变量的真实值和期望值相同，那么就原子的更新目标变量的值为要更新的值，设置成功返回true，失败返回false；如果真实值和期望值不一样，直接更新失败。

​	原子CAS是一种非阻塞的更新方式，执行一次不是成功就是失败，不会阻塞任何线程。

注：

​	CAS 原子更新变量不会阻塞线程，是因为CAS在硬件层面（CPU、缓存）提供了原子操作的保证，线程说到底也只是操作系统上的操作对象，阻塞线程可以认为是在软件层面进行干扰，所以CAS操作也是有代价的，原子操作和阻塞线程都是为了保证数据安全而做的干扰手段，只不过干扰的层面不一样。



## 5.2 带来的问题

CAS可以有效提高并发效率，但同时也带来了一些问题：

- ABA 问题；
- 循环时间长开销大；
- 只能保证一个共享变量的原子操作；



**ABA 问题** 

​	CAS的思想是如果目标变量的值没有变化，那么就执行更新，否则不更新。CAS的具体实现是通过对比实际值和期望值来确定目标变量的值是否发生变化，这种做法存在一个问题，如果目标变量的值从A变到B，然后再从B变成A，那么按照CAS的思想来说，这种情况CAS是不应该更新成功的，但按照CAS的具体实现执行的话，确实可以更新成功，这就是ABA问题。

解决方案：

​	使用版本号可以解决ABA问题。每次更新目标变量的值，顺带着更新相应的版本号（版本号递增更新），这样就可以解决多线程下的ABA问题了。

**循环时间长开销大** 

​	CAS 更新失败后，我们可以选择继续更新，也可以选择不更新。在构建线程安全的非阻塞算法中，通常会循环CAS 更新值，直到成功。在线程竞争不激烈的情况下，非阻塞的CAS 更新确实可以提高并发效率；但在线程竞争激烈的情况下，CAS循环的次数越来越多，消耗的CPU计算资源也越来越多，开销也就越大。

**只能保证一个共享变量的原子操作** 

​	针对这点，可以将要更新的多个变量封装为一个对象，然后使用 atomic 包下的AtomicReference 原子更新对象的引用，即可解决这一问题。



## 5.3 volatile + CAS 

<img src="https://gitee.com/Jackpotsss/pic_go/raw/master/img/image-20200303123745073.png" alt="image-20200303123745073" style="zoom: 33%;" />

​	concurrent 包的实现底层均是由 volatile 和 CAS 支撑起来的， volatile 是一个关键字，在Java层面无法查看其具体实现细节，而 CAS 也是 Unsafe 类提供的 native 方法实现的，在Java层面也无法查看具体实现细节，所以对于这两部分内容，在Java层面，我们只能学习他的原理和思想。



# 6. Java中的锁

在JDK1.5 之前，Java中的锁都是靠 `synchronized` 关键字来实现的。JDK1.5 引入了 并发包 `java.util.concurrent` 。

## 6.1 Lock 接口

​	在 `Lock` 接口出现之前，Java程序是靠 synchronized 关键字实现锁功能的。而在Java 5 之后，并发包中新增了 Lock接口以及相关实现类来实现锁功能。

### 用法

不同于 synchronized 内置锁**隐式**的获取锁和释放锁的方式，Lock 接口需要**显式**的获取和释放锁。

```java
ReentrantLock lock = new ReentrantLock();
lock.lock();
try {
	//TODO
} finally {
	lock.unlock();
}
```

特别注意：

2. 不要将获取锁的过程写在try块中！因为如果获取锁时发生了异常，异常抛出的同时，也会导致锁无故释放。
2. 一定要在finally 块中释放锁！目的是保证在获取到锁之后，最终能够被释放（即便是临界区内的代码抛出异常）。

### 特性

**Lock 接口提供了 synchronized 内置锁不具备的主要特性：**  

| 特性               | 描述                                                         |
| ------------------ | ------------------------------------------------------------ |
| 尝试非阻塞的获取锁 | 当前线程尝试获取锁，无论是否成功都会立即返回；               |
| 超时获取锁         | 当前线程在指定的截止时间之前获取锁，如果到了超时时间仍然无法获取锁，则返回； |
| 能够被中断的获取锁 | 获取到的锁能够响应中断，当获取到锁的线程被中断时，中断异常将会抛出，同时锁会被释放。 |
| 非块状的加锁       |                                                              |

### 接口

```java
public interface Lock {
    
    //阻塞的获取锁,当前线程获取锁之后,从该方法返回
    void lock();	
   
    //非阻塞的尝试获取锁,该方法调用后立即返回,成功返回true,失败返回false
	boolean tryLock(); 
    
    //可响应中断的获取锁
    void lockInterruptibly() throws InterruptedException;
    
    /**
 	* 可响应中断的超时获取锁,调用该方法在以下3种情况会返回:
	*
 	* 1.当前线程在超时时间内获取到锁,返回true;
 	* 2.当前线程在超时时间内没有获取到锁,返回false;
 	* 3.当前线程在超时时间内被中断;
 	*/
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
    
    //释放锁
    void unlock();	
    
    /**
	 * 获取等待通知组件,该组件和当前的锁绑定,当前线程只有获取了锁,才能调用该组件中的await()方法,
	 * 调用后,当前线程进入等待状态并释放锁;
 	 */
	Condition newCondition();
}
```

​	在使用内置锁时，阻塞在同步块的线程会一直阻塞下去，直到获取到锁，而且无法响应中断，若使用不当可能会产生死锁，死锁是一个严重的问题，恢复程序的唯一方法是重新启动程序，而防止死锁的唯一方法就是在设计程序时避免出现不一致的锁顺序。

​	可定时和可轮询的获取锁，为解决死锁提供了另一种选择。定时的获取锁，如果指定时间内没有获取成功，那么就使该操作平缓的失败；使用 tryLock 一次获取两把锁，如果不能同时获得两把锁，那么就回退并重新尝试。

​	Lock 接口是面向锁的**使用者**设计的接口，而下面的队列同步器AQS 是面向锁的**实现者**所设计的接口。



## 6.2 队列同步器 AQS（核心）

​	队列同步器 （AbstractQueuedSynchronizer）是实现**锁**和任何**同步组件**的关键，只有掌握了同步器的工作原理，才能更加深入的理解并发包中的其他并发组件。

### 接口及实例

同步器要重写的方法：

| 方法                                        | 描述                 |
| ------------------------------------------- | -------------------- |
| protected boolean tryAcquire(int arg)       | 独占式获取同步状态   |
| protected boolean tryRelease(int arg)       | 独占式释放同步状态   |
| protected int tryAcquireShared(int arg)     | 共享式获取同步状态   |
| protected boolean tryReleaseShared(int arg) | 共享式释放同步状态   |
| protected boolean isHeldExclusively()       | 是否被当前线程所独占 |

​	上面这5个方法的默认实现是直接抛出异常 `throw new UnsupportedOperationException()` ，等待被重写；tryAcquire 的实现需要读取当前同步状态的值，并判断是否符合预期，然后再进行CAS设置同步状态。

​	实现自定义同步组件时，需要调用同步器提供的模板方法：

| 方法                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| public final void acquire(int arg)                           | 独占式获取同步状态；<br />如果当前线程获取同步状态成功，则返回，<br />否则将会进入同步队列等待，该方法会调用重写的tryAcquire 方法； |
| public final void acquireInterruptibly(int arg)         throws InterruptedException | 可响应中断的独占式获取同步状态；如果未获取同步状态而进入同步队里，此时中断线程，会立即返回并抛出中断异常 |
| public final boolean tryAcquireNanos(int arg, long nanosTimeout)         throws InterruptedException | 在 acquireInterruptibly 基础上增加了超时限制，在超时时间限制内获取锁返回true，否则返回false； |
| public final void acquireShared(int arg)                     | 共享式获取同步状态；与独占式获取同步状态主要的不同是同一时刻可以有多个线程获取到同步状态； |
| public final void acquireSharedInterruptibly(int arg)         throws InterruptedException | 可响应中断的共享式获取同步状态；                             |
| public final boolean tryAcquireSharedNanos(int arg, long nanosTimeout)         throws InterruptedException | 在 acquireSharedInterruptibly基础上增加了超时限制            |
| public final boolean release(int arg)                        | 独占式释放同步状态；<br />释放同步状态之后，将同步队列中的第一个节点包含的线程唤醒； |
| public final void acquireShared(int arg)                     | 共享式释放同步状态；                                         |
| public final Collection<Thread> getQueuedThreads()           | 获取等待在同步队列上的线程集合；                             |

​	上面的模板方法全部是 `public final` 修饰的方法，可分为3类：独占式获取和释放同步状态，共享式获取和释放同步状态以及获取同步队列中的线程集合。

**实例**

`AbstractQueuedSynchronizer` 是抽象类，子类需要继承队列同步器，

```java
/**
 * 自定义互斥锁
 */
public class Mutex implements Lock{
	//基于AQS构建同步器
	private static class Sync extends AbstractQueuedSynchronizer {
		
		protected boolean isHeldExclusively() {
			return getState() == 1;
		}

		//获取锁
		@Override
		public boolean tryAcquire(int acquires) {
			
			if (compareAndSetState(0, 1)) {
				setExclusiveOwnerThread(Thread.currentThread());
				return true;
			}
			return false;
		}
		//释放锁
		@Override
		protected boolean tryRelease(int releases) {
			if (getState() == 0) throw new IllegalMonitorStateException();
			setExclusiveOwnerThread(null);
			setState(0);
			return true;
		}

		Condition newCondition() {
			return new ConditionObject();
		}
	}

	//将需要的操作代理到Sync上即可
	private final Sync sync = new Sync();

	public void lock() {sync.acquire(1);}

	public boolean tryLock() {return sync.tryAcquire(1);}

	public void unlock() {sync.release(1);}

	public Condition newCondition() {return sync.newCondition();}

	public boolean isLocked() {return sync.isHeldExclusively();}

	public boolean hasQueuedThreads() {return sync.hasQueuedThreads();}

	public void lockInterruptibly() throws InterruptedException {
		sync.acquireInterruptibly(1);
	}

	public boolean tryLock(long timeout, TimeUnit unit) throws InterruptedException {
		return sync.tryAcquireNanos(1, unit.toNanos(timeout));
	}
}
```



### 同步状态

同步状态是一个 volatile 修饰的全局变量 （信号量）。

```java
public abstract class AbstractQueuedSynchronizer{

    private transient volatile Node head;
    private transient volatile Node tail;
	//同步状态
    private volatile int state;

    protected final int getState() {
        return state;
    }
    protected final void setState(int newState) {
        state = newState;
    }
}
```

### CAS 操作

CAS 原子操作

```java
/** 提供CAS的Unsafe类 */
private static final Unsafe unsafe = Unsafe.getUnsafe();

/** CAS设置同步状态 */
protected final boolean compareAndSetState(int expect, int update) {
    return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
}

/** CAS设置队列头部结点*/
private final boolean compareAndSetHead(Node update) {
   return unsafe.compareAndSwapObject(this, headOffset, null, update);
}
/** CAS设置队列尾部结点*/
private final boolean compareAndSetTail(Node expect, Node update) {
   return unsafe.compareAndSwapObject(this, tailOffset, expect, update);
}
/** CAS设置等待状态*/
private static final boolean compareAndSetWaitStatus(Node node,int expect, int update){
   return unsafe.compareAndSwapInt(node, waitStatusOffset,expect, update);
}
/** CAS设置下一个节点*/
private static final boolean compareAndSetNext(Node node,Node expect,Node update) {
   return unsafe.compareAndSwapObject(node, nextOffset, expect, update);
}
```



### 同步队列

<img src="https://gitee.com/Jackpotsss/pic_go/raw/master/img/image-20200307235319912.png" alt="image-20200307235319912" style="zoom: 50%;" />

队列（双向链表）

```java

private volatile Node head;
private volatile Node tail;

static final class Node {
    
    volatile int waitStatus;
    volatile Node prev;
    volatile Node next;
    volatile Thread thread;
}
```

同步器持有同步队列的头部和尾部的引用，同步节点保存了**线程引用，前后节点引用和等待状态**。

**节点**

​	队列的节点保存了线程引用、前后节点引用和等待状态，同步队列的同步节点和等待队列的条件节点都使用该内部类构建。节点有2种模式和5种状态。

```java
static final class Node {
    static final Node SHARED = new Node();		//共享模式
    static final Node EXCLUSIVE = null;			//独占模式
}
```

尝试独占式的获取同步状态，成功即返回；如果失败就构建一个独占模式的新节点加入同步队列，然后中断当前线程。

```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) && acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```



```java
private void setHead(Node node) {
    head = node;
    node.thread = null;
    node.prev = null;
}
```

​	这里重点说一下同步队列的头，在没有多个线程竞争同一把独占锁的时候，同步队列为空；当有其他线程获取锁失败，则把当前线程构建成一个新的节点并加到同步队列的队尾，这里有个细节就是同步队列必须进行一次初始化操作，将一个不包含线程的新节点当做队头，然后再将刚刚构建的包含线程的新节点加入到队尾，具体细节在 `enq()` 入队方法中；这么做的目的是为了符合接下来的游戏规则：同步队列中的每个非头部节点都会一直自旋判断当前节点状态 `node.prev == head && tryAcquire(arg)` ，当前节点的前置节点是否为头部，如果是就尝试获取锁，具体实现细节在 `acquireQueued()` 方法中，所以同步队列的头部默认拥有同步状态。

​	同步队列的头也不会保存线程引用，因为既然拥有同步状态说明当前线程就是头部节点对应的线程。



<img src="https://gitee.com/Jackpotsss/pic_go/raw/master/img/image-20200328195804261.png" alt="image-20200328195804261" style="zoom: 33%;" />





## 6.3 重入锁和非公平锁

ReentrantLock 提供了重入锁和公平/非公平锁的实现，默认是非公平锁。

### ReentrantLock 重入锁

​	重入锁，就是线程在获取到锁的情况下，可以重复获取锁；上面自定义的 Mutex 互斥锁就不支持重进入。

**获取可重入锁**

```java
        final boolean nonfairTryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                if (compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0) throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
```

​	第一个if判断是在无锁状态下获取锁，第二个if 判断就是为了重入锁而设计的逻辑了，判断当前线程是否是获取到锁的线程，如果是，那么在此获取锁成功，并且将同步变量+1，每次重进入锁都会+1；

**释放可重入锁**

```java
        protected final boolean tryRelease(int releases) {
            int c = getState() - releases;
            if (Thread.currentThread() != getExclusiveOwnerThread())
                throw new IllegalMonitorStateException();
            boolean free = false;
            if (c == 0) {
                free = true;
                setExclusiveOwnerThread(null);
            }
            setState(c);
            return free;
        }
```

​	每次释放锁，也都会修改同步变量 - 1，**如果该锁获取了N次，则需要释放N次**，并且前（N-1）次 tryRelease 方法返回false；直到同步变量的值为0，才释放成功；

注：	synchronized 内置锁隐式的支持重进入。



### 非公平性机制

​	公平性与否是针对获取锁而言的，公平锁意味着锁的获取是严格按照请求顺序来完成的，即先请求的锁的线程最先得到锁，也就是FIFO。

​	ReentrantLock 中提供了两个同步器： 非公平锁同步器 `NonfairSync` 和公平锁同步器 `FairSync`  ；二者获取同步锁的方法分别是 nonfairTryAcquire（）和 tryAcquire（），两个方法唯一的不同就是公平锁多了一次校验 `hasQueuedPredecessors()` ，非公平锁是直接尝试获取同步状态，同步队列中的第二个节点和此时此刻同步器外的线程有可能同时获取同步状态，这样的行为在请求顺序上来说是不公平的。

​	hasQueuedPredecessors 方法用来判断同步队里中是否存在比当前线程请求时间更早的线程，如果当前线程之前有排队等候的线程，则返回true；如果队列为空或当前线程位于队列开头，则返回false；

```java
		protected final boolean tryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                if (!hasQueuedPredecessors() &&
                    compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0)
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
```

​	在非公平情况下，线程A和线程B交替获取锁，线程A最先请求，获取到锁，线程B处于阻塞状态；线程A释放锁之后唤醒B，同时A又去争夺锁，那么有很大的可能是线程A又获取到了锁，线程B继续阻塞，这就是非公平的锁。

​	公平锁保证了锁的获取是按照FIFO原则，而代价是进行大量的线程切换；非公平锁虽然可能造成线程“饥渴”，但线程切换次数少，提高了吞吐量。

​	非公平锁的效率比公平锁要高，ReentrantLock 默认是非公平锁，可以通过构造器指定公平性。

### synchronized 和 ReentrantLock 

​	synchronized 是JVM实现的内置锁，ReentrantLock 是Java层面实现的显式锁。在很多场景下，synchronized 和 ReentrantLock 都是可以互换的，二者都是互斥的可重入锁。在JDK1.5之前，Java的互斥锁只能使用内置的 synchronized 实现，JDK1.5版本引入了并发包类库，ReentrantLock 由此诞生；根据官方提供的数据，1.5版本ReentrantLock 的并发性能比synchronized 高出很多，优势非常明显，但在JDK1.6版本，JVM开发团队对synchronized 进行了优化，自此二者的性能差距已经非常小了，所以在今后的版本中，性能已经不是用于比较二者所要考虑的主要因素了。

​	那么synchronized 和 ReentrantLock 到底选择哪个更好一些呢？答案其实是得视情况而定，synchronized 和ReentrantLock 各有各的优势与劣势。

​	synchronized 语法结构紧凑（块状结构），并且最早出现在Java应用的并发环境中，被更多人所接受和使用，synchronized 比ReentrantLock 使用起来更安全一些（自动获取锁和释放锁），在使用显式锁时如果忘记在finally块中调用 unlock 语句，那么这将是一个隐患，程序运行初期可能没有问题，但后期就是一个定时炸弹，出现问题也不容易排查。

​	在内置锁无法满足需求的场景下，应该使用ReentrantLock 提供的高级功能，如**可定时、可中断、可轮询的获取锁，公平锁以及非块结构的锁**；否则，应该优先使用 synchronized。



## 6.4 读写锁

​	之前提到的锁都是**排他锁**，（synchronized 和 ReentrantLock ），这些锁同一时刻只允许一个线程操作共享变量，而读写锁允许同一时刻被多个线程访问。

​	读锁也叫共享锁，线程A获取读锁时，其他线程均可以获取读锁，但是不可以获取写锁；

​	写锁也叫独占锁，线程A获取写锁时，其他线程既不可以获取读锁，也不可以获取写锁；

<img src="https://gitee.com/Jackpotsss/pic_go/raw/master/img/image-20200308182146454.png" alt="image-20200308182146454" style="zoom: 33%;" />



### ReadWriteLock 接口

ReadWriteLock 维护了一对关联的锁，读锁和写锁。读锁用于只读操作，写锁用于写入。

```java
public interface ReadWriteLock {
    /** 读锁 */
    Lock readLock();
    /** 写锁 */
    Lock writeLock();
}
```



### ReentrantReadWriteLock 的使用

ReentrantReadWriteLock 是 ReadWriteLock接口的主要实现，下面是个栗子：

```java
/**读写锁的使用*/
public class Cache {
	
	private static final HashMap<String, Object> cache = new HashMap<String, Object>();
	
	private static ReentrantReadWriteLock lock = new ReentrantReadWriteLock();
	private static ReadLock readLock = lock.readLock();
	private static WriteLock writeLock = lock.writeLock();
	
	public static final Object getCache(String key) {
		Object value = null;
		readLock.lock();
		try {
			value = cache.get(key);
		} finally {
			readLock.unlock();
		}
		return value;
	}
	
	public static final void putCache(String key,Object value) {
		writeLock.lock();
		try {
			cache.put(key,value);
		} finally {
			writeLock.unlock();
		}
	}
}
```

​	HashMap 本不是线程安全的，在读操作上加上读锁，在写操作上加上写锁，在多读少写的场景下，既保证了线程安全，也提高了并发效率。



### 实现原理

​	ReentrantReadWriteLock 的实现主要包括：读写状态的设计、写锁的获取与释放、读锁的获取与释放以及锁降级。

#### 读写状态的设计

​	Doug Lea 使用**位运算**在一个同步变量上维护两个状态，即读状态和写状态。int 类型的变量共32位，高16位维护读状态，低16位维护写状态。

```java
public class ReentrantReadWriteLock implements ReadWriteLock{
    
    //基于AQS实现的同步器
    abstract static class Sync extends AbstractQueuedSynchronizer {
        
     	static final int SHARED_SHIFT   = 16;
		static final int SHARED_UNIT    = (1 << SHARED_SHIFT);		//1 0000 0000 0000 0000 
		static final int MAX_COUNT      = (1 << SHARED_SHIFT) - 1;	//1111 1111 1111 1111 (16个1)
		static final int EXCLUSIVE_MASK = (1 << SHARED_SHIFT) - 1;

		/** 获取读锁的数量 */
		static int sharedCount(int c)    { return c >>> SHARED_SHIFT; }
		/** 获取写锁的数量(支持重进入)  */
		static int exclusiveCount(int c) { return c & EXCLUSIVE_MASK; }   
    }
}
```



<img src="https://gitee.com/Jackpotsss/pic_go/raw/master/img/image-20200724090808242.png" alt="image-20200724090808242" style="zoom: 33%;" />



#### 写锁的获取与释放

写锁是一个可重进入的排他锁，同一时刻只能有一个线程可以获取到写锁，获取到写锁后，其他线程均不可以再获取写锁或读锁。

```java
//尝试获取独占锁的源码如下
protected final boolean tryAcquire(int acquires) {
    Thread current = Thread.currentThread();
    int c = getState();
    int w = exclusiveCount(c);
    if (c != 0) {
        //情况1:其他线程已经获取到读锁,则获取独占锁失败; if c != 0 and w == 0 then shared count != 0
        //情况2:其他线程已经获取到写锁,则获取独占锁失败;
        if (w == 0 || current != getExclusiveOwnerThread())
            return false;
        if (w + exclusiveCount(acquires) > MAX_COUNT)	//写状态不能大于65535
            throw new Error("Maximum lock count exceeded");
        //情况3:当前线程之前获取过写锁，独占锁重进入:写状态+1
        setState(c + acquires);
        return true;
    }
    //情况4:同步状态为0,直接获取独占锁
    if (writerShouldBlock() || !compareAndSetState(c, c + acquires))
        return false;
    setExclusiveOwnerThread(current);//设置独占锁拥有者为当线程
    return true;  
}
```

​	根据源码的条件判断可以看到获取独占锁的四种情况，除了以上几种情况，还有一种情况需要注意，如果当前线程已经持有读锁，那么该线程将无法继续持有写锁的，也就是**线程不能在持有读锁的情况下再获取写锁，不支持锁升级**；但反过来却可以，**线程可以在持有写锁的情况情况下继续获取读锁，支持锁降级**。

​	为什么不可以在持有读锁的情况下再获取写锁呢？按理来说即便这么设计也不会有线程不安全的问题，仔细想想，这与读状态和写状态的性质有关，在读写锁中，读状态和写状态都可以大于1，写状态大于1一定是某个线程重复获取写锁导致的，而读状态大于1，却分不清到底有几个线程持有读锁，因为读锁即支持单个线程重复进入，也支持多个线程同时获取，所以单单靠判断读状态和写状态，以及当前持有独占锁的线程引用是无法实现的。如果同步状态不为0，写状态为0，那么读状态肯定不为0，因为无法判断当前获取读锁的是哪个线程，所以不能加写锁（理论上如果获取到读锁的只是当前线程，那么当前线程继续获取写锁，不会有问题）。



#### 读锁的获取与释放

​	读锁是一个可重进入的共享锁，多个线程可以同时获取。在写锁状态为0的情况下，读锁总是可以获取成功；当其他线程获取写锁后，获取读锁会一直阻塞，直到写锁释放。

```java
//共享式的获取锁
protected final int tryAcquireShared(int unused) {
    
    Thread current = Thread.currentThread();
    int c = getState();
    //情况1:其他线程已经获取独占锁,获取共享锁失败
    if (exclusiveCount(c) != 0 && getExclusiveOwnerThread() != current)
         return -1;
    int r = sharedCount(c);
    //情况2:写锁状态为0 或者当前线程获取到写锁,则可以获取读锁,读状态+1
    if (!readerShouldBlock() && r < MAX_COUNT && compareAndSetState(c, c + SHARED_UNIT)) {
         return 1;
    }
    return fullTryAcquireShared(current);  
}
```



#### 锁降级

​	读写锁的降级是指获得写锁后，再获得读锁，然后释放写锁，从而完成从写锁到读锁的锁降级。注意，线程获取写锁后释放写锁，然后再获取读锁，这并不是锁降级，必须是在持有写锁的前提下再获得读锁。

​	读写锁不支持锁升级，只能降级。也就是说一个线程在持有读锁的情况下不能继续获取写锁。

```java
	public static void lockDown() {
		//1.先获取写锁
		writeLock.lock();
		try {
			doWrite();//修改操作
			//2.持有写锁的情况下,获取读锁;
			readLock.lock();
		} finally {
            //3.获取读锁后释放写锁,完成锁降级;
			writeLock.unlock();
		}
		try {
			value = cache.get(key);
			doWork();//后续操作
		} finally {
            //4.最后释放读锁
			readLock.unlock();
		}
	}
```

​	如果不是在持有写锁的情况下获取读锁，而是释放写锁后紧接着获取读锁，那么在这个间隙时间里，其他线程是有可能获取读锁然后修改数据的，此时当前线程后续操作的数据未必是最新的，也就是对这部分数据的修改感知不到，从而造成数据不安全。

​	如果是在持有写锁的情况下获取读锁，当前线程在读锁中进行后续操作，期间其他线程时无法修改这部分数据的，所以是线程安全的。



### 读写锁与互斥锁

​	尽管读写锁是为了提高并发性能而设计出来的，但读写锁并不是在所有的场景下都比互斥锁的效率高。读写锁的设计比互斥锁更复杂一些，所以只有在适当的场景下使用读写锁，才能有效的提高并发性能。

​	如果数据不经常修改，而是经常查询，即读操作比写操作的频率大很多，那么使用读写锁是不错的选择。但情况如果是写操作比读操作频率还高，那么读写锁的效率要低于互斥锁。

​	最终，只有性能分析和充分测试才能确定读写锁是否适用于你的程序。如果读写锁并没有提高并发性能，由于 ReadWriteLock 使用Lock 实现读写锁部分的设计，所以可以很容易的把读写锁换成互斥锁。



## 6.5 LockSupport 工具

JUC 包中Lock接口对于线程阻塞和唤醒的实现均使用了LockSupport的 park() 和 unpark()方法。

LockSupport 是 Java **阻塞和唤醒**线程的基础工具，它可以在任何场合阻塞线程，也可以指定线程进行唤醒。

LockSupport 提供了 `park` 开头的阻塞线程方法和 `unpark(Thread thread)` 开头的唤醒线程方法。



```java
public class LockSupport {
    private LockSupport() {} 
    /** 阻塞当前线程;*/
    public static void park(Object blocker) {
    	Thread t = Thread.currentThread();
        setBlocker(t, blocker);
        UNSAFE.park(false, 0L);
        setBlocker(t, null);
    }
    /** 超时阻塞;最长不超过 nanos 纳秒;*/
    public static void parkNanos(Object blocker, long nanos) {
        if (nanos > 0) {
            Thread t = Thread.currentThread();
            setBlocker(t, blocker);
            UNSAFE.park(false, nanos);
            setBlocker(t, null);
        }
    }
    /** 超时阻塞;直到 deadline 时间;*/
    public static void parkUntil(Object blocker, long deadline) {
        Thread t = Thread.currentThread();
        setBlocker(t, blocker);
        UNSAFE.park(true, deadline);
        setBlocker(t, null);
    }   
    /** 唤醒线程;*/
    public static void unpark(Thread thread) {
        if (thread != null)
            UNSAFE.unpark(thread);
    } 
    
    public static Object getBlocker(Thread t) {
        if (t == null) throw new NullPointerException();
        return UNSAFE.getObjectVolatile(t, parkBlockerOffset);
    }
}
```

​	可以看到 LockSupport 工具定义了一组公共的静态方法，提供了最基本的阻塞和唤醒线程的功能，LockSupport 也成为了构建同步组件的基础工具。

​	值得注意的是，**blocker** 代表阻塞线程的对象，用于监控和调试使用，是1.6版本新增的方法，可以通过 `getBlocker(Thread t)` 获取阻塞线程的对象；park 和 unpark 底层调用的是 **UnSafe** 类的 native 方法。

注：

​	有意思的是park 和 unpark 的调用可以没有先后顺序，它引入一个许可证【permit】的概念（其实就是一个同步变量），调用 unpark 提供给线程一个许可，调用park 消耗一个许可，如果线程没有许可证，则阻塞当前线程；如果有许可证，则会立即从该方法返回。

​	许可证可以提前提供，也就是先调用unpark 再调用 park ，并不会阻塞线程；unpark 多次连续调用和一次调用的效果是一样的，只会增加一个许可证（permit），所以先唤醒两次线程再阻塞两次线程，会阻塞线程。

​	**Java 中有两套阻塞和唤醒线程的机制，一个是Object的 wait 和 notify，另一个是LockSupport 的park 和 unpark 。这两套机制是独立运行、没有关联的。**也就是说你不可以使用 Object.wait() 阻塞线程，而又使用 LockSupport.unpark() 唤醒线程。

​	另外 Object的 wait 和 notify，与Condition 的 await 和 signal 可以达到相同的效果，他们都会在持有锁的情况下放弃当前占有的资源（也就是释放锁）进入等待状态，然后唤醒线程再次获取锁；Condition 的 await 和 signal 阻塞和唤醒的功能是 LockSupport 的park 和 unpark 提供的，park 和 unpark 仅仅提供线程阻塞和唤醒的功能，释放和获取锁资源是Condition 接口的实现类实现的，也就是说**获取和释放锁资源与线程的阻塞与唤醒是两套工作，理论上，它俩没有必然的关系；**只是获取锁和阻塞线程在并发编程中有逻辑关系，才将这两套工作放在一起。在并发包中，将这两套工作分开实现了，获取和释放锁资源的实现是队列同步器AQS提供的，阻塞和唤醒线程是LockSupport调用 native 方法提供的；从编程语言的层面说的再直白点，获取和释放锁资源的逻辑在Java中实现，阻塞和唤醒线程的逻辑在C++中实现；

​	我看到过这么一个博客：LockSupport.park 会不会释放锁资源？答案是不会。看过源码的设计就知道了，LockSupport.park 只是单纯的阻塞线程，释放锁的工作在它之前，由AQS实现。在这方面 Thread.sleep() 和LockSupport.park 是一样的，他们不关心锁资源是否获取和释放，他们只管阻塞线程就行了。



## 6.6 Condition

​	前面说了synchronized + Object.wait () / notify()方法，提供了等待/通知机制，同样，Lock + Condition 也实现了等待/通知机制。

​	Condition 定义了等待/通知两种类型的方法，**当线程调用这些方法时，必须提前获取Condition 对象关联的锁。**Condition 对象由Lock对象创建，Condition 是依赖 Lock对象的。

### API

```java

public interface Condition {
    
   /** 当前线程进入等待状态,直到被通知(signal)或中断*/
   void await() throws InterruptedException; 
    
   /** 当前线程进入等待状态,直到被通知、中断或超时*/
   boolean await(long time, TimeUnit unit) throws InterruptedException;
   
   /** 当前线程进入等待状态,直到被通知、中断或到某个时间 */
   boolean awaitUntil(Date deadline) throws InterruptedException;
    
   /** 当前线程进入等待状态,直到被通知(不响应中断)*/
   void awaitUninterruptibly();
  
   /** 唤醒一个等待在Condition上的线程,该线程从等待方法返回前必须获得与Condition相关联的锁*/
   void signal();
    
   /** 唤醒所有等待在Condition上的线程,该线程从等待方法返回前必须获得与Condition相关联的锁*/
   void signalAll();
}
```

### 使用

​	使用Lock + Condition 实现一个有界缓冲区（阻塞队列的原型），在空缓冲区上执行获取操作将阻塞线程，直到缓冲区不为空；在满缓冲区上执行添加操作将阻塞线程，直到缓冲区不为满；我们可以使用两个Condition实例来实现。

```java
public class BoundedBuffer {

	final Lock lock = new ReentrantLock();
	final Condition notFull = lock.newCondition();
	final Condition notEmpty = lock.newCondition();

	final Object[] items = new Object[100];
	int putptr, takeptr, count;

	public void put(Object x) throws InterruptedException {
		lock.lock();
		try {
			while (count == items.length)
				notFull.await();
			items[putptr] = x;
			if (++putptr == items.length)
				putptr = 0;
			++count;
			notEmpty.signal();
		} finally {
			lock.unlock();
		}
	}

	public Object take() throws InterruptedException {
		lock.lock();
		try {
			while (count == 0)
				notEmpty.await();
			Object x = items[takeptr];
			if (++takeptr == items.length)
				takeptr = 0;
			--count;
			notFull.signal();
			return x;
		} finally {
			lock.unlock();
		}
	}
}
```



​	notEmpty 条件队列保存着等待不为空的线程引用，也就是在缓冲区为空的情况下，调用take() 方法获取元素时，应该使用Condition的 await() 方法将当前线程加入到条件队列，并阻塞；当其他线程往缓冲区添加元素，使缓冲区不为空的条件满足时，也就是 put(Object x) 方法添加元素成功后，应该调用 Condition的 signal() 方法 唤醒一个等待在notEmpty 条件队列中的一个线程。

​	notFull 条件队列保存着等待不为满的线程引用，也就是在缓冲区为满的情况下，调用 put(Object x) 方法添加元素时，应该使用Condition的 await() 方法将当前线程加入到条件队列，并阻塞；当其他线程在缓冲区获取元素，使缓冲区不为满的条件满足时，也就是 take() 方法获取元素成功后，应该调用 Condition的 signal() 方法 唤醒一个等待在notFull 条件队列中的一个线程。



### 实现

在持有锁的前提下，调用condition.await() 

1. 构建节点加入条件队列；
2. 释放锁资源；
3. 阻塞当前线程；



**ConditionObject** 

ConditionObject 是 AQS 的内部类

```java
public class ConditionObject implements Condition{
	
    private Node firstWaiter;
    private Node lastWaiter;
}
```



<img src="https://gitee.com/Jackpotsss/pic_go/raw/master/img/image-20200328110238704.png" alt="image-20200328110238704" style="zoom:80%;" />

​	这里有个细节，Node 是 AQS 的内部类，是同步队列和等待队列共用的节点，但是同步队列是一个双向链表，而等待队列是一个单向链表。


​	
```java
	static final class Node {
        
        /** 标记为共享模式下的等待节点 */
        static final Node SHARED = new Node();
        /** 标记为独占模式下的等待节点 */
        static final Node EXCLUSIVE = null;

        /** waitStatus value to indicate thread has cancelled */
        static final int CANCELLED =  1;
        /** waitStatus value to indicate successor's thread needs unparking */
        static final int SIGNAL    = -1;
        /** waitStatus value to indicate thread is waiting on condition */
        static final int CONDITION = -2;
        /**
         * waitStatus value to indicate the next acquireShared should
         * unconditionally propagate
         */
        static final int PROPAGATE = -3;
        
        volatile int waitStatus;
    	volatile Node prev;
        volatile Node next;
        volatile Thread thread;
        Node nextWaiter;
        
        final boolean isShared() {
            return nextWaiter == SHARED;
        }
        
        Node() {    // Used to establish initial head or SHARED marker
        }

        Node(Thread thread, Node mode) {     // Used by addWaiter
            this.nextWaiter = mode;
            this.thread = thread;
        }

        Node(Thread thread, int waitStatus) { // Used by Condition
            this.waitStatus = waitStatus;
            this.thread = thread;
        }
	}
```



​	调用condition.await() 方法的前提是当前线程获取了condition相关联的锁，condition.await() 方法返回的前提(除了中断和超时)也必须是当前线程获取了condition相关联的锁。

**在Java对象的监视器模型上，一个对象拥有一个同步队列和等待队列，而并发包中的同步器拥有一个同步队列和多个等待队列。**

<img src="https://gitee.com/Jackpotsss/pic_go/raw/master/img/image-20200326135351176.png" alt="image-20200326135351176" style="zoom:50%;" />



## 6.7 小结

​	在Java中，关于线程“阻塞”的状态就有三种：BLOCKED、WAITING、TIMED_WAITING；其实在操作系统中，线程阻塞只用一种状态表示：WAITING；

​	其中在 Lock + Condition 实现的锁机制中，不会使线程有 BLOCKED 这种状态，使线程处于 BLOCKED 状态的行为只有一种，那就是线程阻塞于 synchronize 关键字修饰的代码块或方法上。

**小问题**

​	在 BLOCKED 这个线程状态的设计上，我不能理解为什么多设计出一个BLOCKED 状态，阻塞在 synchronize 块的线程为 BLOCKED 状态，而在同步块里调用 wait（）方法使线程成为WAITING状态，而两种行为的效果是一样的，都是阻塞线程；这种情况在   Lock + Condition 并没有，目前笔者想不出为什么非得设计 BLOCKED 状态，我认为WAITING完全可以替代BLOCKED ，从而使线程的状态设计的更简单些。（2020-04-12）



# 7. 并发容器和框架

​	并发包提供了一系列常用的并发安全的集合容器供开发者使用。

​	线程安全的队列有两种，非阻塞队列和阻塞队列。非阻塞队列有  ConcurrentLinkedQueue 和 ConcurrentLinkedDeque；阻塞队列实现 BlockingQueue  接口，它的主要实现有 LinkedBlockingQueue, ArrayBlockingQueue, SynchronousQueue, PriorityBlockingQueue, 和 DelayQueue，不同的类涵盖了生产者-消费者、消费传递、并行任务和相关并发设计的常见用法。

​	TransferQueue 接口和实现该接口的 LinkedTransferQueue 提供了一种同步传输数据的的方式，生产者可以阻塞等待消费者。

​	BlockingDeque 接口和实现类 LinkedBlockingDeque 提供了阻塞双端队列，支持 FIFO 和 LIFO。

​	除了队列之外，JUC包还提供了专门在多线程环境下使用的集合：ConcurrentHashMap，ConcurrentSkipListMap，ConcurrentSkipListSet，CopyOnWriteArrayList 和 CopyOnWriteArraySet。在多线程环境下，ConcurrentHashMap 通常比同步的HashMap更可取，而ConcurrentSkipListMap通常比同步的TreeMap更可取。 当预期的读取和遍历次数大大超过列表的更新次数时（多读少写），CopyOnWriteArrayList 也优于同步的 ArrayList。



## 7.1 并发容器一览表

| 接口/类               | 描述                           |
| --------------------- | ------------------------------ |
| BlockingQueue         | 阻塞队列接口                   |
| LinkedBlockingQueue   | 阻塞队列【链表实现】           |
| ArrayBlockingQueue    | 阻塞队列【数组实现】           |
| SynchronousQueue      | 同步队列（不存储元素）         |
| PriorityBlockingQueue | 优先阻塞队列                   |
| DelayQueue            | 延迟队列（非常实用）           |
| BlockingDeque         | 阻塞双端队列接口               |
| LinkedBlockingDeque   | 阻塞双端队列                   |
| LinkedTransferQueue   | 传输队列（同步传输）           |
| ConcurrentLinkedQueue | 并发安全的队列（非阻塞）       |
| ConcurrentLinkedDeque | 并发安全的双端队列（非阻塞）   |
| ConcurrentHashMap     | 并发安全的HashMap              |
| ConcurrentSkipListMap | 并发安全的Map（跳表实现）      |
| ConcurrentSkipListSet | 并发安全的Set（跳表实现）      |
| CopyOnWriteArrayList  | 并发安全的动态数组（写时复制） |
| CopyOnWriteArraySet   | 并发安全的数组集合（写时复制） |



## 7.2 Queue 队列

**并发安全的队列有两种：非阻塞队列和阻塞队列。** 

**非阻塞队列使用volatile + 循环 CAS 保证线程安全，阻塞队列通过维护同步状态然后控制线程的阻塞还是唤醒保证线程安全。** 

### 非阻塞队列

并发包提供了两个非阻塞并发安全的队列：

- ConcurrentLinkedQueue	
- ConcurrentLinkedDeque



#### ConcurrentLinkedQueue

ConcurrentLinkedQueue 是并发安全的无界非阻塞队列，基于链表实现。使用 volatile + 循环CAS 的方式保证线程安全。

```java
//部分核心代码:
public class ConcurrentLinkedQueue<E> implements Queue<E>{
    
    //入队:将指定元素插入到队列尾部
    public boolean offer(E e) {
        //构建新节点
      	final Node<E> newNode = new Node<E>(e); 
        //无限循环
        for (Node<E> t = tail, p = t;;) {
            Node<E> q = p.next;
            if (q == null) {
                if (p.casNext(null, newNode)) {//成功CAS:将新节点插入到队列尾部
                    if (p != t) casTail(t, newNode);  // 这一步是允许失败的.
                    return true;
                }
            }
        }
    }
    //出队:获取并移除队首元素
    public E poll() {
        //无限循环
        for (;;) {
            for (Node<E> h = head, p = h, q;;) {
                E item = p.item;
                //成功CAS:将队首元素删除
                if (item != null && p.casItem(item, null)) {
                    if (p != h) updateHead(h, ((q = p.next) != null) ? q : p);
                    return item;
                }
            }
        }
    }
}
```



#### ConcurrentLinkedDeque

​	ConcurrentLinkedDeque 与 ConcurrentLinkedQueue 的区别就在于它是一个并发安全的**非阻塞双端队列**。下面讲阻塞队列还会降到 LinkedBlockingDeque，它是并发安全的**阻塞双端队列**。

### 阻塞队列

阻塞队列实现 BlockingQueue 接口，阻塞队列有两个阻塞行为:

- 新增：当队列已满时，新增元素会阻塞线程，直到队列不满。
- 删除：当队列为空时，删除元素会阻塞线程，直到队列不空。



#### BlockingQueue

**阻塞队列的接口方法:** 

|         | Throws exception | Special value | Blocks | Times out            |
| ------- | ---------------- | ------------- | :----- | :------------------- |
| Insert  | add(e)           | offer(e)      | put(e) | offer(e, time, unit) |
| Remove  | remove()         | poll()        | take() | poll(time, unit)     |
| Examine | element()        | peek()        | --     | --                   |

​	BlockingQueue 接口方法提供了操作数据的四种不同形式，添加和删除数据有**抛异常、返回特殊值、阻塞、等待超时**四种不同的表现，查找数据有抛异常和返回特殊值两种不同的表现。

​	抛异常和返回特殊值是父接口Queue 就有的，BlockingQueue 主要提供可阻塞的put和 take 方法，以及可定时的 offer 和 poll 方法。如果队列已经满了，那么执行put 方法会阻塞，直到队列不满；如果队列为空，执行 take 方法会阻塞，直到队列不空。

​	队列可以是有界的也可以是无界的，无界队列永远不会满，所以put 方法永远不会阻塞。



**阻塞队列的主要实现：** 

| 接口/类               | 描述                 |
| --------------------- | -------------------- |
| LinkedBlockingQueue   | 阻塞队列（链表实现） |
| ArrayBlockingQueue    | 阻塞队列（数组实现） |
| DelayQueue            | 延迟队列             |
| PriorityBlockingQueue | 优先阻塞队列         |
| SynchronousQueue      | 同步阻塞队列         |



#### LinkedBlockingQueue

​	LinkedBlockingQueue 是链表实现的阻塞队列。可通过构造器指定队列的最大容量，若不指定默认容量为 Integer.MAX_VALUE（约21亿），即无界阻塞队列。

​	内部使用**两把互斥锁和两个等待队列**保证数据安全，所有的入队操作使用一把锁，所有的出队操作使用另一把锁。



```java
//LinkedBlockingQueue 部分核心代码：
public class LinkedBlockingQueue<E> implements BlockingQueue<E>{

	private final int capacity;
	private final AtomicInteger count = new AtomicInteger();
	Node<E> head;
	private Node<E> last;

    /** 入队操作的锁 */
    private final ReentrantLock putLock = new ReentrantLock();
    /** 入队操作的等待队列 */
    private final Condition notFull = putLock.newCondition();
    /** 出队操作的锁 */
    private final ReentrantLock takeLock = new ReentrantLock();
    /** 出队操作的等待队列 */
    private final Condition notEmpty = takeLock.newCondition();
    
    static class Node<E> {
        E item;
        Node<E> next;
        Node(E x) { item = x; }
    }
}
```



#### ArrayBlockingQueue

​	ArrayBlockingQueue是**数组**实现的**有界阻塞队列**，队列内的元素FIFO（先进先出）。元素在队列尾部添加，在队列头部获取或移除。

​	ArrayBlockingQueue 是一个经典的有界缓冲区，队列一旦创建，容量无法更改。生产者插入元素，消费者提取元素，将元素添加到一个装满的队列，将导致线程阻塞，直到队列不满；从空队列中移除元素同样导致线程阻塞，直到队列不空。

​	支持可选的公平性策略，默认是非公平的，通过构造器设置为公平性的队列按照FIFO的顺序授予线程访问权限。一般来说，公平性会降低吞吐量，但可以有效避免线程饥饿。

**实现** 

内部使用**一把互斥锁和两个等待队列**保证数据安全，所有的入队和出队操作使用同一把锁。



```java
public class ArrayBlockingQueue<E> implements BlockingQueue<E>{
	
    final Object[] items;
    int takeIndex;
    int putIndex;
    int count;
    
    /** 访问队列的主锁 */
    final ReentrantLock lock;
    /** 获取元素的等待队列 */
    private final Condition notEmpty;
    /** 插入元素的等待队列 */
    private final Condition notFull;
    
    public ArrayBlockingQueue(int capacity, boolean fair) {
        if (capacity <= 0) throw new IllegalArgumentException();
        this.items = new Object[capacity];
        lock = new ReentrantLock(fair);
        notEmpty = lock.newCondition();
        notFull =  lock.newCondition();
    }
    //入队
    public void put(E e) throws InterruptedException {
        checkNotNull(e);
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
            while (count == items.length)
                notFull.await();
            enqueue(e);
        } finally {
            lock.unlock();
        }
    }
    //出队
    public E take() throws InterruptedException {
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
            while (count == 0)
                notEmpty.await();
            return dequeue();
        } finally {
            lock.unlock();
        }
    }
    
    public int size() {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            return count;
        } finally {
            lock.unlock();
        }
    }
}
```



#### SynchronousQueue

​	SynchronousQueue是一个同步阻塞队列，同步队列不存储任何元素，是一个空队列。每一个插入操作必须等待另一个线程的删除操作，也就是说，只有当删除时，元素才存在。同步队列不能进行迭代，因为没有迭代的内容。

​	打个比方，你负责给同事发送文件，你是直接送到他手里呢，还是先发送到他的邮箱呢？这两种情况都有可能。他可能现在没有时间接文件，先发送到他的邮箱，等他有时间了再去看文件，这是异步阻塞队列的效果；当他有时间，或者说这份文件很急，必须当面交到他手里然后处理，这种情况就只能送到他手里了，这就是同步队列的效果。

**适用场景** 

​	仅当有足够多的消费者，并且总有一个消费者准备好获取数据时，同步队列才适用。



#### PriorityBlockingQueue

​	PriorityBlockingQueue 是一个有界的优先阻塞队列，内部使用**优先队列**存储元素，使用一把互斥锁和一个等待队列保证数据安全。

优先队列使用数组实现。



```java
public class PriorityBlockingQueue<E> implements BlockingQueue<E> {

    private Object[] queue;
    private int size;
    private Comparator<? super E> comparator;
    
    private final ReentrantLock lock;
    private final Condition notEmpty;
    private volatile int allocationSpinLock;
    
    private PriorityQueue<E> q;
    
    public PriorityBlockingQueue(int initialCapacity,Comparator<? super E> comparator) {
        if (initialCapacity < 1) throw new IllegalArgumentException();
        this.lock = new ReentrantLock();
        this.notEmpty = lock.newCondition();
        this.comparator = comparator;
        this.queue = new Object[initialCapacity];
    }
}
```



#### DelayQueue

延迟队列是一个非常实用的阻塞队列。常用于构建定时任务执行器、缓存淘汰机制。

DelayQueue 内部使用**优先队列**存储元素，比较的是延迟时间，使用一把互斥锁保证线程安全。

```java
//核心源码
public class DelayQueue<E extends Delayed> implements BlockingQueue<E> {
    
    private final ReentrantLock lock = new ReentrantLock();
    private final Condition available = lock.newCondition();
    
    private final PriorityQueue<E> q = new PriorityQueue<E>();  
    private Thread leader = null;
    public DelayQueue() {}
    
    //入队：添加元素到队列尾部
    public boolean offer(E e) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            q.offer(e);
            if (q.peek() == e) {
                leader = null;
                available.signal();
            }
            return true;
        } finally {
            lock.unlock();
        }
    }
    //出队:删除并返回队首元素
    public E poll() {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            E first = q.peek();
            if (first == null || first.getDelay(NANOSECONDS) > 0)
                return null;
            else
                return q.poll();
        } finally {
            lock.unlock();
        }
    }
}
```



​	Delayed 延迟接口，该接口的实现类提供 compareTo 方法，排序规则与getDelay 方法一致。getDelay 方法返回关联对象的剩余时间。

```java
public interface Delayed extends Comparable<Delayed> {
    long getDelay(TimeUnit unit);
}
```



#### 生产者-消费者模式

​	阻塞队列可以很容易的构建生产者-消费者这种设计模式。该模式把“需要完成的工作”与“执行工作”这两个过程分离开来，工作项放入一个“代办”列表中，以便后续处理，而不是有了工作项后立即处理。

​	当数据生成时，生产者把数据放入队列，消费者从队列中取数据使用。生产者不需要关心消费者的标识和数量，也不用知道自己是不是唯一的生产者，只需要将数据放入队里即可；同样的，消费者也不用关心生产者的标识和数量，只需要从队列中消费数据即可。生产者和消费者的数量可以是任意的。

​	常见的生产者-消费者模式便是线程池，提交到线程池的任务是数据，线程池把任务放到阻塞队列中，线程池中的线程不断的执行任务，从而消费数据。在线程池中，提交任务的对象是生产者，执行任务的线程时消费者。

#### BlockingDeque

​	Java6 增加了两个容器类型：Deque 和 BlockingDeque，它们分别对 Queue 和 BlockingQueue进行拓展。Deque 是一个双端队列，实现了在队头和队尾的高效插入和删除。具体实现包括 ArrayDeque 和 LinkedBlockingDeque。

​	阻塞双端队列的接口方法有很多，它提供了操作**队首和队尾**的**增删查**三种操作，并且每种操作提供了四种不同的行为（抛异常、返回特殊值、阻塞、等待超时）。



操作队首	First Element (Head)

|         | Throws exception | Special value | Blocks      | Times out                 |
| ------- | ---------------- | ------------- | ----------- | ------------------------- |
| Insert  | addFirst(e)      | offerFirst(e) | putFirst(e) | offerFirst(e, time, unit) |
| Remove  | removeFirst()    | pollFirst()   | takeFirst() | pollFirst(time, unit)     |
| Examine | getFirst()       | peekFirst()   | --          | --                        |

操作队尾	Last Element (Tail)

|         | Throws exception | Special value | Blocks     | Times out                |
| ------- | ---------------- | ------------- | ---------- | ------------------------ |
| Insert  | addLast(e)       | offerLast(e)  | putLast(e) | offerLast(e, time, unit) |
| Remove  | removeLast()     | pollLast()    | takeLast() | pollLast(time, unit)     |
| Examine | getLast()        | peekLast()    | --         | --                       |

下面是双端队列针对队列接口的等价方法：

**Insert**

| BlockingQueue Method | Equivalent BlockingDeque Method |
| -------------------- | :------------------------------ |
| add(e)               | addLast(e)                      |
| offer(e)             | offerLast(e)                    |
| put(e)               | putLast(e)                      |
| offer(e, time, unit) | offerLast(e, time, unit)        |

**Remove**

| BlockingQueue Method | Equivalent BlockingDeque Method |
| -------------------- | :------------------------------ |
| remove()             | removeFirst()                   |
| poll()               | pollFirst()                     |
| take()               | takeFirst()                     |
| poll(time, unit)     | pollFirst(time, unit)           |

**Examine**

| BlockingQueue Method | Equivalent BlockingDeque Method |
| -------------------- | :------------------------------ |
| element()            | getFirst()                      |
| peek()               | peekFirst()                     |



#### LinkedBlockingDeque

LinkedBlockingDeque 是 双端阻塞队列的一个主要实现，LinkedBlockingDeque 内部使用一把锁保证数据安全。

```java
public class LinkedBlockingDeque<E> implements BlockingDeque<E>{
    
    Node<E> first;
    Node<E> last;
    
    final ReentrantLock lock = new ReentrantLock();
    /** Condition for waiting takes */
    private final Condition notEmpty = lock.newCondition();
    /** Condition for waiting puts */
    private final Condition notFull = lock.newCondition();

    public void putFirst(E e) throws InterruptedException {
        if (e == null) throw new NullPointerException();
        Node<E> node = new Node<E>(e);
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            while (!linkFirst(node))
                notFull.await();
        } finally {
            lock.unlock();
        }
    }   
    
    private boolean linkFirst(Node<E> node) {

        if (count >= capacity)  return false;
        Node<E> f = first;
        node.next = f;
        first = node;
        if (last == null) last = node;
        else  f.prev = node;
        ++count;
        notEmpty.signal();
        return true;
    }    
    
    public E takeFirst() throws InterruptedException {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            E x;
            while ( (x = unlinkFirst()) == null)
                notEmpty.await();
            return x;
        } finally {
            lock.unlock();
        }
    }
    
    private E unlinkFirst() {

        Node<E> f = first;
        if (f == null)  return null;
        Node<E> n = f.next;
        E item = f.item;
        f.item = null;
        f.next = f; // help GC
        first = n;
        if (n == null) last = null;
        else  n.prev = null;
        --count;
        notFull.signal();
        return item;
    }
}
```



#### 工作密取设计

​	正如阻塞队列适用于生产者-消费者模式，双端队列同样适用于另一种相关模式，即工作密取（work-stealing）。在阻塞队列中，所有的消费者共用一个工作队列，而在工作密取模式中，每个消费者都有自己的双端队列，**如果消费者完成了自己的所有任务，还可以获取其他消费者双端队列的任务来执行**。

​	工作密取比传统的生产者-消费者模式具有更高的可伸缩性，这是因为消费者大多数情况下不会在一个工作队列上进行竞争。**当工作线程需要访问另一个双端队列时，它会从队列的尾部而不是头部获取数据，这样的设计进一步降低了队列的竞争程度。**

​	Fork-Join 框架内部使用双端阻塞队列装载任务，使用的就是工作密取设计。



## 7.3 ConcurrentHashMap

​	在说 ConcurrentHashMap 之前，有必要说一下 HashMap 和 HashTable，它们三个都是基于哈希表实现的Map，不同的是，在并发环境下 HashMap 线程不安全，HashTable 虽然可以保证线程安全，但并发效率却不高。HashTable 和 synchronize-Map 在每个方法上加入锁同步，同一时刻只允许一个线程操作Map，并发度低；ConcurrentHashMap 使用更细的锁力度提供更好的并发性能，即同一时刻可以有多个线程操作Map。

### ConcurrentHashMap 

​	首先声明一下，ConcurrentHashMap 是JDK1.5加入的类，1.7版本源码量为 1600 行，而1.8版本的源码量为6300行，是上一个版本的4倍！从代码量就知道，两个版本的实现方式差别很大。

​	无论哪个版本的 ConcurrentHashMap ，作者都是 Doug Lea，这就是所谓的 “我自己提出，我自己设计，我自己升级，然后我超越自己”，不得不承认，人牛到一定程度，做什么都是规范，都是标准。



#### 第一版本（JDK1.5 - 1.7）

​	先说下1.7及之前版本的设计方式，ConcurrentHashMap 使用**分段锁** （Segment ）提高并发效率。



<img src="https://gitee.com/Jackpotsss/pic_go/raw/master/img/image-20200412182911237.png" alt="image-20200412182911237" style="zoom:80%;" />



​	第一版本采用 **Segment 数组+ HashEntry 数组**的方式实现，Segment 本身是一个重入的互斥锁，一个 Segment 里面包含一个 HashEntry 数组，HashEntry 存储key和value，并且可以形成单链表，以拉链的方式解决哈希冲突。

```java
public class ConcurrentHashMap<K,V> implements ConcurrentMap<K,V>{
	
    final Segment<K,V>[] segments;
   
    static final class Segment<K,V> extends ReentrantLock{
        
        volatile HashEntry<K,V>[] table;
    }
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
    }
}
```

​	ConcurrentHashMap 在初始化时，会计算出Segment 数组的大小和每个 Segment 中HashEntry 数组的大小。并发度（Segment 数组的长度）默认为16，如果Map的初始化容量为64，那么每个  Segment 中HashEntry 数组的大小是64 / 16 = 4；并发度是多少，理论上最多支持多少个线程同时并发访问Map。



#### 第二版本（JDK1.8 及以后）

​	第二版本完全放弃了分段锁segment的设计，锁力度进一步降低，没有了 Segment分段锁数组，使用Node 代替原来的HashEntry，存储K-V数据（其实就是换了个名字）。

```java
public class ConcurrentHashMap<K,V> implements ConcurrentMap<K,V>{
	
    volatile Node<K,V>[] table;

	static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        volatile V val;
        volatile Node<K,V> next;

        Node(int hash, K key, V val, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.val = val;
            this.next = next;
        }
    }
}
```

**保证线程安全** 

​	在 Map的 put()，remove() ，replace() ，clear()  等修改Node 的方法中，使用**内置锁synchronized 保证数据安全**，加锁的是 Node对象。也就是说只有多个线程同时操作了同一个Node（K-V数据），才存在竞争关系，此时内置锁使用同步机制保证线程安全，这种设计使得线程竞争的程度大大降低。

​	此外在循环迭代器时，并发修改数据不会引发 ConcurrentModificationException异常，但这有可能会造成数据不准确，ConcurrentHashMap 的迭代器被设计一次只能由一个线程使用，提供了弱一致性的数据保证。

​	与HashTable 一样，但与HashMap不同，此类不允许将null 值作为键或值。



### Map的原子操作

ConcurrentMap 接口定义了Map相关的线程安全的原子操作方法（非常实用）：

```java
public interface ConcurrentMap<K, V> extends Map<K, V> {
   	//仅当key没有映射值时才插入
	V putIfAbsent(K key, V value);

	//仅当key映射到value时才删除
	boolean remove(Object key, Object value);

	//仅当key映射到oldValue时才替换为newValue
	boolean replace(K key, V oldValue, V newValue);

	//仅当key有映射值时才替换为value
	V replace(K key, V value); 
}
```

​	使用过Redis服务的应该熟悉， Redis 的 `SETNX`  指令和Map接口的putIfAbsent 方法是一样的，在ConcurrentMap 中,该操作也具有原子性。

## 7.4 ConcurrentSkipList

​	ConcurrentSkipList是并发安全的跳跃表。

​	我们知道二叉查找树的平均查找效率是O(lgN)，效率已经很高了，跳跃表的效率可以和二叉查找树相媲美。有意思的是，在Java类库中，并没有提供线程不安全的跳跃表，只提供了并发安全的跳跃表，即 ConcurrentSkipListMap 和 ConcurrentSkipListSet，它们为get、put、remove和containsKey 操作及其变体提供了平均 log（n）的时间成本，增删改查可以在多线程并发环境下安全使用。

​	HashSet 底层使用 HashMap 实现，同样的设计， ConcurrentSkipListSet 底层使用 ConcurrentSkipListMap实现的。

### ConcurrentSkipListMap

在多线程环境下，ConcurrentSkipListMap 要比 **同步的 TreeMap** 更可取。



### ConcurrentSkipListSet

ConcurrentSkipListSet 底层使用 ConcurrentSkipListMap 实现，是一个跳表实现的，并发安全的Set。



## 7.5 CopyOnWrite 写时复制

​	CopyOnWrite  写时复制的思想，即读取数据不加锁，在对容器进行写操作（增删改）时，拷贝源数据，然后在新容器中进行写操作，写完之后，把原来的数组引用指向新容器；

​	**CopyOnWrite 容器的读操作是不需要加锁的**，这一点和读锁不同，在加读锁的情况下，其他线程只能加读锁，不能加写锁，并且一直在维护读写锁的状态，所以在读取次数远大于更新次数时，CopyOnWrite 的并发性能更高一些。

### CopyOnWriteArrayList

当预期的读取和遍历次数大大超过列表的更新次数时，CopyOnWriteArrayList 要优于**同步的 ArrayList** ，

```java
//部分核心代码:
public class CopyOnWriteArrayList<E> implements List<E>{
    
    //只能通过getArray 和 setArray访问
    private volatile Object[] array;
    //全局一把互斥锁:所有修改操作使用同一把锁;
 	final ReentrantLock lock = new ReentrantLock();
    
    final Object[] getArray() { return array; } 
    final void setArray(Object[] a) { array = a; }
    
    public CopyOnWriteArrayList() {
        setArray(new Object[0]);
    }
    //读取数据不加锁
    public E get(int index) {
        return get(getArray(), index);
    }
    private E get(Object[] a, int index) {return (E) a[index];}
    
    //修改数据前需要加锁,并且需要复制一份源数据
    public boolean add(E e) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            Object[] elements = getArray();
            int len = elements.length;
            Object[] newElements = Arrays.copyOf(elements, len + 1);//赋值源数据,并新增元素
            newElements[len] = e;
            setArray(newElements);//直接修改array的引用
            return true;
        } finally {
            lock.unlock();
        }
    }
    //另外add(int index, E element),E remove(int index),E set(int index, E element)等修改操作同上面操作一样,
    //修改数据前需要加锁,并且需要复制一份源数据
}
```

​	所有的修改操作（add、set、remove等）都需要先获取锁，然后复制一份源数据，在复制出来的新数组上进行修改操作，最后把新数组的引用重新赋值给array 变量。

​	array 数组被 **volatile** 修饰，所有的修改操作需要获取同一把**互斥锁**，所以在并发读和写的情况下，对array 数组最多只会出现一写多读的情况，我们知道，在**一写多读**的场景下，volatile 是可以保证内存可见性的，所以可以保证并发数据的安全。

​	当然，在修改频率高的并发场景下，CopyOnWrite 容器并不适用。

### CopyOnWriteArraySet

CopyOnWriteArraySet 的所有操作都被代理到CopyOnWriteArrayList 上，即内部使用 CopyOnWriteArrayList 完成所有操作。

```java
public class CopyOnWriteArraySet<E>{

    private final CopyOnWriteArrayList<E> al;
    
    public CopyOnWriteArraySet() {
        al = new CopyOnWriteArrayList<E>();
    }
    
    public boolean add(E e) { return al.addIfAbsent(e);}
    public boolean remove(Object o) {return al.remove(o);}
    
    public void clear() {al.clear();}
    public int size() {return al.size();}
    public boolean isEmpty() {return al.isEmpty();} 
    public boolean contains(Object o) {return al.contains(o);} 
}
```



## 7.6 Fork/Join 框架

​	Fork/Join 是一个基于分治思想的并行执行任务的框架，把一个大任务分割成若干个小任务，最终汇总每个小任务的执行结果得到大任务结果的框架。

<img src="https://gitee.com/Jackpotsss/pic_go/raw/master/img/image-20200405111655966.png?raw=true" alt="image-20200405111655966" style="zoom: 80%;" />

<img src="..\md_resource\image-20200405111655966.png" alt="image-20200405111655966" style="zoom: 20%;" />

Arrays.parallelSort() 并行排序就是通过 Fork/Join 框架实现的。



# 8. 原子操作类

## 8.1 atomic 包

​	原子操作类都在 `java.util.concurrent.atomic` 包中，这个包的原子操作类提供了一种**用法简单、性能高效、线程安全**地更新一个变量的方式。原子操作类提供了4种类型的原子更新方式，分别是：

1. 原子更新基本类型
2. 原子更新数组
3. 原子更新引用
4. 原子更新字段



## 8.2 原子更新基本类型

- AtomicBoolean	原子更新布尔类型
- AtomicInteger      原子更新整型
- AtomicLong          原子更新长整型



## 8.3 原子更新数组

- AtomicIntegerArray	原子更新整型里的元素
- AtomicLongArray        原子更新长整型里的元素
- AtomicReferenceArray<E>     原子更新引用类型数组里面的元素



## 8.4 原子更新引用

- AtomicReference<V>								原子更新引用类型
- AtomicMarkableReference<V>               原子更新带有**布尔标记**的引用类型
- AtomicStampedReference<V>                原子更新带有**整型版本号**的引用类型



## 8.5 原子更新字段

- AtomicIntegerFieldUpdater<T>			  原子更新整型字段的更新器
- AtomicLongFieldUpdater<T>                  原子更新长整型字段的更新器
- AtomicReferenceFieldUpdater<T, V>     原子更新引用类型里字段的更新器



​	**想要原子的更新字段类需要两步：**

1. 更新类的字段必须使用 `public volatile` 修饰符；
2. 原子更新字段的更新器都是抽象类，只能使用静态方法 newUpdater() 创建一个更新器，然后设置需要更新的类和字段；



## 8.6 LongAdder

JDK8 新增的原子操作类，比AtomicLong  性能更好，减少乐观锁的重试次数。



## 8.7 原理

**原理：**

​	**volatile + 循环CAS **



看一下 AtomicInteger 是怎么实现的，下面是部分源码：

```java
public class AtomicInteger {
    
    private static final Unsafe unsafe = Unsafe.getUnsafe();
    private volatile int value;
    
    public final int get() {
        return value;
    }
    public final void set(int newValue) {
        value = newValue;
    }
    
    /**
     * 以原子方式设置为给定值并返回旧值。
     * @param	新值
     * @return 先前的值
     */
    public final int getAndSet(int newValue) {
        for (;;) {
            int current = get();
            if (compareAndSet(current, newValue))
                return current;
        }
    }
    
    /**
     * 以原子方式将当前值 +1
     * @return 先前的值
     */
    public final int getAndIncrement() {
        for (;;) {
            int current = get();
            int next = current + 1;
            if (compareAndSet(current, next))
                return current;
        }
    }
    
    /** 如果当前值和期望值相等，则以原子方式将该值设置为给定的更新值 */
     public final boolean compareAndSet(int expect, int update) {
        return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
    }
}
```

​	AtomicInteger 的更新操作并没有加互斥锁，而是使用 volatile + 循环CAS 的方式进行安全更新。 volatile 的内存可见性和 CAS 的原子性为原子操作类线程安全的更新值提供了保障。



# 9. Executor 框架



## 9.1 Executor 任务执行器

​	Executor 提供一种思想：将任务的提交和执行进行解耦，分离开来。

​	用户无需关心线程如何创建、执行，用户只需要提供任务对象，将任务的执行逻辑交给执行器 Executor 即可。Executor 完成线程的调度和任务的执行。

```java
public interface Executor {
	void execute(Runnable command);
}
```

​	**Executor 框架是基于生产者-消费者模式设计的，提交任务相当于生产者，执行任务则相当于消费者。**如果要在程序中实现一个生产者-消费者的设计，那么通常最简单的方式就是使用Executor 实现。



## 9.2 ExecutorService

ExecutorService 定义了异步任务执行器的基本动作，包含了任务的提交和执行，以及关闭操作。



```java
public interface ExecutorService extends Executor {
    
    Future<T> submit(Callable<T> task);
    Future<?> submit(Runnable task);
    Future<T> submit(Runnable task, T result);
    
    List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks) throws InterruptedException;
    List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks,long timeout, TimeUnit unit) 
        throws InterruptedException;
    
    T invokeAny(Collection<? extends Callable<T>> tasks) throws InterruptedException, ExecutionException;
    T invokeAny(Collection<? extends Callable<T>> tasks,long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException;
    
    void shutdown();
    
    List<Runnable> shutdownNow();
    
    boolean isShutdown();
    
    boolean isTerminated();
    
    boolean awaitTermination(long timeout, TimeUnit unit) throws InterruptedException;
}
```



## 9.3 带有返回值的任务：Callable和Future 

Callable 和 Future 都是与**有返回值的任务**有关的接口。

Callable 代表有返回结果的任务，可以抛出异常。JDK1.5加入的接口。

```java
public interface Callable<V> {
	V call() throws Exception;
}
```



### Callable 和 Runnable 

​	Callable 和 Runnable 都是任务，只不过 Callable 有返回值，并且可以抛出异常，而 Runnable 无返回值，也不能抛出异常。Runnable 接口从JDK1.0版本就存在，是元老级别的接口，Callable 接口是JDK1.5版本引入的。

​	Thread 的构造方法里并不能加入一个Callable ，只能放入无返回值的Runnable ，这是早期的设计决定的。所以Thread 线程要运行一个有返回值的Callable 任务，需要一个**适配器**，将Callable接口转化为Runnable 接口。

```java
//FutureTask部分代码(已修改)
public class FutureTask<V> implements Runnable{
    
	private Callable<V> callable;
    private Object outcome;
    
    public FutureTask(Callable<V> callable) {
        this.callable = callable;
    }
    public void run() {
        outcome = callable.call();
    }
}
```

​	**FutureTask** 是一个Runnable，调用run方法会代理到call方法上，同时将执行结果保存在全局变量中，FutureTask 是一个将Callable接口转化为Runnable 接口的适配器。当然FutureTask 的功能远不止这些，其他功能会在下面详细讲解。

​	并发包同样也提供了将Runnable 接口转化为Callable接口的适配器-**RunnableAdapter** ，可通过 Executors.callable(runnable, result) 静态方法实现。

```java
//将Runnable 转换为 Callable
public static <T> Callable<T> callable(Runnable task, T result) {
    if (task == null) throw new NullPointerException();
    return new RunnableAdapter<T>(task, result);
}

static final class RunnableAdapter<T> implements Callable<T> {
    final Runnable task;
    final T result;
    RunnableAdapter(Runnable task, T result) {
        this.task = task;
        this.result = result;
    }
    public T call() {
        task.run();
        return result;
    }
}
```



### Future

Future 是**用于管理任务的生命周期**而抽象出来的一个接口。获取任务执行结果，处理异常，取消任务，判断是否取消和完成。

```java
public interface Future<V> {
    //阻塞获取执行结果
    V get() throws InterruptedException, ExecutionException;
    //超时阻塞获取执行结果
    V get(long timeout, TimeUnit unit) throws InterruptedException, ExecutionException, TimeoutException;
    //取消任务
    boolean cancel(boolean mayInterruptIfRunning);
    //是否取消
    boolean isCancelled();
    //是否完成
    boolean isDone();
}
```

**生命周期** 

Future 的生命周期有3个状态：

- 未启动
- 已启动
- 已结束



​	其中任务已结束又分为**正常结束、异常结束、中断结束、取消结束**这四种，分别代表任务执行的四种情况。Future 接口的 `cancel(boolean mayInterruptIfRunning)` 方法的入参，用于控制是中断任务还是取消任务，true 代表中断，false代表取消；在任务启动之前执行 cancel 方法，无论入参是什么，任务都不会再执行；在任务执行过程中（正在执行run方法），取消任务是无效的，中断任务是可以的；在任务执行完毕后执行cancel 方法会立即返回false；

​	**cancel（true）是真正去中断线程，打上中断标记，如果任务可以响应中断，那么就会中断任务**；cancel（false）不会去中断线程，只是去改变Future 本身的状态，而这个状态在任务执行开始时会进行校验，所以一定要在任务执行开始之前取消任务才有效，一旦任务运行起来，取消是无效的。cancel（false）给开发者的意思是**任务要么不执行，要么执行完毕，不要在执行过程中中断**。如果你不确定中断一个任务有什么后果，可以使用cancel（false）。



Future接口的get() 方法会阻塞式的获取执行结果：

- 如果任务没有执行结束，线程会一直阻塞直到任务完成；
- 如果任务执行过程中抛出异常，get() 方法会抛出 ExecutionException 异常；
- 如果任务被中断， get() 方法会抛出 InterruptedException异常；
- 如果任务被取消， get() 方法会抛出受检查的 CancellationException异常；
- 如果是定时获取执行结果，超时后会抛出 TimeoutException异常；



​	ExecutorService 和 Future接口是一组CP（好搭档），ExecutorService.submit 方法返回一个Future用来管理任务，取消或中断任务，判断是否取消或完成以及获取任务执行结果。

```java
interface ArchiveSearcher { 
    String search(String target); 
}

class App {
   ExecutorService executor = ...
   ArchiveSearcher searcher = ...
   void showSearch(final String target) throws InterruptedException {
     Future future = executor.submit(new Callable() {
         public String call() {
             return searcher.search(target);
         }});
     displayOtherThings(); // do other things while searching
     try {
       displayText(future.get()); // use future
     } catch (ExecutionException ex) { cleanup(); return; }
   }
}}
```



## 9.4 FutureTask

FutureTask 实现Future和 Runnable 接口，表示一个异步任务，并管理其生命周期。



<img src="https://gitee.com/Jackpotsss/pic_go/raw/master/img/image-20200403151324394.png" alt="image-20200403151324394" style="zoom: 80%;" />

构造方法中传入一个任务：

<img src="https://gitee.com/Jackpotsss/pic_go/raw/master/img/image-20200624062228808.png" alt="image-20200624062228808" style="zoom:80%;" />

因为FutureTask 既可以管理任务生命周期，同时也是一个任务，所以FutureTask 和 Executor 的使用方式有两种：

```java
public static void exec(ExecutorService executor) {
    
    //方式一: Future作为 executor.submit方法的返回值
	Future<?> result = executor.submit(new CallTask());
	result.get(); //省略异常处理代码
    
    //方式二: FutureTask 作为任务被executor.execute执行
	FutureTask<String> futureTask = new FutureTask<String>(new CallTask());
	executor.execute(futureTask);
	futureTask.get(); //省略异常处理代码
}
static class CallTask implements Callable<String> {

	@Override
	public String call() throws Exception {
		return doWork();;
	}
}
```

​	Future.get() 方法的返回值是 Callable任务的 call() 方法返回的。

​	有一点需要注意，FutureTask 本身作为任务被 Executor 执行器执行时，FutureTask 就只是个Runnable，因此返回的Future是无法获取执行结果的。

```java
//错误示范: 不能获取异步执行结果
FutureTask<String> futureTask = new FutureTask<String>(new CallTask());
Future<?> result = executorService.submit(futureTask);
result.get();
```

## 9.5 小结

​	在Executor 框架的整个设计中，任务执行器有自己的生命周期，任务本身也有自己的生命周期。



# 10. 任务执行器-线程池

​	在Java应用中，使用线程来异步执行任务。线程的创建和销毁需要一定的开销，如果我们为每一个任务创建一个新的线程来执行，这些线程的创建和销毁将会消耗大量的计算资源。

​	线程池是一种"池化"思想，即将宝贵的资源（线程）放到池子里，反复使用，减少开销，提升效率。



## 10.1 ThreadPoolExecutor

<img src="https://gitee.com/Jackpotsss/pic_go/raw/master/img/image-20200403094205684.png" alt="image-20200403094205684" style="zoom: 80%;" />

​	ThreadPoolExecutor 是 Java 线程池的核心实现类。

​	Executor 是顶层接口，只提供一个方法 `void execute(Runnable command)` ，该接口设计的目的是将任务的提交和执行进行解耦；

​	ExecutorService 接口增加了一些能力：1. 增强执行任务的能力，可以为一个或多个异步任务生成执行结果 Future 并返回；2. 提供了管控线程池的方法，如停止线程池的shutdown() 和 shutdownnow()。

​	AbstractExecutorService 抽象类则对 ExecutorService 接口的 submit() 和 invoke() 方法提供了默认实现；

​	ThreadPoolExecutor 则实现了最复杂的运行部分，包括线程和任务的管理，运行时各种参数的变化等；

### 设计思路

<img src="https://gitee.com/Jackpotsss/pic_go/raw/master/img/image-20200403113248635.png" alt="image-20200403113248635" style="zoom: 25%;" />

ThreadPoolExecutor 执行execute 方法分四种情况：

1. 当前运行的线程数小于核心线程数【corePoolSize】，创建新的线程执行任务；
2. 当前运行的线程数大于等于核心线程数【corePoolSize】，则将任务加入任务队列中；
3. 如果任务队列已满，则创建新的线程执行任务；
4. 如果创建新线程后使当前线程数大于最大线程数【maximumPoolSize】，任务将被拒绝。



```java
    public void execute(Runnable command) {
        if (command == null)  throw new NullPointerException();        
		int c = ctl.get();
        if (workerCountOf(c) < corePoolSize) {	//1.小于corePoolSize,创建新线程执行任务
            if (addWorker(command, true))  return;
            c = ctl.get();
        }
        if (isRunning(c) && workQueue.offer(command)) {//2.将任务加入任务队列中
            int recheck = ctl.get();
            if (! isRunning(recheck) && remove(command)){
                reject(command);
            }
            else if (workerCountOf(recheck) == 0){
                addWorker(null, false);
            }
        }
        else if (!addWorker(command, false))//3.任务队列已满,增加线程数量
            reject(command);		//4.线程数量饱和,触发拒绝策略
    }
```



### 创建线程池

使用 ThreadPoolExecutor 的构造器来创建一个线程池：

```java
new ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,
    TimeUnit unit,BlockingQueue<Runnable> workQueue,ThreadFactory threadFactory,
    RejectedExecutionHandler handler)
```

几个重要的参数：

**corePoolSize:**

​	核心线程数，当前线程数量少于核心线程数时，则会创建新的线程来执行新提交的任务，即便是当前有空闲线程也会创建，直到到达核心线程数。

**maximumPoolSize：**

​	线程池中的最大线程数。

**keepAliveTime：** 

​	当前线程数大于核心线程数时，多余的空闲线程在终止之前等待新任务的最长时间。

**BlockingQueue 阻塞队列：** 

​	当达到核心线程数后，将不再创建新的线程，而是将任务加到阻塞队列中，等待被执行。常用阻塞队列如下：

- ArrayBlockingQueue

- LinkedBlockingQueue
- PriorityBlockingQueue 



**threadFactory 线程工厂：** 

​	创建线程的工厂。通过线程工厂给每个创建出来的线程设置更有意义的名字；

**RejectedExecutionHandler 拒绝策略：** 

​	当队列和线程池都满了，说明线程池处于饱和状态，此时新提交的任务将会由拒绝策略来处理。

ThreadPoolExecutor 提供了四种拒绝策略：

- AbortPolicy 		         直接抛异常（默认）；
- DiscardPolicy              不处理，忽略提交的任务；
- CallerRunsPolicy	     使用调用者的线程来执行任务；
- DiscardOldestPolicy   丢弃队列中等待时间最长的任务，并执行当前任务；

当然你也可以通过实现接口自定义拒绝策略。



### 提交任务

​	ThreadPoolExecutor 使用 `execute()` 或者 `submit()` 方法向线程池提交任务。

​	execute() 方法用于提交无返回值的任务，无法判断任务是否执行成功，是否执行完毕。

​	submit() 方法用于提交有返回值的任务，返回一个Future<T> 类型的对象，通过future的get 方法来获取返回值，get 方法会阻塞当前线程直到拿到返回结果，当然也可以使用 `get(long timeout, TimeUnit unit)` 超时获取异步任务的返回结果，这时候任务有可能还没有完成。



### 关闭线程池

​	通过调用线程池的 `shutdown` 和 `shutdownnow` 来关闭线程池。

​	原理是遍历线程池中的工作线程，然后逐个调用线程的 interrupt 方法来中断线程，所以**无法响应中断的任务可能永远无法终止**。

​	`shutdown` 和 `shutdownnow` 的区别在于，调用 `shutdown` 使线程池进入SHUTDOWN 状态，调用 `shutdownnow` 使线程池进入STOP  状态。

| 运行状态 | 描述                                                         |
| -------- | ------------------------------------------------------------ |
| SHUTDOWN | 不再接受新提交的任务，但可以正常处理正在进行的任务和队列中的任务； |
| STOP     | 不再接受新提交的任务，也不能处理队列中的任务，并且中断正在进行的任务； |



### 合理配置线程池

​	给定一个单核处理器的服务器，在单核情况下，CPU可以通过快速切换任务实现多任务并发处理，但是同一时刻，一个核心只能处理一个任务，也就是只有一个线程可以获取到CPU时间片，执行任务；如果配置多个线程处理多个任务，那么整体的执行效率反而会下降，因为线程上下文切换需要开销。

​	所以理论上，给定一个N个核心的处理器，为了能够发挥出处理器的最大计算能力，同时又不过度频繁的上下文切换，应该配置N个线程来执行任务，这样在同一时刻，才有可能最大程度利用CPU计算能力。

​	程序运行在内存中，任务的执行不仅需要CPU，也需要磁盘和网络的参与，也就是说IO操作也需要考虑进去，无论是磁盘IO还是网络IO，相比CPU和内存的速度都是慢的很多，所以如果一个任务在执行过程中，有IO等待的情况，那么此时处理器的时间片是空闲的，并没有利用起来，如果IO需要等待10ms，那么这10ms 的时间CPU可以处理成百上千个任务了，所以通常配置的线程数量比CPU的核心数要多一点，具体多出多少呢，这要根据任务的具体类型来分析。



要想合理配置线程池，就必须先分析任务的特性，可以从以下几个角度来分析：

- 任务的性质：CPU密集型任务、IO密集型任务、混合任务；
- 任务的优先级：高、中、低；
- 任务执行时间：长、中、短；
- 任务的依赖性：是否依赖其他系统资源，如数据库连接等；



​	针对性质不同的任务，要使用不同的策略来配置线程池。

​	CPU密集型任务应配置尽可能小的线程数量，如**配置 N + 1个线程**的线程池；IO密集型任务的线程并不是一直在执行任务，则应该配置尽可能多的线程数量，如**配置 2 * N 个线程**。

​	如果一个线程池中混合了长任务和短任务，那么这个线程池其实是很难进行调优的，最好的办法就是**创建两个线程池**，一个服务于长任务，一个服务于短任务。

​	优先级不同的任务可以使用 PriorityBlockingQueue 来处理，它可以让优先级高的任务先执行。但要注意，如果一直有优先级高的任务提交到队列，那么优先级低的任务可能永远不会执行。



### 监控线程池



**配置参数**

| 参数                     | 描述                                                         |
| ------------------------ | ------------------------------------------------------------ |
| corePoolSizer            | 核心线程数量                                                 |
| maximumPoolSize          | 最大线程数量                                                 |
| workQueueSize            | 任务队列容量(有界队列/无界队列)                              |
| RejectedExecutionHandler | 拒绝策略(默认直接抛出异常)<br />线程池提供了四种拒绝策略,当然也可以自定义拒绝策略; |
| keepAliveTime            | 空闲线程等待工作的超时时间(单位:纳秒)                        |
| allowCoreThreadTimeOut   | 是否允许核心线程超时(默认false)                              |

注：

​	这些配置参数既可以在初始化线程池的时候设置（即在 ThreadPoolExecutor 构造方法中设置），也可以在线程池运行时动态设置（通过相关set方法）。

​	任务队列的容量直接影响了线程池的行为，比如配置一个无界队列，那么 maximumPoolSize 参数就是没有意义的，在任务很多的情况下，线程数量一直等于核心线程数量，不会达到最大线程数量，也就不会触发拒绝策略；



**运行参数**

| 参数               | 描述                     |
| ------------------ | ------------------------ |
| runState           | 运行状态                 |
| workerCount        | 线程池中线程数量         |
| active threads     | 正在执行任务的线程数量   |
| queued tasks       | 任务队列中的任务数量     |
| completedTaskCount | 已完成任务数量           |
| largestPoolSize    | 曾经线程池内的最大线程数 |

注：

​	线程池的这些运行参数都有相关的API可以获得，另外 ThreadPoolExecutor.toString() 方法也可以直接打印出来。 

**扩展线程池**

​	通过拓展线程池进行监控，继承线程池来自定义线程池，重写 beforeExecute() 、afterExecute() 、terminated() 方法，可以在任务执行前，执行后和线程池关闭的时机进行监控。例如记录任务的执行开始时间，结束时间，执行结果等信息；

​	上面三个方法是空实现的钩子函数，专门用来继承重写的。

```java
protected void beforeExecute(Thread t, Runnable r) { }
protected void afterExecute(Runnable r, Throwable t) { }
protected void terminated() { }
```



### 线程池的生命周期

​	线程池的生命周期设计的很简单，运行状态只是**单向**的流转变化：

| 运行状态   | 描述                                                         |
| ---------- | ------------------------------------------------------------ |
| RUNNING    | 能接受新提交的任务，也可以处理阻塞队列中的任务；             |
| SHUTDOWN   | 关闭状态，不能接受新提交的任务，但可以处理队列中的任务；     |
| STOP       | 不能接受新任务，也不能处理队列中的任务，并且中断正在进行的任务； |
| TIDYING    | 所有任务都已终止，线程数为0；                                |
| TERMINATED | 调用terminated() 进入该状态;                                 |



<img src="https://gitee.com/Jackpotsss/pic_go/raw/master/img/image-20200405195833538.png" alt="image-20200405195833538" style="zoom:80%;" />



### 实现细节

**运行状态**

​	线程池内部使用一个 `ctl` 变量维护两部分信息：线程池的运行状态 (runState) 和线程池内有效线程的数量 (workerCount) ，高3位保存runState，低29位保存 workerCount。

```java
    private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));

    private static final int COUNT_BITS = Integer.SIZE - 3;		//低29位用来计数
    private static final int CAPACITY   = (1 << COUNT_BITS) - 1;	//可以计数约5.3亿个线程

    // runState 存储在高3位
    private static final int RUNNING    = -1 << COUNT_BITS;
    private static final int SHUTDOWN   =  0 << COUNT_BITS;
    private static final int STOP       =  1 << COUNT_BITS;
    private static final int TIDYING    =  2 << COUNT_BITS;
    private static final int TERMINATED =  3 << COUNT_BITS;

    // Packing and unpacking ctl
    private static int runStateOf(int c)     { return c & ~CAPACITY; }
    private static int workerCountOf(int c)  { return c & CAPACITY; }
    private static int ctlOf(int rs, int wc) { return rs | wc; }
```

运行状态和线程数量的获取都是通过**位运算**来实现的。



**阻塞队列阻塞线程**

​	最开始接触线程池的时候，我很好奇，线程池是怎么让线程空闲的情况下还不销毁？答案是依靠阻塞队列的性质使线程处于阻塞等待状态。

​	创建一个新的工作线程，加入到线程集合，然后开始执行任务，首先执行第一个任务，当任务执行完毕之后，线程就去工作队列中取任务，如果取到任务就执行任务，如果阻塞队列为空，那么线程就阻塞住，一旦有新任务提交，任务队列不为空，则唤醒线程继续执行新任务。

相关源码如下：

**Worker**

​	Worker 是 ThreadPoolExecutor 的私有内部类，构造一个 Worker 需要传入第一个要执行的任务，每构造一个 Worker 就创建一个新的工作线程来执行任务；Worker 同时也是 Runnable，将 firstTask 的执行代理到自己的run 方法上；Worker 同时也是任务执行时的同步器 AQS，Worker 类设计的相当紧凑。

```java
private final class Worker 
    extends AbstractQueuedSynchronizer implements Runnable{
    
    final Thread thread;
    Runnable firstTask;
    
    Worker(Runnable firstTask) {
    	setState(-1); // inhibit interrupts until runWorker
    	this.firstTask = firstTask;
    	this.thread = getThreadFactory().newThread(this);
    }
    
    public void run() {
       runWorker(this);
    }
}
```

​	这里有个小细节，Worker 作为同步器，同步状态的初始值为 -1，这其实是为了后面线程池 shutdownNow 准备的，shutdown 是通过遍历 工作线程集合，中断所有符合条件的线程来实现的，shutdownNow 不会中断新加入到线程集合 works并且还没有开始执行任务的线程。

​	运行任务之前需要先更新一下同步状态；

```java
void interruptIfStarted() {
    Thread t;
    if (getState() >= 0 && (t = thread) != null && !t.isInterrupted()) {
       try {
           t.interrupt();
       } catch (SecurityException ignore) {
       }
    }
}
```



**addWorker**

​	创建一个新的工作线程并加入到线程集合中，运行该线程。

```java
//截取部分核心代码:
private boolean addWorker(Runnable firstTask, boolean core) {
    final ReentrantLock mainLock = this.mainLock;
   	Worker w = new Worker(firstTask);	//1.创建一个新的工作线程
    final Thread t = w.thread;
    
    mainLock.lock();
    try {
       workers.add(w); 	//2.存储工作线程的集合是HashSet(线程不安全),对该集合的访问需要加锁;
    } finally {
        mainLock.unlock();
    }
    t.start();			//3.开始执行任务
}
```



**runWorker**

​	Worker 类的 run 方法调用 runWorker 方法，线程执行任务的具体细节，被创建出来的新线程执行完第一个任务 firstTask，接着去任务队列中获取任务，如果队列为空，则会阻塞线程进入等待状态，直到队列不为空，唤醒线程继续执行新的任务。

```java
final void runWorker(Worker w) {
    Thread wt = Thread.currentThread();
    Runnable task = w.firstTask;
    w.firstTask = null;
    w.unlock(); // allow interrupts

   	while (task != null || (task = getTask()) != null) { //获取任务;没有任务将被阻塞;
        w.lock();
        if ((runStateAtLeast(ctl.get(), STOP) ||
             (Thread.interrupted() && runStateAtLeast(ctl.get(), STOP))) &&
            !wt.isInterrupted()){
            wt.interrupt();
        }
        try {
            beforeExecute(wt, task);
            Throwable thrown = null;
            try {
                task.run();		//执行任务
            } catch (RuntimeException x) {
                thrown = x; throw x;
            } finally {
                afterExecute(task, thrown);
            }
        } finally {
            task = null;
            w.completedTasks++;
            w.unlock();
        }
    }
}
```

​	task.run() 为执行任务，在执行任务前后有 beforeExecute 和 afterExecute 两个钩子函数，他们是空实现，子类可以复写这两个方法加入自定义逻辑；

​	可以看到工作线程work被阻塞空闲时的同步状态为 0，执行任务时的同步状态为1，所以想要获取当前正在执行任务的线程数量 getActiveCount() ，只要查看works 集合中有多少处于加锁状态的 Work 就可以了。

```java
    public int getActiveCount() {
        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();
        try {
            int n = 0;
            for (Worker w : workers)
                if (w.isLocked())	//判断加锁状态
                    ++n;
            return n;
        } finally {
            mainLock.unlock();
        }
    }
```



## 10.2 ScheduledThreadPoolExecutor

<img src="https://gitee.com/Jackpotsss/pic_go/raw/master/img/image-20200404101523868.png" alt="image-20200404101523868" style="zoom: 80%;" />



### 接口设计

ScheduledExecutorService 接口

```java
public interface ScheduledExecutorService extends ExecutorService {
    
   	public ScheduledFuture<?> schedule(Runnable command,long delay, TimeUnit unit);

	public ScheduledFuture<V> schedule(Callable<V> callable,long delay, TimeUnit unit);

	public ScheduledFuture<?> scheduleAtFixedRate(Runnable command,long initialDelay,
                                                  long period, TimeUnit unit);

	public ScheduledFuture<?> scheduleWithFixedDelay(Runnable command,long initialDelay,
                                                     long delay,TimeUnit unit); 
}
```

​	ScheduledExecutorService 接口方法的返回值类型均是 ScheduledFuture，这是个复合接口，除了继承Future接口，用来管理定时任务的生命周期，还继承Delayed 定时接口，用来获取关联对象的剩余延迟时间。

```java
//复合接口 ScheduledFuture
public interface ScheduledFuture<V> extends Delayed, Future<V> {
}
//定时接口
public interface Delayed extends Comparable<Delayed> {
    long getDelay(TimeUnit unit);
}
```

注：

​	scheduleAtFixedRate 和 scheduleWithFixedDelay 单凭方法名有些不太好区分，二者的区别在于是否受任务执行时间的影响。scheduleAtFixedRate（）是以固定的频率周期性地执行，不受任务执行时间的影响；scheduleWithFixedDelay（）也是周期性的执行，只不过受任务执行时间的影响，在任务执行完之后延期指定时间再次执行。

​	scheduleAtFixedRate（）和scheduleWithFixedDelay（）用于运行周期任务，默认情况下，如果在run（）方法中有异常发生，下一次周期任务将不会执行；如果想一直执行下去，try-catch 捕获异常即可；



### 实现



<img src="https://gitee.com/Jackpotsss/pic_go/raw/master/img/image-20200406163243418.png" alt="image-20200406163243418" style="zoom:33%;" />



​	ScheduledThreadPoolExecutor 为了实现定时、周期性地执行任务，对ThreadPoolExecutor 做了一些改动：

- 使用延迟队列作为工作队列；
- 获取任务的方式不同；
- 执行完任务后增加了额外的处理；



#### ScheduledFutureTask

​	定时任务对象，ScheduledThreadPoolExecutor 的私有内部类，内部维护三个重要变量：

```java
private final long sequenceNumber;	//加入队列的序列号
private long time;					//执行时间(单位:纳秒)
private final long period;			//执行周期(单位:纳秒)
```



#### DelayedWorkQueue

​	**延迟工作队列**，ScheduledThreadPoolExecutor 的静态内部类，内部使用**优先队列**存储定时任务对象，并且使用 Lock + Condition 保证线程安全，最先执行的任务放在头部。



### 实际应用

​	下面是一个定时任务功能，后台易拓展，前台可配置，可以说是功能比较齐全的计划任务了：

<img src="https://gitee.com/Jackpotsss/pic_go/raw/master/img/image-20200424093342450.png" alt="image-20200424093342450" style="zoom: 80%;" />

<img src="https://gitee.com/Jackpotsss/pic_go/raw/master/img/image-20200424093514857.png" alt="image-20200424093514857" style="zoom:80%;" />



## 10.3 ForkJoinPool

<img src="https://gitee.com/Jackpotsss/pic_go/raw/master/img/image-20200404202724107.png" alt="image-20200404202724107" style="zoom:80%;" />

​		JDK1.7版本加入的并行处理线程池。ForkJoinPool 提供了一个Executor，主要用于处理 ForkJoinTask 及其子类的实例。ForkJoinTask 通常是计算密集型任务，使用并行任务处理可以提高该类型任务的吞吐量，最大程度的利用CPU计算资源。



## 10.4 Executors 工具

Executors 工具提供了快速创建线程池的静态方法：

### 单线程的线程池

​	核心线程数量和最大线程数量都为1；

```java
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,0L, TimeUnit.MILLISECONDS,new LinkedBlockingQueue<Runnable>()));
}
```



### 固定线程数量的线程池

​	核心线程数量和最大线程数量相同，说明线程池内的线程数量是固定的，线程数量饱和之后，空闲的时候线程数量是那些，任务非常多的时候线程数量也是那些。

```java
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads, 0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
}
```

### 缓存线程池 CachedThreadPool

​	核心线程数量设置为0，最大线程数量为 `Integer.MAX_VALUE`，所有被创建出来的线程的空闲时间不会超过60秒，所以长时间处于空闲的缓存线程池内没有线程，不会占用任何资源。

```java
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
}
```

​	**SynchronousQueue 是一个没有容量的阻塞队列**，即添加任务时offer（），如果没有空闲线程在阻塞获取任务poll（），那么就会添加失败，则会创建新线程来执行任务；如果添加任务时有空闲线程在阻塞获取任务，那么就会添加成功，复用原线程执行任务；

​	也就是说，如果用户提交任务的速度大于线程池处理任务的速度，那么 CachedThreadPool 将会不断创建新线程，极端情况下，会因为创建线程过多而耗尽CPU和内存资源。



<img src="https://gitee.com/Jackpotsss/pic_go/raw/master/img/image-20200406162636159.png" alt="image-20200406162636159" style="zoom: 33%;" />

### 不要使用Executors 工具

之前我看过一篇博客，是这么写的：

 	`首先，我们的线程池类型在 JDK1.8 版本之前一共有 4 种，分别是 newSingleThreadPool、newFixedThreadPool、newCachedThreadPool、newScheduledThreadPool 四种，在 JDK1.8 又加入了一种：newWorkStealingPool，所以现在一共是 5 种。`

​	上面这种说法，对线程池的认识并不到位，因为 Executors 可以直接创建线程池，就以 Executors 创建出来的几种线程池作为分类，这很片面，理解的也很表面。

​	看到这里，我觉得《阿里巴巴Java开发手册》里面对于线程池使用的强制要求是合理的：

**【强制】线程池不允许使用 Executors 创建，而是通过 ThreadPoolExecutor 的方式创建，这样的处理方式能让编写代码的工程师更加明确线程池的运行规则，规避资源耗尽的风险。**

说明：

​	Executors 返回的线程池对象弊端如下：

1. SingleThreadPool 和 FixedThreadPool ，允许工作队列的长度为 Integer.MAX_VALUE，可能会堆积大量的请求，从而导致OOM；
2. CachedThreadPool 和 ScheduledThreadPool，允许创建线程的数量为 Integer.MAX_VALUE，可能会创建大量的线程，从而导致OOM；



​	若非要对线程池进行一个分类，在JDK1.8 中可以分为 **普通线程池、定时任务线程池和分支并行线程池**三种。



# 11. 同步工具类

​	Java并发包提供了一些现成的同步工具类供开发人员使用，这些工具类设计的非常巧妙，而且很实用，在对应的业务场景中可以考虑使用它们。

- Semaphore  信号量
- CountDownLatch   闭锁
- CyclicBarrier		同步屏障
- Exchanger   线程间交换数据

## 11.1 Semaphore  信号量

​	Semaphore  是一个可以自定义同步数量的同步工具，不支持重进入，支持公平和非公平锁。Semaphore  与重入锁 ReentrantLock  的设计方式一样，内部基于 AQS 构建同步器，使用 volatile + 循环CAS的方式更新同步状态。

### 许可证 

​	Semaphore  引入许可证的概念，许可证的数量就是同步状态的值，只不过信号量中许可证的更新动作和同步锁中同步状态的更新动作正好相反，在AQS构建的同步锁设计中，同步状态初始值为0，获取同步锁，就会对同步状态加1，释放同步锁，就对同步状态减1；

​	Semaphore  同样是AQS构建出来的同步工具，构造一个Semaphore  需要先指定初始许可证的初始数量，获取一个许可证，许可证数量（同步状态）减1，释放一个许可证，许可证数量（同步状态）加1；



```java
public class Semaphore {

    public Semaphore(int permits) {
        sync = new NonfairSync(permits);
    }
    
   	public Semaphore(int permits, boolean fair) {
        sync = fair ? new FairSync(permits) : new NonfairSync(permits);
    }
}
```

注：

​	Semaphore 释放和获取许可证是不支持线程重进入的，也就是说同一个线程连续获取N个许可证，Semaphore 就会减少N个许可证。

​	许可证的初始数量可以为负数，但这种情况下，必须先释放许可证，使许可证的数量大于零，Semaphore 才能正常使用。

​	通常情况下，获取许可证和释放许可证是成对出现的，并且获取许可通常在释放许可之前发生。但Semaphore并没有这么强制设计，理论上，你可以先释放许可证，而且没有次数限制，释放一个就会多一个许可证，释放十个就会多十个许可证。

### 使用场景

**Semaphore 通常用于限制访问资源的并发线程数，控制任务处理速度。** 

```java
class Pool {

   private final Semaphore available = new Semaphore(10, false);

   public Object getItem() throws InterruptedException {
     available.acquire();
     return getNextItem();
   }

   public void putItem(Object x) {
     if (markAsUnused(x))
       available.release();
   }

   protected synchronized Object getNextItem() {
     return doWork(); 
   }

   protected synchronized boolean markAsUnused(Object item) {
     return doWork();
   }
 }
```



## 11.2 CountDownLatch 闭锁

​	CountDownLatch 是一种灵活的闭锁同步工具，可以延迟线程的进度直到达到开锁状态。

​	闭锁包含一个计数器，CountDownLatch 初始化一个正整数作为计数器的值，表示要等待的事件数量。await() 方法会阻塞线程，直到计数器归零（又或者是线程中断或超时），countDown() 方法会使计数器的数量递减一，并且计数器只能减，不能加或重置，因此闭锁是一次性工具。

**使用场景**

闭锁用于确保某些任务直到其他任务都执行完毕才继续执行。

```java
//并发执行任务的测试程序:
public class ConcurrentExec {
	private int count;
    
	public ConcurrentExec(int count) {
		this.count = count;
	}
	public long execute(Runnable task) {
        final CountDownLatch startLatch = new CountDownLatch(1);
		final CountDownLatch endLatch = new CountDownLatch(count);
        
		for (int i=0 ;i< count;i++) {
			new Thread(new Runnable() {
				
				@Override
				public void run() {
					try {
						startLatch.await();
						try {
							task.run();
						} finally {
							endLatch.countDown();
						}
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
				}
			}).start();
		}
		long startTime = System.nanoTime();
		startLatch.countDown();
		endLatch.await();
		long endTime = System.nanoTime();
		return endTime-startTime;
	}
}
```

​	上面的例子，是一个并发执行任务的测试程序，N 个线程同时执行一个任务，可以统计总耗时。只使用一个CountDownLatch 就可以让多个线程并发执行任务，加入第二个CountDownLatch 是为了等待所有的线程执行完任务再统计时间。

注：CountDownLatch 是一次性工具，无法重置计数。如果需要重置计数，可以考虑使用 CyclicBarrier。



## 11.3 CyclicBarrier 同步屏障

​	并发包还提供了一个可以反复使用的同步工具 CyclicBarrier 同步屏障，在某些场景下CyclicBarrier 和CountDownLatch 确实可以互相替换，但二者又有区别。

​	CyclicBarrier 有一个内置计数器，初始化CyclicBarrier 的同时给计数器设置初始值，调用 await() 使计数器递减一，然后检查计数器是否归零，如果不为零，阻塞当前线程。如果计数器归零，CyclicBarrier 会唤醒所有等待在同步屏障上的线程，所有线程并发执行任务。

​	计数器归零后，CyclicBarrier 自动重置计数器，以供反复使用。

```java
//部分源码如下(已简化)
public class CyclicBarrier {
    private final ReentrantLock lock = new ReentrantLock();
    private final Condition trip = lock.newCondition();
    private final int parties;
    private final Runnable barrierCommand;
    
    public int await() throws InterruptedException, BrokenBarrierException {
        
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            if (Thread.interrupted()) {//响应中断
                nextGeneration();
                throw new InterruptedException();
            }
            int index = --count;// 递减一
            if (index == 0) {  // 计数器归零
                try {
                    if (barrierCommand != null) barrierCommand.run();//屏障任务执行
                    nextGeneration();	//唤醒所有等待的线程,并重置计数器
                    return 0;
                } finally {
                    nextGeneration();
                }
            } 
        } finally {
            lock.unlock();
        }
    }
    //唤醒所有等待的线程,并重置计数器
    private void nextGeneration() {
        trip.signalAll();
        count = parties;
    }
}
```



上面 ConcurrentExec 的例子使用CyclicBarrier 同样可以实现：

```java
//并发执行任务的测试程序:
public class ConcurrentExec {
	private int count;
    
	public ConcurrentExec(int count) {
		this.count = count;
	}
	public long execute(Runnable task) {
        final CyclicBarrier barrier = new CyclicBarrier(count);
		final CountDownLatch endLatch = new CountDownLatch(count);
        
		for (int i=0 ;i< count;i++) {
			new Thread(new Runnable() {
				
				@Override
				public void run() {
					try {
						barrier.await();
						try {
							task.run();
						} finally {
							endLatch.countDown();
						}
					} catch (InterruptedException | BrokenBarrierException e) {
						e.printStackTrace();
					}
				}
			}).start();
		}
		long startTime = System.nanoTime();
		endLatch.await();
		long endTime = System.nanoTime();
		return endTime-startTime;
	}
}
```



此外CyclicBarrier 还有两个功能:

1.  支持超时等待的 `await(timeout,unit)` 方法;
2.  初始化CyclicBarrier 时可以指定一个任务 `CyclicBarrier(int parties, Runnable barrierAction)` ，计数器归零后执行;

#### 闭锁和同步屏障 

​	**闭锁和同步屏障的关键区别在于，所有线程必须都到达屏障位置，才能继续执行；而闭锁是所有线程都触发某个事件，才能在闭锁位置继续执行。屏障用于等待其他线程，闭锁用于等待事件。** 

​	这就好比一道关闭的高科技门，闭锁的门和门锁不在同一个位置，而且门和门锁之间有一段距离，每个人必须先去门锁这将右手放到指纹识别器上，然后再来到闭锁的门这等待门开启，假如闭锁的门必须收集齐5个人的指纹，才能开启，那么这5个人必须都去指纹识别，当4个人完成指纹识别并在门下等待的时候，第5个人完成指纹识别后，闭锁的门会立即打开，这4个人是不会等待第五个人到达门这才走的，而是等到指纹识别全部完成后，立即就走了。（是不是和盗墓笔记里的墓门机关很像，最后一个人总是被留在门外）

​	屏障的门和门锁是一体的，且是红外识别的门，人只要到达门前，门就会自动识别，同样假如屏障的门必须收集齐5个人的红外信息才能开启，那么当第五个人到达的时候，门就会开启，此时5个人可以同时进入门内。这里等待的是人，而不是开锁的事件。



## 11.4 Exchanger   交换数据

​	Exchanger 是用于**线程间交换数据**的同步工具类。它提供一个同步交换点，只有两个线程同时到达交换点，才能交换数据。若只有一个线程到达交换点，则进入阻塞等待状态，直到其他线程到达这个同步交换点。

```java
public class ExchangerDemo {

	private static final Exchanger<String> exchanger = new Exchanger<String>();
	
	public static void main(String[] args) {
		
		new Thread(new Runnable() {	
			@Override
			public void run() {
				try {
					String resultA = "result A";
					//数据交换:把数据传递给其他线程,同时获取其他线程的数据
					String resultB = exchanger.exchange(resultA);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		},"thread-1").start();
		
		new Thread(new Runnable() {
			@Override
			public void run() {
				try {
					String resultB = "result B";
					//数据交换
					String resultA = exchanger.exchange(resultB);
					boolean isEquals = resultB.equals(resultA);//校验两个执行结果是否相等
					
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		},"thread-2").start();
	}
}
```



# 12. Jstack 工具

JDK自带的查询Java线程信息的命令工具

```shell
[root@mikai-CS ~]# jstack 22155
2020-04-11 22:23:57
Full thread dump Java HotSpot(TM) 64-Bit Server VM (24.80-b11 mixed mode):

"dna-2522" daemon prio=10 tid=0x00007f4c7c585800 nid=0x70a5 in Object.wait() [0x00007f4cd6c16000]
   java.lang.Thread.State: TIMED_WAITING (on object monitor)
	at java.lang.Object.wait(Native Method)
	at com.jiuqi.dna.core.spi.work.WorkingManager.getWorkToDo(WorkingManager.java:412)
	- locked <0x00000007a04857e8> (a com.jiuqi.dna.core.spi.work.WorkingThread)
	at com.jiuqi.dna.core.spi.work.WorkingThread.run(WorkingThread.java:42)
	
"[ThreadPool Manager] - Idle Thread" daemon prio=10 tid=0x00007f4c8c005000 nid=0x56fd in Object.wait() [0x00007f4cdfa6b000]
   java.lang.Thread.State: WAITING (on object monitor)
	at java.lang.Object.wait(Native Method)
	at org.eclipse.equinox.internal.tpt.threadpool.Executor.run(Executor.java:106)
	- locked <0x0000000710854420> (a org.eclipse.equinox.util.tpt.threadpool.Executor)
	
"dna-4 Acceptor0 SelectChannelConnector@0.0.0.0:9412" daemon prio=10 tid=0x00007f4c84b2b000 nid=0x56af waiting for monitor entry [0x00007f4ce46be000]
   java.lang.Thread.State: BLOCKED (on object monitor)
	at sun.nio.ch.ServerSocketChannelImpl.accept(ServerSocketChannelImpl.java:234)
	- waiting to lock <0x000000071089f080> (a java.lang.Object)
	at org.eclipse.jetty.server.nio.accept(SelectChannelConnector.java:109)
	at com.jiuqi.dna.core.spi.work.WorkingThread.run(WorkingThread.java:44)

"GC task thread#0 (ParallelGC)" prio=10 tid=0x00007f4cf8022000 nid=0x568e runnable

```

输出信息依次是

```
"dna-2522"	//线程名称
daemon		//代表守护线程
prio=10		//优先级为10

java.lang.Thread.State: TIMED_WAITING		//线程状态
- waiting to lock	//等待一个锁:也就是说目前线程处于阻塞状态;
- locked 			//已上锁:最近一行的栈帧就是加锁的位置;
```



# 13. 死锁

​	多个线程获取多个锁资源，才有可能产生死锁。多线程只去争夺一个锁资源是无法产生死锁的。

​	线程A 先获取 锁x，线程B获取锁y，在线程A和B都不释放锁资源的情况下，线程A尝试获取锁y， 被阻塞住，线程B尝试获取锁 x，被阻塞住；两个线程都持有对方想要的锁，又各自都不释放，从而产生死锁。此时若没有第三方的介入，线程A和B都不能继续工作。



## 13.1 原因

下面看两个发生死锁的示例

```java
public class DeadLockDemo {
	//例一: 简单的锁顺序死锁
	private static Object lock1 = new Object();
	private static Object lock2 = new Object();
    
	public static void run1() {
		synchronized (lock1) {
			synchronized (lock2) {
				doWork();
			}
		}
	}
	public static void run2() {
		synchronized (lock2) {
			synchronized (lock1) {
				doWork();
			}
		}
	}
    //例二: 动态的锁顺序死锁
	public void transferMoney(Account fromAccount,Account toAccount,double amount) {
		synchronized (fromAccount) {
			synchronized (toAccount) {
				fromAccount.debit(amount);
				toAccount.credit(amount);
			}
		}
	}
}
```

​	例一是一个简单的锁顺序死锁，当多线程同时调用run1() 和 run2() 方法时，很容易产生死锁。例二表面看起来所有进入同步代码块的线程加锁顺序是一样的，但其实加锁的顺序依赖方法的入参情况，如下面这种情况就有可能产生死锁：

- transferMoney(myAccount, yourAccount,10);
- transferMoney(yourAccount, myAccount, 20);



**jstack工具查看 Java 进程的死锁**

无论是 synchronized 监视器锁, 还是并发包中的Lock 锁, 如果发生死锁, 使用jstack工具查看Java 进程，都会在输出最后有死锁记录：

```shell
Found one Java-level deadlock:
=============================
"thread-2":
  waiting to lock monitor 0x0001021fc68 (object 0x00000007ab534b68, a java.lang.Object),
  which is held by "thread-1"
"thread-1":
  waiting to lock monitor 0x0001021d118 (object 0x00000007ab534b78, a java.lang.Object),
  which is held by "thread-2"

Java stack information for the threads listed above:
===================================================
"thread-2":
        at com.jys.jackpot.DeadLockTest.run2(DeadLockTest.java:28)
        - waiting to lock <0x00000007ab534b68> (a java.lang.Object)
        - locked <0x00000007ab534b78> (a java.lang.Object)
        at com.jys.jackpot.DeadLockTest$task2.run(DeadLockTest.java:47)
        at java.lang.Thread.run(Thread.java:745)
"thread-1":
        at com.jys.jackpot.DeadLockTest.run1(DeadLockTest.java:20)
        - waiting to lock <0x00000007ab534b78> (a java.lang.Object)
        - locked <0x00000007ab534b68> (a java.lang.Object)
        at com.jys.jackpot.DeadLockTest$task1.run(DeadLockTest.java:38)
        at java.lang.Thread.run(Thread.java:745)

Found 1 deadlock.
```



<img src="https://gitee.com/Jackpotsss/pic_go/raw/master/img/image-20200414093239882.png" alt="image-20200414093239882" style="zoom: 67%;" />



## 13.2 解决

处理死锁有三种主要方式：

1. 采用某种规范或协议来预防或避免死锁，确保系统将永远不会进入死锁状态；
2. 运行系统进入死锁状态，检测它，然后恢复；
3. 完全忽略死锁问题，并假设系统永远都不会出现死锁；



​	方式一，限制多线程加锁的顺序是非常有效的预防手段；方式二是MySQL服务的处理方式，mysql会立即处理死锁, 自动回滚持有锁最少的事务，并抛出异常；方式三是大多数操作采用的方式，包括Linux 和 WIndows。



## 13.3 预防死锁

​	死锁其实是一个比较棘手的问题，程序可能运行几万次都没有问题，但也许下一次就发生死锁了，而且这种错误常常是不容易复现的。

- 多线程加锁的顺序要保持一致！如果每个线程都是以相同的顺序加锁并以相反的顺序释放锁，那么程序是不可能发生死锁的。



# 14. 分布式锁

​	本文的所有内容，讲述的都是在Java单体应用内如何并发编程，即在一个进程内的多个线程同时操作共享数据。但在集群环境下（不同的进程同时操作共享资源），只靠单个应用内的同步手段或锁机制是无法保证数据安全的，在这种场景下，分布式锁可以解决这样的问题。

​	分布式锁的实现方式不止一种，质量和代价也不太一样，目前主流分布式锁的解决方案有如下几种：

- 基于 Redis 服务实现的分布式锁-**Redlock** 
- 基于 ZooKeeper 服务实现的分布式锁



# 15. 如何学习并发编程

​	编写出并发安全的程序不是一件容易的事情，具有不小的挑战性。我从一开始接触线程和并发的概念，到后来系统的学习如何并发编程，也积累了一些心得，主要有以下几点分享给大家：

- 并发包的源码必须读，而且是反复读，看接口、实现类、具体的实现逻辑，源码没必要全部都看，但核心实现细节要研究明白；
- 跑程序，反复调试，看多线程环境下程序是如何保证数据安全的；
- 官方提供的API文档和源码中的注释读清楚，都是官方总结的精华；
- 《Java并发编程实战》是Java并发编程领域写的非常好的一本书，书籍的作者就是并发包源码的作者（专家组），足够权威和深度。我本人并不是直接读的这本书，最初接触的是国内的方腾飞写的《Java并发编程的艺术》，这本书较上一本门槛稍低一些，更适合初学者学习吧，后来我读并发编程实战的内容，怎么评价呢？“没有一句多余的废话，知识点非常紧凑，系统，有深度”，所以如果初学者一开始就看这本书，可能很多地方都不容易理解，有可能打击自信心和兴趣。所以选书籍要根据个人情况决定，只要是感兴趣还能学到东西，那就没问题。
- 主动分享给其他人。并发这部分知识点很多，当你学了一小部分之后，就可以和别人探讨一下了，探讨的过程中，你就会发现自己还有很多理解不到位的地方，然后反过来自己再钻研一遍。分享的方式有很多，直接聊天，分享笔记等都可以。



# README

版权声明：

​	**自由转载-保持署名-可修改-非商业 （知识共享3.0许可证）** 

作者：

​	米开

微信公众号：《米开的网络私房菜》

<img src="https://gitee.com/Jackpotsss/pic_go/raw/master/img/QRcode.png" />

参考：

​	  并发包源码（ JDK1.7/1.8 ）

​	《Java并发编程的艺术》方腾飞 （本书围绕JDK7 进行研究的）

​	《Java并发编程实战》Doug Lea、Joshua Bloch等	2011年出版

​	  Java8 api 官方文档 [并发包部分](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/package-summary.html)

​	《深入理解计算机系统》第三版



米开	2020-03-01	第一次修订

米开	2020-03-26	修订	

米开	2020-04-19	修订（2万字）

米开	2020-07-20	修订（3万字）

米开	2020-07-25	第一版 0.95.0 定稿