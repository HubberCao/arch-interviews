## 线程

### 线程之间是如何通信的
```
1.通过线程之间共享变量的方式
(1) synchronized + Object的 wait(),notify(),以及notifyAll() 
(2) 使用lock.newCondition().await() 和 signal() 方法实现线程之间交互 
(3) 利用volatile
2. 通过队列(BlockingQueue)来实现线程的通信

用java.util.concurrent包中linkedBlockingQueue 来进行线程间交互； 
java.util.concurrent.LinkedBlockingQueue 是一个基于单向链表的、范围任意的（其实是有界的）、FIFO 阻塞队列。
访问与移除操作是在队头进行，添加操作是在队尾进行，并分别使用不同的锁进行保护，只有在可能涉及多个节点的操作才同时对两个锁进行加锁。

3. 套接字（Socket），不同的机器之间进行通信

java线程间通信：

1：线程上下文
2：共享内存
3：IPC通信
4：套接字（Socket），不同的机器之间进行通信

linux常用的进程间的通讯方式
（1）、管道(pipe)：管道可用于具有亲缘关系的进程间的通信，是一种半双工的方式，数据只能单向流动，允许一个进程和另一个与它有共同祖先的进程之间进行通信。
（2）、命名管道(named pipe)：命名管道克服了管道没有名字的限制，同时除了具有管道的功能外（也是半双工），它还允许无亲缘关系进程间的通信。命名管道在文件系统中有对应的文件名。命名管道通过命令mkfifo或系统调用mkfifo来创建。
（3）、信号（signal）：信号是比较复杂的通信方式，用于通知接收进程有某种事件发生了，除了进程间通信外，进程还可以发送信号给进程本身；linux除了支持Unix早期信号语义函数sigal外，还支持语义符合Posix.1标准的信号函数sigaction（实际上，该函数是基于BSD的，BSD为了实现可靠信号机制，又能够统一对外接口，用sigaction函数重新实现了signal函数）。
（4）、消息队列：消息队列是消息的链接表，包括Posix消息队列system V消息队列。有足够权限的进程可以向队列中添加消息，被赋予读权限的进程则可以读走队列中的消息。消息队列克服了信号承载信息量少，管道只能承载无格式字节流以及缓冲区大小受限等缺
（5）、共享内存：使得多个进程可以访问同一块内存空间，是最快的可用IPC形式。是针对其他通信机制运行效率较低而设计的。往往与其它通信机制，如信号量结合使用，来达到进程间的同步及互斥。
（6）、内存映射：内存映射允许任何多个进程间通信，每一个使用该机制的进程通过把一个共享的文件映射到自己的进程地址空间来实现它。
（7）、信号量（semaphore）：主要作为进程间以及同一进程不同线程之间的同步手段。
（8）、套接字（Socket）：更为一般的进程间通信机制，可用于不同机器之间的进程间通信。起初是由Unix系统的BSD分支开发出来的，但现在一般可以移植到其它类Unix系统上：Linux和System V的变种都支持套接字。
```

### 守护进程和用户进程
```
1. 主线程/守护线程/用户线程
	主线程
	main，但不是守护线程。

	守护线程
	是指在程序运行的时候在后台提供一种通用服务的线程。如gc。如果全部的User Thread已经撤离，Daemon 没有可服务的线程，JVM撤离

	非守护线程
	也叫用户线程，由用户创建。

	关系：
	主线程和守护线程一起销毁；

	主线程和非守护线程互不影响。	

2.用户线程和守护线程的适用场景
由两者的区别及dead时间点可知，守护线程不适合用于输入输出或计算等操作，因为用户线程执行完毕，程序就dead了，适用于辅助用户线程的场景，如JVM的垃圾回收，内存管理都是守护线程，还有就是在做数据库应用的时候，使用的数据库连接池，连接池本身也包含着很多后台线程，监听连接个数、超时时间、状态等。

3.创建守护线程
调用线程对象的方法setDaemon(true)，设置线程为守护线程。
(1)thread.setDaemon(true)必须在thread.start()之前设置。
(2)在Daemon线程中产生的新线程也是Daemon的。
(3)不是所有的应用都可以分配给Daemon线程来进行服务，比如读写操作或者计算逻辑。因为Daemon Thread还没来得及进行操作，虚拟机可能已经退出了
```

### 有了进程，为什么还要有线程？
```
线程产生的原因：
进程可以使多个程序能并发执行，以提高资源的利用率和系统的吞吐量；但是其具有一些缺点：

1、进程在同一时间只能干一件事
2、进程在执行的过程中如果阻塞，整个进程就会挂起，即使进程中有些工作不依赖于等待的资源，仍然不会执行。

因此，操作系统引入了比进程力度更小的线程，作为并发执行的基本单位，从而减少程序在并发执行时所付出的时空开销，提高并发性。和进程相比，线程的优势如下：

1、从资源上来讲：线程是一种非常"节俭"的多任务操作方式。在linux系统下，启动一个新的进程必须分配给它
独立的地址空间，建立众多的数据表来维护它的代码段、堆栈段和数据段，这是一种"昂贵"的多任务工作方式。
2、从切换效率上来讲：运行于一个进程中的多个线程，它们之间使用相同的地址空间，而且线程间彼此切换所
需时间也远远小于进程间切换所需要的时间。据统计，一个进程的开销大约是一个线程开销的30倍左右。（
3、从通信机制上来讲：线程间方便的通信机制。对不同进程来说，它们具有独立的数据空间，要进行数据的
传递只能通过进程间通信的方式进行，这种方式不仅费时，而且很不方便。线程则不然，由于同一进城下的线
程之间贡献数据空间，所以一个线程的数据可以直接为其他线程所用，这不仅快捷，而且方便。

除以上优点外，多线程程序作为一种多任务、并发的工作方式，还有如下优点：

1、使多CPU系统更加有效。操作系统会保证当线程数不大于CPU数目时，不同的线程运行于不同的CPU上。
2、改善程序结构。一个既长又复杂的进程可以考虑分为多个线程，成为几个独立或半独立的运行部分，这样
的程序才会利于理解和修改。
```
### 什么是线程？它与进程的区别？为什么要使用多线程？
```
1、什么是线程？

线程是指程序在执行过程中，能够执行程序代码的一个执行单元，在Java语言中，线程有四种状态：运行，就绪，挂起，结束。

2、线程与进程的区别？

进程是一段正在运行的程序，而线程有时也被称为轻量级进程，它是进程的执行单元，一个进程可以拥有多个线程，各个线程之间共享程序的内存空间，但是，各个线程拥有自己的栈空间。

3、为什么使用多线程？

（1）、使用多线程可以减少程序的响应时间。单线程如果遇到等待或阻塞，将会导致程序不响应鼠标键盘等操作，使用多线程可以解决此问题，增强程序的交互性。

（2）、与进程相比，线程的创建和切换开销更小，因为线程共享代码段、数据段等内存空间。

（3）、多核CPU，多核计算机本身就具有执行多线程的能力，如果使用单个线程，将无法重复利用计算资源，造成资源的巨大浪费。

（4）、多线程可以简化程序的结构，使程序便于维护，一个非常复杂的进程可以分为多个线程执行。
```

### Java为什么不推荐使用线程组？
```
线程组ThreadGroup对象中的stop，resume，suspend会导致安全问题，主要是死锁问题，已经被官方废弃，多以价值已经大不如以前。
线程组ThreadGroup不是线程安全的，在使用过程中不能及时获取安全的信息。
```
参考
- [避免使用线程组](https://blog.csdn.net/Deaht_Huimie/article/details/85013810)
- [Thread及ThreadGroup杂谈](https://www.cnblogs.com/yiwangzhibujian/p/6212104.html)

### Java中用到的线程调度算法

```
抢占式。一个线程用完CPU之后，操作系统会根据线程优先级、线程饥饿情况等数据算出一个总的优先级并分配下一个时间片给某个线程执行。

操作系统中可能会出现某条线程常常获取到VPU控制权的情况，为了让某些优先级比较低的线程也能获取到CPU控制权，可以使用Thread.sleep(0)手动触发一次操作系统分配时间片的操作，这也是平衡CPU控制权的一种操作。
```

### 同步方法和同步块

参考
- [JAVA中线程同步的方法（7种）汇总](https://cloud.tencent.com/developer/article/1095920)

### 如何控制某个方法允许并发访问线程的个数
```
1. semaphore
2. 阻塞队列
3. Hystrix
4. RateLimiter
...
```
参考
- [Java中如何限制方法访问的并发数](https://blog.csdn.net/manzhizhen/article/details/81413014)

### ThreadLocal
```
每个Thread 维护一个 ThreadLocalMap 映射表，这个映射表的 key 是 ThreadLocal实例本身，value 是真正需要存储的 Object。
ThreadLocal 本身并不存储值，它只是作为一个 key 来让线程从 ThreadLocalMap 获取 value。
值得注意的是图中的虚线，表示 ThreadLocalMap 是使用 ThreadLocal 的弱引用作为 Key 的，弱引用的对象在 GC 时会被回收。
ThreadLocalMap使用ThreadLocal的弱引用作为key，如果一个ThreadLocal没有外部强引用来引用它，那么系统 GC 的时候，
这个ThreadLocal势必会被回收，这样一来，ThreadLocalMap中就会出现key为null的Entry，就没有办法访问这些key为null的Entry的value，如果当前线程再迟迟不结束的话，这些key为null的Entry的value就会一直存在一条强引用链：Thread Ref -> Thread -> ThreaLocalMap -> Entry -> value永远无法回收，造成内存泄漏。
ThreadLocal里面使用了一个存在弱引用的map, map的类型是ThreadLocal.ThreadLocalMap. Map中的key为一个threadlocal实例。这个Map的确使用了弱引用，不过弱引用只是针对key。每个key都弱引用指向threadlocal。 当把threadlocal实例置为null以后，没有任何强引用指向threadlocal实例，所以threadlocal将会被gc回收。 

但是，我们的value却不能回收，而这块value永远不会被访问到了，所以存在着内存泄露。因为存在一条从current thread连接过来的强引用。
只有当前thread结束以后，current thread就不会存在栈中，强引用断开，Current Thread、Map value将全部被GC回收。
最好的做法是将调用threadlocal的remove方法，这也是等会后边要说的。使用static的ThreadLocal，延长了ThreadLocal的生命周期，可能导致内存泄漏。
分配使用了ThreadLocal又不再调用get(),set(),remove()方法，那么就会导致内存泄漏，因为这块内存一直存在
```

## ReentrantLock

### 是如何实现可重入性的？

```
为每个锁关联一个获取计数器和一个所有者线程,当计数值为0的时候,这个所就没有被任何线程只有.当线程请求一个未被持有的锁时,
JVM将记下锁的持有者,并且将获取计数值置为1,如果同一个线程再次获取这个锁,技术值将递增,退出一次同步代码块,计算值递减,
当计数值为0时,这个锁就被释放.ReentrantLock里面有实现
```
- [可重入锁ReentrantLock详解](https://www.iteye.com/blog/donald-draper-2360411)
- [Java可重入锁学习笔记](https://www.shiyanlou.com/questions/2460/)

### 原理

参考
- [Lock原理分析](https://blog.csdn.net/u011212394/article/details/82314489)
- [Lock的实现原理](https://www.cnblogs.com/shoshana-kong/p/10772679.html)
- [Lock实现原理](https://www.cnblogs.com/shoshana-kong/p/10772608.html)

## 并发工具
```
CountDownLatch：
  某线程需要等待多个线程执行完毕，再执行。实现多个线程共同等待，同时开始执行任务，不可重用。(此类似于CyclicBarrier，可重用)
  
CyclicBarrier：
  CyclicBarrier允许一组线程互相等待，直到到达某个公共屏障点。与CountDownLatch不同的是该barrier在释放等待线程后可以重用，所以称它为循环（Cyclic）的屏障

CountDownLatch、CyclicBarrier差别：CountDownLatch和CyclicBarrier都能够实现线程之间的等待，只不过它们侧重点不同： 
CountDownLatch一般用于某个线程A等待若干个其他线程执行完任务之后，它才执行； 
而CyclicBarrier一般用于一组线程互相等待至某个状态，然后这一组线程再同时执行；
CountDownLatch是不能够重用的，而CyclicBarrier是可以重用的。

Semaphore：
  Semaphore是一个计数信号量，它的本质是一个”共享锁”。信号量维护了一个信号量许可集。线程可以通过调用acquire()来获取信号量的许可；
当信号量中有可用的许可时，线程能获取该许可；否则线程必须等待，直到有可用的许可为止。 线程可以通过release()来释放它所持有的信号量许可。
Semaphore其实和锁有点类似，它一般用于控制对某组资源的访问权限。

Exchanger：
  用于进行线程间的数据交换。两个线程通过exchange方法交换数据该工具类的线程对象是成对的
```
