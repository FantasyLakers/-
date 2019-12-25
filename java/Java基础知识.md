# Java知识点

## 1、元注解
- @Rentention：保留策略  
	- @Rentention(RententionPolicy.SOURCE)：编译器要丢弃的注释  
	- @Rentention(RententionPolicy.CLASS)：编译器把注释记录在类文件中，但在运行时不需要保留注释  
	- @Rentention(RententionPolicy.RUNTIME)：编译器把注释记录在类文件中，在运行时刻通过反射来获取  
- @Target：注解的目标对象 
	- ANNOTATION_TYPE：标注注释类型    
	- METHOD：标注方法  
	- CONSTRUCTOR：标注构造方法  	      
	- PACKAGE：标注包  
	- FIELD：标注字段  	      
	- PARAMETER：标注参数  
	- LOCAL_VARIABLE：标注局部变量  
	- TYPE：标注类、接口（包括注释类型）或枚举声明
- @DOCUMENTED
	- 如果类型声明是用Documented来注释的，则其注释将成为注释元素的公共API的一部分。
- @Inherited
	- 允许子类继承父类中的注解

## 2、锁的分类与使用
[锁的分类与使用](https://www.cnblogs.com/hustzzl/p/9343797.html)

## 3、synchronized
synchronized是Java中的关键字，是一种同步锁。它修饰的对象有以下几种：
- 修饰一个代码块，被修饰的代码块称为同步语句块，其作用的范围是大括号{}括起来的代码，作用的对象是调用这个代码块的对象； 
- 修饰一个方法，被修饰的方法称为同步方法，其作用的范围是整个方法，作用的对象是调用这个方法的对象； 
- 修饰一个静态的方法，其作用的范围是整个静态方法，作用的对象是这个类的所有对象；

```java
public class SynchronizeTest {

	/**
	* 对当前对象加锁
	* synchronized修饰方法块时，不同线程调用同一个对象去调用该方法块，只会有一个线程执行该段代码
	* 不同线程不同的对象能同时访问该部分代码块
	*/
	public void test1() {
		synchronized(this){
			for (int num = 0; num < 50; num++) {
				System.out.println("test1 "+Thread.currentThread().getName());
			}
		}
	}
	
	/**
	* synchronized(Object.class)
	* 当对Object对象上锁时，同一时间只能有一个线程访问该代码块
	*/
	public void test2() {
		synchronized(Object.class){
			for (int num = 0; num < 50; num++) {
				System.out.println("test2 "+Thread.currentThread().getName());
			}
		}
	}
	
	/**
	* synchronized(SynchronizeOnCodeBlock.class)
	* 当对SynchronizeOnCodeBlock对象上锁时，同一时间只能有一个线程访问该代码块
	*/
	public void test3() {
		synchronized (SynchronizeOnCodeBlock.class) {
			for (int num = 0; num < 50; num++) {
				System.out.println("test3 " + Thread.currentThread().getName());
			}
		}
	}
	
	/**
	 * synchronized修饰方法时，不同线程调用同一个对象去调用该方法，只会有一个线程执行该段代码
	*/
	public synchronized void test4() {
		for (int num = 0; num < 50; num++) {
			System.out.println("test1 " + Thread.currentThread().getName());
		}
	}

	/**
	* synchronized修饰静态方式时，和synchronized(SynchronizeOnMethod.class)类似，同一时间只能有一个线程访问
	*/
	public static synchronized void test5() {
		for (int num = 0; num < 50; num++) {
			System.out.println("test2 " + Thread.currentThread().getName());
		}
	}

}
```

**线程间通信**
```java
public class WaitAndNotifyTest {

	static Object obj = new Object();
	static int state = 1;
	
	public static void main(String[] args) {
		Thread t1 = new PrintOddThread();
		Thread t2 = new PrintEvenThread();
		t1.start();
		t2.start();
	}

	/**
	 * 打印奇数，每次打印五个，然后另一个线程打印偶数
	 */
	static class PrintOddThread extends Thread{
		
		@Override
		public void run() {
			synchronized (obj) {
				int num = 1;
				
				while(num <100){
					
					if(state  != 1){
						try {
							obj.wait();
						} catch (InterruptedException e1) {
							e1.printStackTrace();
						}
					}
				
					for(int i = 0;i < 5;i++){
						System.out.println(num);
						num = num + 2;
					}
					
					state =2;
					
					/**
					 * 唤醒之后从while处再次执行，到wait处停下，CPU自动选择一个等待中的线程执行
					 */
					obj.notifyAll();

				}
			}
		}
	}
	
	/**
	 * 打印偶数
	 */
	static class PrintEvenThread extends Thread{

		@Override
		public void run() {
			synchronized (obj) {
				int num = 2;
				
				while(num <100){
					
					if(state  != 2){
						try {
							obj.wait();
						} catch (InterruptedException e1) {
							e1.printStackTrace();
						}
					}
				
					for(int i = 0;i < 5;i++){
						System.out.println(num);
						num = num + 2;
					}
					
					state =1;
					
					obj.notifyAll();

				}
			}
		}
	}
}

```

## 4、ReentrantLock
ReentrantLock和synchronized都是可重入锁。
```java
	class X {
   		private final ReentrantLock lock = new ReentrantLock();

   		public void m() { 
     			lock.lock();  // block until condition holds
     			try {
       				// ... method body
     			} finally {
      		 		lock.unlock()
     			}
   		}	
	 }
```

**使用ReentrantLock和Condition实现消费者和生产者模型**
```java
/**
 * 线程间通信
 */
public class LockAndConditionTest{
	
	static int state = 1;
	
	static Lock lock = new ReentrantLock();
	static Condition oddCon = lock.newCondition();
	static Condition evenCon = lock.newCondition();
	
	public static void main(String[] args) {
		
		Thread t1 = new PrintOddThread();
		Thread t2 = new PrintEvenThread();
		t1.start();
		t2.start();
	}

	/**
	 *	打印奇数，每次打印五个，然后另一个线程打印偶数
	 */
	static class PrintOddThread extends Thread{
		
		@Override
		public void run() {
			
			int num = 1;
			
			while(num <100){
				
				// 非阻塞
				lock.lock();
				
				if(state  != 1){
					try {
						// 阻塞
						oddCon.await();
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
				}
			
				for(int i = 0;i < 5;i++){
					System.out.println(num);
					num = num + 2;
				}
				
				state =2;
				
				evenCon.signalAll();
				
				lock.unlock();
				
			}
		}
	}
	
	static class PrintEvenThread extends Thread{

		@Override
		public void run() {
			
			int num = 2;
			
			while(num <100){
				
				lock.lock();
				
				if(state  != 2){
					try {
						evenCon.await();
					} catch (InterruptedException e1) {
						e1.printStackTrace();
					}
				}
			
				for(int i = 0;i < 5;i++){
					System.out.println(num);
					num = num + 2;
				}
				
				state =1;
				
				oddCon.signalAll();
				
				lock.unlock();
				
			}
		}
	}
	
}

```

### ReentrantLock和synchronized的比较
- ReentrantLock功能性方面更全面，比如时间锁等候，可中断锁等候，锁投票等，因此更有扩展性。在多个条件变量和高度竞争锁的地方，用ReentrantLock更合适，ReentrantLock还提供了Condition，对线程的等待和唤醒等操作更加灵活，一个ReentrantLock可以有多个Condition实例，所以更有扩展性。
- ReentrantLock必须在finally中释放锁，否则后果很严重，编码角度来说使用synchronized更加简单，不容易遗漏或者出错。
- ReentrantLock 的性能比synchronized会好点。
- ReentrantLock提供了可轮询的锁请求，他可以尝试的去取得锁，如果取得成功则继续处理，取得不成功，可以等下次运行的时候处理，所以不容易产生死锁，而synchronized则一旦进入锁请求要么成功，要么一直阻塞，所以更容易产生死锁。
- Lock的某些方法可以决定多长时间内尝试获取锁，如果获取不到就抛异常，这样就可以一定程度上减轻死锁的可能性。
如果锁被另一个线程占据了，synchronized只会一直等待，很容易错序死锁 
- synchronized的话，锁的范围是整个方法或synchronized块部分；而Lock因为是方法调用，可以跨方法，灵活性更大 

## 5、线程池 
线程池的思想是：在系统中开辟一块区域，其中存放一些待命的线程，这个区域被称为线程池。如果有需要执行的任务，则从线程池中借一个待命的线程来执行指定的任务，到任务结束可以再将所借线程归还。这样就避免了大量重复创建线程对象，浪费CPU，内存资源。
### 线程池的核心参数
- poolSize：线程池中当前线程的数量。
- corePoolSize：池中所保存的线程数，包括空闲线程
- maximumPoolSize：线程池最大线程数量。  
- keepAliveTime：线程数量超过corePoolSize时，多于的空闲线程的存活时间（超过这段时间，该空闲线程会被销毁）。  
- unit：keepAliveTime的单位，参见枚举类TimeUnit  
- workQueue：一个阻塞队列，用来存储等待执行的任务。这队列用来保持那些execute()方法提交的还没有执行的任务。常用的队列有- SynchronousQueue,LinkedBlockingQueue,ArrayBlockingQueue。 
- threadFactory：执行程序创建新线程使用的工厂  
- handler：由于超出线程范围和队列容量而使执行被阻塞时所使用的处理程序。当线程的数量已经到了边界值，并且workQueue中任务也达到最大值，此时需要使用处理器处理多余的任务。该参数有四个值，分别是CallerRunsPolicy，AbortPolicy，DiscardPolicy，DiscardOldestPolicy。这四个处理器是线程池的内部类，都实现了RejectedExecutionHandler接口。
	- CallerRunsPolicy:在调用execut方法的调用线程中直接执行线程池拒绝的任务；
	- AbortPolicy：以抛出一个RejectedExecutionException的方式处理拒绝执行的任务；
	- DiscardPolicy：以直接忽略的方式处理拒绝执行的任务；
	- DiscardOldestPolicy：忽略掉最老的没有处理的拒绝任务，然后继续尝试执行execute方法，直到线程池关闭。
	
### 那么poolSize、corePoolSize、maximumPoolSize三者的关系是如何的呢？当新提交一个任务时：
- 如果poolSize<corePoolSize，新增加一个线程处理新的任务。
- 如果poolSize=corePoolSize，新任务会被放入阻塞队列等待。
- 如果阻塞队列的容量达到上限，且这时poolSize<maximumPoolSize，新增线程来处理任务。
- 如果阻塞队列满了，且poolSize=maximumPoolSize，那么线程池已经达到极限，会根据饱和策略RejectedExecutionHandler执行新的任务

**如果观察jdk提供的各种线程池的源码实现可以发现，除了jdk8新增的线程池newWorkStealingPool以外，都是基于对ThreadPoolExecutor的封装实现**
```java	
    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }
```

Executors工厂方法来创建线程池：
- Executors.newCachedThreadPool：无界线程池，可以进行自动线程回收。适用于执行大量耗时小的任务   
```java
    public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }
```

- Executors.newFixedThreadPool：线程数量固定的线程池，即只有核心线程，可以更快的响应外界请求
```java
    public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
```

- Executors.newSingleThreadPool：单个后台线程
```java
    public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }
```

- Executors.newScheduledThreadPool：核心线程数量是固定的，非核心线程数量是不固定的，当非核心线程闲置时他会被立即回收。主
要用于执行定时任务和具有固定时期的重复任务。
```java
    public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
        return new ScheduledThreadPoolExecutor(corePoolSize);
    }
    
    public ScheduledThreadPoolExecutor(int corePoolSize) {
        super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS,
              new DelayedWorkQueue());
    }
    
    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue) {
        this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
             Executors.defaultThreadFactory(), defaultHandler);
    }
```

## 6、四种引用--可以让程序员通过代码的方式决定某些对象的生命周期；有利于JVM进行垃圾回收。
- **强引用**：指创建一个对象并把这个对象赋给一个引用变量。
	```java
	Object obj = new Object(); // 只要引用存在，垃圾回收器永远不会回收，只有当obj被这个引用被释放掉之后，对象才会被释放掉
	```

- **软引用SoftReference**：如果一个对象具有软引用，内存空间足够，垃圾回收器就不会回收他。如果内存空间不足，就会被回收。软引用可以
和引用队列(ReferenceQueue)联合使用，如果软引用所引用的对象被垃圾回收了，java虚拟机就会把这个软引用加入到与之关联的引用队列中。
	```java
	Object obj = new Object();
	SoftReference<Object> sf = new SoftReference<Object>(obj);
	obj = null;
	sf.get(); // 有时会返回null
	```
这个时候sf是对obj的软引用，通过sf.get()方法可以去到这个对象，当然，当这个对象被标记为需要回收的对象时，会返回null。软应用主要用来实现类似缓存的功能，在内存足够的时候，直接通过软引用取值，无需从繁忙的真实来源取值，提升性能。当内存不足时，自动删除这些缓存数据，再从真实来源查询这些数据。

- **弱引用WeakReference**：弱引用也是用来描述非必须对象的，当JVM进行垃圾回收时，无论内存是否充足，都会回收被弱引用关联的对象。弱
引用可以和引用队列(ReferenceQueue)联合使用，如果弱引用所引用的对象被垃圾回收了，java虚拟机就会把这个弱引用加入到与之关联的引用队列
中。
	```java		
	Object obj = new Object();
	WeakReference<Object> wf = new WeakReference<Object>(obj);
	obj = null;
	wf.get(); // 有时会返回null
	wf.isEnQueued(); // 返回是否被垃圾回收器标记为即将回收的垃圾。
	```

- **虚引用PhantomReference**：虚引用不影响对象的生命周期，如果一个对象和虚引用关联，则跟没有引用与之关联一样，在任何时候都可能被垃圾回收器回收。虚引用必须和引用队列关联使用，当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会把这个虚引用加入到与之关联的引用队列中。如果程序发现某个虚引用已经被加入到引用队列，那么就可以在所引用对象的内存被回收之前采取必要的行动。
	```java
	Object obj = new Object();
	PhantomReference<Object> pf = new PhantomReference<Object>(obj);
	obj = null;
	pf.get(); // 永远返回null
	pf.isEnQueued();
	```

补充说明：在任何时候我们都可以调用ReferenceQueue的Poll()方法来检查是否有他所关心的非强可及对象被回收。如果队列为空，则返回一个null，否则返回队列中前面的一个Reference对象。

## 7、位运算符
- 按位与 & ：同时为1则为1，否则为0
- 按位或 |：只要一个为1，则为1
- 异或  ^ ：两个相应位值不同，则为1，否则为0
- 右移运算符：>>，右移相当于除以2的N次方，舍弃余数，符号位也会移动，左补0或1，看是正数还是负数
- 左移运算符：<<，左移相当于乘以2的N次方
- 无符号右移运算符：>>>

## 8、垃圾收集算法
### 如何判断对象是否还存活
- 引用计数算法：给对象添加一个引用计数器，每当有一个地方引用它时，计数器值加1，当引用失效时，计数器值减一。但是对象之间相互循环引用会导致该算法出现问题。
- 可达性分析算法：这个算法的基本思路是通过一系列的称之为“GC Roots”的对象作为起点，从这些节点开始向下搜索，搜索走过的路径称为引用链，当一个对象到GC Roots没有任何引用链相连时，则证明此对象是不可用的。在java中，可作为GC Roots的对象包括虚拟机栈中引用的对象、方法区中类静态属性引用的对象、方法区中常亮引用的对象、本地方法栈中JNI（即Native方法）引用的对象。
### 收集算法
- 标记-清除算法：首先标记出所有需要回收的对象，在标记完成后统一回收被标记的对象。不足之处在于 1、效率问题，标记和清除的两个过程效率都不太高 2、空间问题  标记清除之后会产生大量不连续的内存碎片，空间碎片太多可能会导致以后在程序运行过程中需要分配较大对象时，无法找到足够的连续内存而不得不提前触发另一次垃圾收集动作。
- 复制算法：将可用内存按容量划分为等量的两块，每次只使用其中的一块。当这一块内存用完了，就将还存活着的对象复制到另外一块上面，然后再把已使用过的内存空间一次清理掉。这种算法现在大多用来清理新生代。由于新生代朝生夕死，所以将内存分为一个较大的Eden区域和两块较小的Survivor区域，当回收时，将Eden和Survivor上存活的对象复制到另一块Survivor中，然后清理Eden和Survivor，当Survivor空间不足时，需要将某些新生代通过分配担保机制进入老年代。
- 标记-整理算法：类似于标记清除算法，标记完成后将所有存活的对象向一端移动，然后直接清理掉端边界以外的内存。
- 分代收集算法：将对象分为新生代和老年代，在新生代中，每次垃圾收集时都有大批对象死去，就选用复制算法。而老年代中因为对象存活率高，没有额外空间对它进行分配担保，就必须使用标记清理或标记整理算法进行回收。       

## 9、垃圾收集器

- Serial收集器：是最基本、发展历史最悠久的收集器，在jdk1.3.1之前是虚拟机新生代收集的唯一选择。这个收集器是一个单线程的收集器，但它的单线程的意义并不仅仅说明它只会使用一个CPU或一条收集线程去完成垃圾收集工作，更重要的是它进行垃圾收集时，必须暂停其它所有的工作线程，直到它收集结束。“stop the world”这个名字也许听起来很酷，但这项工作实际上是由虚拟机在后台自动发起和自动完成的，在用户不可见的情况下把用户正常工作的线程全部暂停，这对很多应用来说都是难以接受的。但实际到现在为止，它依然是虚拟机运行在client模式下的默认新生代收集器，因为它高效的收集效率。

- ParNew收集器：它是Serial收集器的多线程版本，除了使用多条线程进行垃圾收集之外，其余行为包括Serial收集器可用的所有控制参数、收集算法、stop the world、对象分配规则、回收策略等都与Serial收集器完全一样。它是运行在Server模式下的虚拟机中首选的新生代收集器，一个很重要的原因是，除了Serial收集器外，目前只有它能与CMS收集器配合工作。

- Parallel Scavenge收集器：它是一个新生代收集器，它也是使用**复制算法**的收集器，又是并行（**指的是多条垃圾收集线程并行工作，但用户线程此时仍然处于等待状态**）的多线程收集器。它的特点是它的关注点与其它收集器不同，CMS等收集器的关注点是尽可能缩短收集时用户线程的停顿时间，而Parallel Scavenge收集器的目标则是达到一个可控制的吞吐量。所谓吞吐量就是CPU用于运行用户代码的时间与CPU总总消耗时间的比值，即吞吐量=运行用户代码的时间 / (运行用户代码的时间 + 垃圾收集时间)，虚拟机总共运行了100分字，其中垃圾收集花掉1分钟，那吞吐量就是99%，而高吞吐量则可以高效率地利用CPU时间，尽快完成程序的运算任务，主要适合在后台运算而不需要太多交互的任务。Parallel Scavenge收集器有一个参数-XX:+UseAdaptiveSizePolicy值得关注。当这个参数开关打开之后，就不需要收集指定新生代的大小、Eden与Survivor的比例、晋升老年代对象大小等细节参数了，虚拟机会很具当前系统的运行情况收集性能监控信息，动态调整这些参数以提供最合适的停顿时间或最大吞吐量，这种调节方式称为GC的自适应的调节策略。

- Serial Old收集器：它是Serial收集器的老年代版本，它同样是一个单线程收集器，**使用“标记-整理”算法**。这个收集器的主要意义也是在于给Client模式下的虚拟机使用。如果在Server模式下，那么它主要还有两大用途：一种是在JDK1.5之后的版本中与Parallel Scavenge收集器搭配使用，另一种用途就是作为CMS收集器的后背预案，在并发（**指的是用户线程和垃圾收集线程同时执行，但不一定是并行的，可能会交替执行**）收集发生Concurrent Mode Failure时使用。

- Parallel old收集器：它是Parallel Scavenge收集器的老年代版本，使用多线程和“**标记-整理**”算法。这个收集器在JDK1.6中才开始提供的，在此之前，新生代的Parallel Scavenge收集器一直处于比较尴尬的状态。原因是，如果新生代选择了Parallel Scavenge收集器，老年代除了Serial Old收集器外别无选择。

- CMS收集器：Concurrent Mark Sweep收集器是一种以获取最短回收停顿时间为目标的收集器。从名字上就可以看出，它是基于“**标记-清除**”算法的。它的运作过程更复杂一些，整个过程分为4个步骤：

  - 初始标记：仅仅只是标记一下GC Roots能直接关联到的对象，速度很快，仍然需要Stop the world
  - 并发标记：进行GC Roots跟踪的过程，不需要Stop the world
  - 重新标记：修正并发标记期间用户程序继续运作而导致标记产生变动的那一部分标记记录，仍然需要Stop the world
  - 并发清除：清除GC Roots不可达对象，不需要Stop the world

  CMS收集器的缺点：

  - 对CPU资源非常敏感，对用户程序的影响随着CPU数量的减少而增加，适合于多CPU的环境
  - 无法处理浮动垃圾，可能出现Concurrent Mode Failure失败而导致另一次Full GC的产生。由于CMS并发清理阶段用户线程还在运行，伴随着程序运行自然就会有垃圾不断产生，而在当次收集中无法处理掉他们，只好留待下一次GC时再清理掉，这一部分垃圾就称为浮动垃圾。**也是由于在垃圾收集阶段用户线程还需要运行，那也就还需要预留有足够的内存空间给用户线程使用**，因此CMS收集器不能像其它收集器那样等老年代几乎完全填满了再进行收集，要是CMS运行期间预留的内存无法满足程序需要，就会出现一次Concurrent Mode Failure失败，这时虚拟机会启动后背预案，临时启用Serial Old收集器来重新进行老年代的垃圾收集。
  - 会出现大量的空间碎片：由于CMS是基于“标记-清除”算法实现的，这意味着收集结束时会有大量空间碎片产生，空间碎片过多时，会给大对象分配带来很大麻烦，不得不提前触发一次Full GC。

- G1收集器：Garbage-First收集器是一款卖你想服务端应用的垃圾收集器，它的使命是替换掉JDK1.5中发布的CMS收集器。在G1收集器之前的其它收集器收集的范围都是整个新生代或者老年代，而G1不再是这样，它将整个Java堆内存划分为多个大小相等的独立区域（Region），虽然它还保留新生代和老年代的概念，但新生代和老年代不再是物理隔离的了，而都是一部分Region（可以不连续）的集合。G1收集器具备以下特点：

  - 并行与并发：G1能充分利用多CPU、多核环境下的硬件优势，使用多个CPU来缩短stop the world停顿的时间，部分其它收集器原本需要停顿Java线程执行的GC动作，G1收集器仍然可以通过并发的方式让Java程序继续执行
  - 分代收集：G1可以不需要其它收集器配合就能独立管理整个GC堆
  - 空间整合：G1整体上来看是基于“**标记-整理**”算法实现的收集器，不会产生内存空间碎片
  - 可预测的停顿：能建立可预测的停顿时间模型，能让使用者明确指定在一个长度为m毫秒的时间片段内，消耗在垃圾收集上的时间不得超过n毫秒
  - 垃圾优先：G1收集器避免全区域垃圾收集，它跟踪各个Region里面的垃圾堆积的价值大小（回收所获得的空间大小以及回收所需时间的经验值），在后台维护一个优先级列表，每次根据允许的收集时间，优先回收价值最大的Region（这也是Garbage-First名称的由来）。区域划分和优先级区域回收机制，确保G1收集器可以在有限时间获得最高的垃圾收集效率

  G1收集器的运作大致可分为以下几个步骤：

  - 初始标记（Initial Marking）：标记一下GC Roots能直接关联到的对象，这阶段需要stop the world，但时间很短
  - 并发标记（Concurrent Marking）：这阶段是从GC Root开始对堆中对象开始可达性分析，找出存活的对象，这阶段耗时较长，但可以并发执行
  - 最终标记（Final Marking）：修正在并发标记期间因用户程序继续运作而导致标记产生变动的那一部分记录，该阶段需要stop the world
  - 筛选回收（Live Data Counting and Evacuation）：对各个Region的回收价值和成本进行筛选排序，根据用户所期望的GC停顿时间来指定回收计划

- ZGC垃圾收集器：在JDK11当中，加入了实验性质的ZGC。它的回收耗时平均不到2毫秒。ZGC是一个并发、基于区域（region）、增量式压缩的收集器，它是一款低停顿高并发的收集器。ZGC几乎在所有地方都是并发执行的，**除了初始标记是stop the world的**，所以停顿时间几乎就耗费在初始标记上。

  ZGC主要新增了两项技术，一个是着色指针colored pointer，另一个是读屏障load barrier。



### 理解GC日志

33.125：[GC  [DefNew：3324K -> 152K (3712K)，0.0025925 secs] 3324K -> 152K（11904K），0.0031680 secs]

100.667：[Full GC  [Tenured：0K-> 210K（10240K），0.0149142 secs]  4603K -> 210K（19456K），[Perm：2999K -> 2999K (21248K) ] ，0.015007 secs]  [Times：user=0.01 sys=0.00 real=0.02 secs]

最前面的数字33.125和100.667代表了GC发生的时间，这个数字的含义是从Java虚拟机启动依赖经过的秒数。

GC日志开头的GC和Full GC说明了这次垃圾收集的停顿类型，而不是用来区分新生代GC还是老年代GC的。如果有Full，说明这次GC是发生了Stop-The-World的。新生代收集器ParNew也会出现"[Full GC"（这一般是因为分配担保失败之类的问题，所以才导致STW）。如果是调用System.gc()方法所触发的收集，那么在这里将显示"FULL GC(System)"。

接下来的[DefNew、[Tenured、[Perm表示GC发生的区域，这里显示的区域名称与使用的GC收集器是密切相关的，例如上面阳历所使用的Serial收集器中新生代名为“Default New Generation”，所以显示的是[DefNew。Serial收集器的老年代和永久代分别表示为**"Tenured"**、**"Perm"**。如果是Parnew收集器，新生代名称就会变为[ParNew。如果使用的是Parallel Scavenge收集器，那他配套的新生代称为“PSYoungGen”，老年代和永久代同理，名称也是由收集器决定的。

后面方括号内部的“3324K -> 152K（3712K）”含义是GC前该区域已使用容量、GC后该内存区域已使用容量、该内存区域总容量。而在方括号之外的3324K -> 152K（11904K）表示GC前Java堆已使用容量、Java堆总容量。

再往后“0.0025925 secs”表示该内存区域GC所占用的时间，单位是秒。有的收集器会给出更具体的时间数据，如[Times：user=0.01 sys=0.00 real=0.02 secs]，这里的表示与Linux的time命令所输出的时间含义一致，分别代表用户态消耗的CPU时间、内核态消耗的CPU时间和操作从开始到结束所经过的墙钟时间。



## 10、内存分配与回收策略

- 对象优先在Eden分配：大多数情况下，对象在新生代Eden区中分配。当Eden区没有足够空间进行分配时，虚拟机将发起一次**Minor GC**
- 大对象直接进入老年代：所谓的大对象是指，需要大量连续内存的空间对象，最典型的就是那种很长的字符串以及数组。
- 长期存活的对象将进入老年代：既然虚拟机采用了分代收集的思想来管理内存，那么内存回收时就必须能识别哪些对象应放在新生代，哪些对象应放在老年代中。为了做到这点，虚拟机给每个对象定义了一个对象年龄（Age）计数器。如果对象在Eden出生并经过第一次Minor GC后仍然存活，并且能被Survivor容纳的话，将被移动到Survivor空间中，并且对象年龄设为1。对象在Survivor区中每熬过一次Minor GC，年龄就增加1岁，当它的年龄增加到一定程度（默认为15岁），就将被晋升到老年代中。
- 动态对象年龄判断：为了能更好地适应不同程序的内存状况，虚拟机并不是永远地要求对象的年龄必须达到了MaxTenuringThreshold才能晋升老年代，如果在Survivor空间中相同年龄所有对象大小的总和大于Survivor空间的一半，年龄大于等于该年龄的对象就可以直接进入老年代。
- 空间分配担保：**在发生Minor GC之前**，虚拟机会检查老年代最大可用的连续空间是否大于新生代所有对象总空间，如果这个条件成立，那么Minor GC可以确保是安全的。如果不成立，则虚拟机会查看HandlePromotionFailure设置值是否允许担保失败。如果允许，那么会继续检查老年代最大可用的连续空间是否大于历次晋升到老年代对象的平均大小，如果大于，将尝试着进行一次Minor GC，尽管这次Minor GC是有风险的；如果小于，或者HandlePromotionFailure设置不允许冒险，那这时也要改为进行一次Full GC。在JDK 6 Update 24之后，HandlePromotionFailure参数在代码中已经不会再使用它，规则变为只要老年代的连续空间大于新生代对象总大小或者历次晋升的平均大小就会进行Minor GC，否则将进行Full GC。



### Minor GC和Full GC

- 新生代GC（Minor GC）：指发生在新生代的垃圾收集动作，因为Java对象大多都具备朝生夕死的特性，所以Minor GC非常频繁，一般回收速度也比较快。
- 老年代GC（Major GC / Full GC）：指发生在老年代的GC，出现了Major GC，经常会伴随着至少一次的Minor GC（但非绝对的，在Parallel Scavenge收集器的收集策略里就有直接进行Major GC的策略选择过程）。Major GC的速度一般会比Minor GC慢10倍以上。



## 11、Java内存区域

- 程序计数器（Program Counter Register）：是一块较小的内存空间，它可以看成是当前线程所执行的字节码的行号指示器。每条线程都需要一个独立的程序计数器，各条线程之间计数器互不影响，独立存储。该内存区域是唯一一个在java虚拟机规范中没有规定任何OutOfMemoryError情况的区域。
- java虚拟机栈（Stack）：和程序计数器一样是线程私有的，它的生命周期和线程相同。每个方法在执行的同时会创建一个栈帧用语存储局部变量表、操作数栈、动态链接、方法出口等信息。局部变量表存放了编译期可知的各种基本数据类型(int/short/float/long/double/boolean/char/byte)、对象引用。如果线程请求的栈深度大于虚拟机所允许的深度，将抛出StackOverfolwError异常，如果虚拟机允许动态扩展，但无法申请到足够的内存，将抛出OutOfMemeroyError异常。
- 本地方法栈（Native Method Stack）：和虚拟机栈类似，本地方法栈用于执行Native方法。
- java堆(Heap)：java堆是被所有线程共享的内存区域，几乎所有的对象实例和数组都要在堆上分配
- 方法区(Method Area )：和java堆一样，是各个线程共享的内存区域，它用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。**对于HotSpot虚拟机，方法区也叫作永久代（Permanent Generation）**，本质上两者并不等价，仅仅是因为Hotspot虚拟机把GC分代收集扩展至方法区，或者说使用永久代来实现方法区而已，这样Hotspot的垃圾收集器可以像管理Java堆一样管理这部分内存区域。对于其它虚拟机（如BEA JRockit、IBM J9等）来说是不存在永久代的概念的。
- 运行时常量池：**是方法区的一部分**。Class文件中除了有类的版本、字段、方法、接口等信息外，还有一项信息是常量池（constant pool table），用于存放编译期生成的各种字面量和符号引用，这部分内容将在类加载后进入方法区的运行时常量池中存放。
- 直接内存（Direct Memory）：直接内存并不是虚拟机运行时数据区的一部分，也不是java虚拟机规范中定义的内存区域。但是这部分内存也被频繁使用，而且也可能导致OutOfMemoryError异常出现。在JDK1.4中新加入了NIO类，引入了一种基于通道（Channel）与缓冲区（Buffer）的I/O方式，它可以使用Native函数库直接分配堆外内存，然后通过一个存储在Java堆中的DirectByteBuffer对象作为作为这块内存的引用进行操作。这样能在一些场景中显著提高性能，因为避免了在Java堆和Native堆中来回复制数据。
- 元空间（Metaspace）：永久代是Hotspot对于JVM方法区规范的实现。在JDK1.8中，Hotspot已经没有永久代这个区间了，取而代之的是元空间。元空间的本质和永久代类似，都是对JVM规范中方法区的实现，不过元空间与永久代之间最大的区别在于：元空间并不在虚拟机中，而是使用本地内存。因此，默认情况下，元空间的大小仅受本地内存限制。类的元数据放入本地内存，字符串池和类的静态变量放入Java堆中。



## 12、Java内存模型（Java Memory Model）

Java内存模型的主要目标是定义程序中各个变量的访问规则，即在虚拟机中将变量存储到内存和从内存中取出变量这样的底层细节。此处的变量（variables）与Java编程中所说的变量有所区别，它包括了**实例字段、静态字段和构成数组对象的元素，但不包括局部变量与方法参数**，因为后者是线程私有的，不会被共享，自然就不会存在竞争问题。

Java内存模型规定了所有的变量都存储在主内存中（Main Memory），每条线程还有自己的工作内存（working memory），线程的工作内存中保存了被该线程使用到的变量的主内存副本拷贝，线程对变量的所有操作（读取、赋值等）都必须在工作内存中进行，而不能直接读写主内存中的变量。不同的线程之间也无法直接访问对方工作内存中的变量，线程间变量值的传递均需要通过主内存来完成。  

![images](https://github.com/FantasyLakers/my-lessons/blob/master/java/Java%E5%86%85%E5%AD%98%E6%A8%A1%E5%9E%8B.jpg)

### 内存间交互操作

关于主内存与工作内存之间具体的交互协议，即一个变量如何从主内存拷贝到工作内存、如何从工作内存同步回主内存之类的实现细节，Java内存模型中定义了以下8中操作来完成，虚拟机实现时必须保证下面提及的每一种操作都是原子的、不可再分的。

- lock（锁定）：作用于主内存的变量，它把一个变量标识为一条线程独占的状态
- unlock（解锁）：作用于主内存的变量，它把一个处于锁定状态的变量释放出来，释放后的变量才可以被其它线程锁定
- read（读取）：作用于主内存的变量，它把一个变量的值从主内存传输到线程的工作内存中，以便后续的load动作使用
- load（载入）：作用于工作内存的变量，它把read操作从主内存中得到的变量值放入工作内存的变量副本中
- use（使用）：作用于工作内存的变量，它把工作内存中一个变量的值传递给执行引擎，每当虚拟机遇到一个需要使用到变量的值的字节码指令时将会执行这个动作
- assign（赋值）：作用于工作内存的变量。它把一个从执行引擎接收到的值赋给工作内存的变量，每当虚拟机遇到一个给变量赋值的字节码指令时执行这个动作
- store（存储）：作用于工作内存的变量，它把工作内存中一个变量的值传送到主内存中，以便随后的write操作使用
- write（写入）：作用于主内存的变量，它把store操作从工作内存中得到的变量的值放入主内存的变量中 

## 13、java.util包和java.util.concurrent包下的关键类和接口
### Queue：队列是一个先进先出的（FIFO）的数据结构。常用的并发队列有阻塞队列和非阻塞队列，前者使用锁实现，后者则使用CAS非阻塞算法实现。
- 非阻塞队列：
	- LinkedList：一个由链表结构组成的双向队列。
	- PriorityQueue：一个由数组结构组成的基于优先级的无界优先级队列。
	- ConcurrentLinkedQueue：一个采用链表实现的无界非阻塞**线程安全**队列。
- 阻塞队列（继承BlockingQueue）
	- ArrayBlockingQueue：一个由数组结构组成的有界阻塞队列。
	- LinkedBlockingQueue：一个由链表结构组成的有界阻塞队列。
	- PriorityBlockingQueue：一个支持优先级排序的无界阻塞队列。默认情况下队列中的元素采取自然顺序排序，也可以通过比较器comparator来指定元素的排序规则。
	- DelayQueue：一个使用优先级队列PriorityBlockingQueue实现的无界阻塞队列，是一个支持延时获取元素的队列。队列中的元素必须实现Delayed接口，在创建元素时可以指定多久才能从队列中获取元素，只有在延迟期满时才能从队列中提取元素。
	- SynchronousQueue：一个不存储元素的阻塞队列。每一个put操作必须等待一个take操作，否则不能继续添加元素。
	- LinkedTransferQueue：一个由链表结构组成的无界阻塞队列。相对于其他阻塞队列，LinkedTransferQueue多了tryTransfer和transfer方法。**transfer方法**：如果当前由消费者正在等待接收元素，transfer可以把生产者传入的元素立刻transfer给消费者。如果没有消费者正在等待，transfer方法会将元素放在队列的tail节点，并等到该元素被消费者消费了才返回。**tryTransfer方法**：是用来试探生产者传入的元素是否能直接传给消费者，如果没有消费者在等待，则返回false。和transfer方法的区别是tryTransfer方法无论消费者是否接受，方法立即返回，而transfer方法是必须等到消费者消费了才返回。
	- LinkedBlockingDeque：一个由链表结构组成的**双向阻塞队列**。
### List
- ArrayList：底层是动态数组。
- LinkedList：底层是双向链表。
- Vector：线程安全的动态数组。
### Map
- HashMap：底层是散列表。
- ConcurrentHashMap：线程安全的。
- HashTable：线程安全的。
- TreeMap：实现了SortedMap接口，能够把它保存的记录根据键值排序，默认是按键值的升序排序，也可以指定排序的比较器，当用Iterator遍历TreeMap时，得到的记录是排过序的。
- LinkedHashMap：是HashMap的子类，LinkedHashMap中的遍历可分为插入顺序和访问顺序两种，底层维护了一个双向链表用来保存元素的顺序。构造方法中的accessOrder默认为为false，表示LinkedHashMap中存储的元素是按照调用put方法插入的顺序进行排序的，如果指定按访问顺序排序，那么调用get方法后，会将这次访问的元素移至链表尾部，不断访问可以形成按访问顺序排序的链表。
- 一般情况下，我们用的最多的是HashMap。HashMap里面存入的键值对在取出的时候是随机的，它根据键的HashCode值存储数据，根据键可以直接获取它的值，具有很快的访问速度。在Map中插入、删除和定位元素，HashMap 是最好的选择。TreeMap取出来的是排序后的键值对，如果要按自然顺序或自定义顺序遍历键，那么TreeMap会更好。LinkedHashMap是HashMap的一个子类，如果需要输出的顺序和输入的相同,那么用LinkedHashMap可以实现。它还可以按读取顺序来排列，像连接池中可以应用。
### Set
- HashSet：底层是HashMap，不能保证元素的顺序。允许null值。HashSet要求放入的对象必须实现HashCode()方法，放入的对象，是以hash值作为标识的。
- TreeSet：底层是TreeMap，可以保证元素是由顺序的。不允许null值。TreeSet的底层实现是采用红-黑树的数据结构，采用这种结构可以从Set中获取有序的序列，但是前提条件是：元素必须实现Comparable接口。

## 13、ThreadLocal类
ThreadLocal类为每一个线程都维护了自己独有的变量拷贝。每个线程都拥有自己独立的变量。ThreadLocal采用了“以空间换时间”的方式：访问并行化，对象独享化。前者仅提供一份变量，让不同的线程排队访问，而后者为每一个线程都提供了一份变量，因此可以同时访问而互不影响。  

```java
    void createMap(Thread t, T firstValue) {
        t.threadLocals = new ThreadLocalMap(this, firstValue);
    }

    private T setInitialValue() {
        T value = initialValue();
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
        return value;
    }

    public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
    }

    public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }
    
    
```
ThreadLocal的实现原理：每个ThreadLocal的对象都有一个ThreadLocalMap，当创建一个ThreadLocal的时候，就会将该ThreadLocal对象添加到该Map中，其中键就是ThreadLocal，值可以是任意类型。

```java
	static class ThreadLocalMap {

        	static class Entry extends WeakReference<ThreadLocal<?>> {
           		 /** The value associated with this ThreadLocal. */
            		Object value;

            		Entry(ThreadLocal<?> k, Object v) {
               	 		super(k);
               	 		value = v;
            		}
        	}
		
		ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
            		table = new Entry[INITIAL_CAPACITY];
            		int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
            		table[i] = new Entry(firstKey, firstValue);
            		size = 1;
            		setThreshold(INITIAL_CAPACITY);
        	}
	}
```

### ThreadLocal为什么会内存泄漏？
ThreadLocalMap使用ThreadLocal的弱引用作为key，如果一个ThreadLocal没有外部强引用用来引用它，那么系统GC的时候，这个ThreadLocal势必会被回收，这样一来，ThreadLocalMap中就会出现key为null的Entry，就没有办法访问这些key为null的Entry的value，如果当前线程再迟迟不结束的话，这些key为null的Entry的value就会一直存在一条强引用链：Thread Ref->Thread->ThreadLocalMap->Entry-value永远无法回收，造成内存泄漏。其实，ThreadLocalMap的设计中已经考虑到了这种情况，也加上了一些防护措施：在ThreadLocal的get（），set（），remove（）的时候都会清除线程ThreadLocalMap里所有key为null的value。

从表面上看内存泄漏的根源在于使用了弱引用。为什么使用弱引用而不是强引用？下面我们分两种情况讨论：
- key使用强引用：引用的ThreadLocal的对象被回收了，但是ThreadLocalMap还持有ThreadLocal的强引用，如果没有手动删除，ThreadLocal不会被回收，导致Entry内存泄漏。
- key使用弱引用：引用的ThreadLocal的对象被回收了，由于ThreadLocalMap持有ThreadLocal的弱引用，即使没有手动删除，ThreadLocal也会被回收。value在下一次ThreadLocalMap调用set,get，remove的时候会被清除。
比较两种情况，我们可以发现：由于ThreadLocalMap的生命周期跟Thread一样长，如果都没有手动删除对应key，都会导致内存泄漏，但是使用弱引用可以多一层保障：弱引用ThreadLocal不会内存泄漏，对应的value在下一次ThreadLocalMap调用set,get,remove的时候会被清除。因此，ThreadLocal内存泄漏的根源是：由于ThreadLocalMap的生命周期跟Thread一样长，如果没有手动删除对应key就会导致内存泄漏，而不是因为弱引用。

## 14、volatile关键字
[volatile关键字](https://www.cnblogs.com/dolphin0520/p/3920373.html)

## 15、IO模型
- 阻塞IO：blocking IO，当用户线程发出IO请求后，内核回去查看是否有数据就绪，如果没有数据就绪，用户线程就会处于阻塞状态，用户线程交出CPU，当数据准备好后，内核会将数据拷贝到用户线程，并返回结果给用户线程，此时用户线程解除block状态。  

- 非阻塞IO：noblocking IO，当用户线程发起一个read操作后，并不需要等待，而是马上就得到了一个结果。如果结果是一个error时，它就知道数据还没有准备好，于是它可以再次发送read操作。一旦内核中的数据准备好了，并且又再次收到了用户线程的请求，那么它马上就将数据拷贝到了用户线程，然后返回。所以事实上，在非阻塞IO模型中，用户线程需要不断地询问内核数据是否就绪，也就说非阻塞IO不会交出CPU，而会一直占用CPU。  

- 信号驱动IO：signal blocking IO，在信号驱动IO模型中，当用户线程发起一个IO请求操作，会给对应的socket注册一个信号函数，然后用户线程会继续执行，当内核数据就绪时会发送一个信号给用户线程，用户线程接收到信号之后，便在信号函数中调用IO读写操作来进行实际的IO请求操作。这个一般用于UDP中，对TCP套接口几乎是没用的，原因是该信号产生得过于频繁，并且该信号的出现并没有告诉我们发生了什么事情。  

- 多路复用IO：IO multiplexing，java中的NIO模型。在多路复用IO模型中，会有一个线程不断去轮询多个socket的状态，只有当socket真正有读写事件时，才真正调用实际的IO读写操作。因为在多路复用IO模型中，只需要使用一个线程就可以管理多个socket，系统不需要建立新的进程或者线程，也不必维护这些线程和进程，并且只有在真正有socket读写事件进行时，才会使用IO资源，所以它大大减少了资源占用。另外多路复用IO为何比非阻塞IO模型的效率高是因为在非阻塞IO中，不断地询问socket状态是通过用户线程去进行的，而在多路复用IO中，轮询每个socket状态是内核在进行的，这个效率要比用户线程要高的多。不过要注意的是，多路复用IO模型是通过轮询的方式来检测是否有事件到达，并且对到达的事件逐一进行响应。因此对于多路复用IO模型来说，一旦事件响应体很大，那么就会导致后续的事件迟迟得不到处理，并且会影响新的事件轮询。  

- 异步IO：asynchronous IO，异步IO模型才是最理想的IO模型，在异步IO模型中，当用户线程发起read操作之后，立刻就可以开始去做其它的事。而另一方面，从内核的角度，当它受到一个asynchronous read之后，它会立刻返回，说明read请求已经成功发起了，因此不会对用户线程产生任何block。然后，内核会等待数据准备完成，然后将数据拷贝到用户线程，当这一切都完成之后，内核会给用户线程发送一个信号，告诉它read操作完成了。也就说用户线程完全不需要关心实际的整个IO操作是如何进行的，只需要先发起一个请求，当接收内核返回的成功信号时表示IO操作已经完成，可以直接去使用数据了。也就说在异步IO模型中，IO操作的两个阶段都不会阻塞用户线程，这两个阶段都是由内核自动完成，然后发送一个信号告知用户线程操作已完成。用户线程中不需要再次调用IO函数进行具体的读写。这点是和信号驱动模型有所不同的，在信号驱动模型中，当用户线程接收到信号表示数据已经就绪，然后需要用户线程调用IO函数进行实际的读写操作；而在异步IO模型中，收到信号表示IO操作已经完成，不需要再在用户线程中调用iO函数进行实际的读写操作。注意，异步IO是需要操作系统的底层支持，在Java 7中，提供了Asynchronous IO，简称AIO。

前面四种IO模型实际上都属于同步IO，只有最后一种是真正的异步IO，因为无论是多路复用IO还是信号驱动模型，IO操作的第2个阶段都会引起用户线程阻塞，也就是从内核进行数据拷贝的过程都会让用户线程阻塞。

## 16、Java中的字节流和字符流
![images](https://github.com/FantasyLakers/my-lessons/blob/master/java/java%20IO%E7%BB%93%E6%9E%84%E5%9B%BE.png)

## 17、NIO
![images](https://github.com/FantasyLakers/my-lessons/blob/master/java/NIO%E6%A8%A1%E5%9E%8B.png)

## 18、零拷贝 zero copy
Java的NIO为了支持零拷贝，提供了一些类：
- DirectByteBuffer：DirectByteBuffer直接在堆外分配内存，底层是直接通过JNI调用操作系统的NIO系统调用，所以性能会比较高。
- FileChannel：是Java NIO提供的用于复制文件的类，可以把文件复制到磁盘或者网络等。transferTo方法直接将当前通道内容传输到另一个通道，也就是说这种方式不会有内核缓冲区到用户缓冲区的读写问题。底层是sendfile系统调用。transferFrom方法同理。

## 19、类加载机制
类从被加载到虚拟机内存中开始，到卸载出内存为止，它的整个生命周期包括：加载（loading），验证（verification），准备（preparation），解析（resolution），初始化（initialization），使用（using）和卸载（unloading）七个阶段。其中验证，准备和解析三个部分统称为连接（linking）。加载、验证、准备、初始化和卸载这5个阶段的顺序是确定的，类的加载过程必须按照这种顺序按部就班地开始，而解析阶段则不一定：它在某些情况下可以在初始化阶段之后再开始，这就是为了支持Java语言的运行时绑定（也称为动态绑定或晚期绑定）。  
什么情况下需要开始类加载过程的第一个阶段：加载？Java虚拟机规范中并没有进行强制约束，这点可以交给虚拟机的具体实现来自由把我。但是对于初始化阶段，虚拟机规范则是严格规定了有且只有5种情况必须立即对类进行“初始化”（而加载、验证、准备自然需要在此之前开始）：
- 遇到new、getstatic、putstatic或者invokestatic这四条字节码指令时，如果类没有进行过初始化，则需要先触发其初始化。生成这四条指令的最常见的Java代码场景是：使用new关键字实例化队形的时候、读取或设置一个类的静态字段（被final修饰、已在编译期把结果放入常量池的静态字段除外）的时候，以及调用一个类的静态方法的时候。
- 使用java.lang.reflect包的方法对类进行反射调用的时候，如果类没有进行过初始化，则需要先出发其初始化。
- 当初始化一个类的时候，如果发现其父类还没有进行过初始化则需要先出发其父类的初始化。
- 当虚拟机启动时，当用户需要指定一个要执行的主类（包含main()方法的那个类），虚拟机会先初始化这个主类。
- 当使用JDK1.7的动态语言支持时，如果一个java.lang.invoke.MethodHandle实例最后的解析结果时REF_getStatic、REF_putStatic、REF_invokeStatic的方法句柄，并且这个方法句柄所对应的类没有进行过初始化，则需要先触发其初始化。  

对于这五种会触发类进行初始化的场景，虚拟机规范中使用了一个很强烈的限定语：有且只有。这五种场景中的行为称为对一个类进行主动调用。除此之外，所有引用类的方式都不会触发初始化，称为被动引用。下面举三个例子来说明何为被动引用：

### 被动引用示例一：通过子类引用父类的静态字段，不会导致子类初始化
```java
/**
 * 父类 
 * 通过子类引用父类的静态字段，不会导致子类初始化
 */
public class SuperClass {

	static {
		System.out.println("I am SuperClass");
	}

	// 父类中的静态字段
	public static int value = 100;
}

/**
 * 子类
 * 通过子类引用父类的静态字段，不会导致子类初始化
 */
public class SubClass extends SuperClass{

	static {
		System.out.println("I am SubCLass");
	}
}

public class Test {

	public static void main(String[] args) {
		// 最终输出I am SuperClass  100
		System.out.println(SubClass.value);
	}

}
```

### 被动引用示例二：通过通过数组定义来引用类，不会触发此类的初始化
```java
public class SuperClass {

	static {
		System.out.println("I am SuperClass");
	}

	// 父类中的静态字段
	public static int value = 100;
}

public class Test {

	public static void main(String[] args) {
		// 不会输出父类中的I am SuperClass
		SuperClass[] sca = new SuperClass[10];
	}

}
```

### 被动引用示例三：常量在编译阶段会存入类的常量池中，本质上没有直接引用到定义常量的类，因此不会触发定义常量的类的初始化
```java
public class ConstClass {

	static {
		System.out.println("I am ConstClass");
	}
	
	// 常量，如果把final去掉则会触发初始化
	public static final String HELLO = "hello world";
	
}

public class Test {

	public static void main(String[] args) {
		System.out.println(ConstClass.HELLO);
	}

}
```

### 接口的加载过程
接口的加载过程与类加载过程稍有一些不同，针对接口需要做一些特殊声明：接口中也有初始化过程，这点与类是一致的，上面的代码都是用静态语句块static来输出初始化信息的，而接口中不能使用static语句块，但编译器仍然会为接口生产“<clinit>()”类构造器，用于初始化接口中定义的成员变量。接口与类真正有所区别的是前面讲述的五种有且仅有需要开始初始化场景中的第三种：当一个类在初始化时，要求其父类全部都已经初始化过了，但是一个接口在初始化时，并不需要其父接口全部都完成了初始化，只有在真正使用到父接口的时候（如引用接口中定义的常量）才会初始化。

## 20、类加载的过程---加载、验证、准备、解析和初始化
- 加载：加载时类加载过程的一个阶段，希望大家没有混淆这两个名词。在加载阶段，虚拟机需要完成以下三件事情
	- 通过一个类的全限定名来获取定义此类的二进制字节流
	- 将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构
	- 在内存中生成一个代表这个类的java.lang.Class对象，作为方法区这个类的各种数据的访问入口  
- 连接：加载阶段与连接阶段的部分内容，如一部分字节码文件格式验证动作时交叉进行的，加载阶段尚未完成，连接阶段可能已经开始，但这些夹在加载阶段之中进行的动作，仍然属于连接阶段的内容，这两个阶段的开始时间仍然保持着固定的先后顺序。
- 验证：验证时连接阶段的第一步，这一阶段的目的时为了确保Class文件的字节流中包含的信息符合当前虚拟机的要求，并且不会危害虚拟机自身的安全。从整体上看，验证阶段大致会完成下面四个动作
	- 文件格式验证：验证字节流是否符合Class文件格式的规范，并且能被当前版本的虚拟机处理，这一阶段可能包含下面这些验证点：是否以魔数0xCAFEBABE开头，主、次版本号是否再当前虚拟机的处理范围内，常量池的常量是否有不被支持的常量类型，指向常量的各种索引值中是否有不存在的常量或不符合类型的常量，CONSTANT_Utf8_info型的常量中是否有不符合UTF8编码的数据，Class文件中各个部分及文件本身是否有被删除的或附加的信息。
	- 元数据验证：第二阶段是对字节码描述的信息进行语义分析，以保证其描述的信息符合Java语言规范。这个阶段可能包括的验证点如下：这个类是否有父类（除了Object类之外，所有的类都应该有父类），这个类的父类是否继承了不被允许继承的类（被final修饰的类），如果这个类不是抽象类，是否实现了其父类或接口之中要求实现的所有方法，类中的字段、方法是否与父类产生矛盾（例如覆盖了父类的final字段或者出现不符合规则的方法重载，例如方法参数都一致，但返回值类型却不同等）
	- 字节码验证：第三阶段时整个验证过程中最复杂的一个阶段，主要目的是通过数据流和控制流分析，确定程序语义是合法的、符合逻辑的。在第二阶段对元数据信息中的数据类型做完校验后，这个阶段将对类的**方法体**进行校验分析，例如：保证任意时刻操作数栈的数据类型与指令代码序列都能配合工作，保证跳转指令不会跳转到方法体以外的字节码指令上，保证方法体中的类型转换是有效的。
	- 符号引用验证：最后一个阶段的校验发生在虚拟机将符号引用转化为直接引用的时候，这个转化动作将在连接的第三阶段---解析中发生。符号引用验证可以看做是对类自身以外（常量池中的各种符号引用）的信息进行匹配性校验，通常需要校验以下内容：符号引用中通过字符串描述的全限定名是否能找到对应的类，在指定的类中是否存在方法的字段描述符以及简单名称所描述的方法和字段，符号引用中的类、字段、方法的访问性（private、protected、public、default）是否可以被当前类访问
- 准备：准备阶段是正式为类变量分配内存并设置类变量初始值的阶段，这些变量所使用的内存都将在方法区中进行分配。这个阶段中有两个容易混淆的概念需要强调一下，首先，这时候进行内存分配的仅包括类变量（被static修饰的变量），而不包括实例变量，实例变量将会在对象实例化时随着对象一起分配在Java堆中。其次，这里说的初始值“通常情况”下会是数据类型的零值，假设一个类变量的定义为
```java
	public static int value = 123;
```
那变量value在准备阶段过后的初始值是0而不是123，因为这时候尚未开始执行任何Java方法，而把value赋值为123的putstatic指令是程序被编译后，存放于类构造器<clinit>()方法之中，所有把value赋值为123的动作将在初始化阶段才会执行。在通常情况下初始值是零值，那相对的就有一些“特殊情况”，如果类字段的字段属性表中存在ConstantValue属性，那在准备阶段变量value就会被初始化为ConstantValue属性所指定的值，假设上面类变量value的定义变为
```java
	public static final int value = 123;
```
编译时Javac将会为value生成ConstantValue属性，在准备阶段虚拟机就会根据ConstantValue的设置将value赋值为123.
- 解析：解析阶段是虚拟机将常量池内的符号引用替换为直接引用的过程。解析动作主要针对类或接口、字段、类方法、接口方法、方法类型、方法句柄和调用点限定符七类符号引用进行。
	- 符号引用：符号引用以一组符号来描述所引用的目标，符号可以是任何形式的字面量，只要使用时能无歧义的定位到目标即可。
	- 直接引用：直接引用可以是直接指向目标的指针，相对偏移量或是一个能间接定位到目标的句柄。
- 初始化：类初始化阶段是类加载过程的最后一步，前面的类加载过程中，除了在加载阶段用户应用程序可以通过自定义类加载器参与之外，其余动作完全由虚拟机主导和控制。到了初始化阶段，才开始真正执行类中定义的Java程序代码（或者说是字节码）。在准备阶段，变量已经赋过一次系统要求的初始值，而在初始化阶段，则根据程序员通过程序制定的主观计划区初始化类变量和其它资源，或者可以用另一个角度来表达：初始化阶段是执行类构造器<clinit>()方法的过程。
	- <clinit>()方法与类的构造方法（或者说实例构造器<init>()方法）不同，它不需要显示地调用父类构造器，虚拟机会保证子类的<clinit>()方法执行之前，父类的<clinit>()方法已经执行完毕。因此在虚拟机中第一个被执行的<clinit>()方法的类肯定是java.lang.Object。
	- 由于父类的<clinit>()方法先执行，也就意味着父类中定义的静态语句块要优先于子类的变量赋值操作。
	- <clinit>()方法对于类或接口来说并不是必需的，如果一个类中没有静态语句块，也没有对变量的赋值操作，那么编译器可以不为这个类生成<clinit>()方法。
	- 接口中不能使用静态语句块，但仍然由变量初始化的赋值操作，因此接口与类一样都会生成<clinit>()方法。但不同的是，执行接口的<clinit>()方法不需要先执行父接口的<clinit>()方法。只有当父接口中定义的变量使用时，父接口才会初始化。另外，接口的实现类在初始化时也一样不会执行接口的<clinit>()方法。
	- 虚拟机会保证一个类的<clinit>()方法在多线程环境中被正确的加锁、同步，如果多个线程同时去初始化一个类，那么只有一个线程去执行这个类的<clinit>()方法，其它线程都需要阻塞等待，直到活动线程执行<clinit>()方法完毕。
	- "clinit>()方法"由编译器自动收集类中的所有变量的赋值动作和静态语句块（static{}中）的语句合并产生的，编译器收集的顺序是由语句在源文件中出现的顺序所决定的，静态语句块中只能访问定义在静态语句块之前的变量，定义在它之后的变量，在前面的静态语句块可以赋值，但是不能访问，如下面的例子所示：
	
```java
public class StaticTest {
	static {
		num = 0;  // 给变量赋值可以正常编译通过
		System.out.println(num); // 这句编译会提示“非法向前引用”
	}
	
	static int num = 1;
}
```

## 21、类与类加载器
类加载器虽然只用于实现类的加载动作，但它在Java程序中起到的作用却远远不限于类加载阶段。对于任意一个类，都需要由加载它的类加载器和这个类本身一同确立其在Java虚拟机中的唯一性，每一个类加载器，都拥有一个独立的类名称空间。这句话可以表达的更通俗一点：比较两个类是否相等，只有在这两个类是由同一类加载器加载的前提下才有意义，否则，即使这两个类来自于同一个Class文件，被同一个虚拟机加载，只要加载他们的类加载器不同，那这两个类必定不相等。

### 双亲委派模型
从Java虚拟机的角度来讲，只存在两种不同的类加载器：一种是启动类加载器（Bootstrap ClassLoader），这个类加载器使用C++语言实现，时虚拟机自身的一部分；另一种就是其它的类加载器，这些类加载器都有Java语言实现，独立于虚拟机外部，并且全都继承自抽象类java.lang.ClassLoader。  
从Java开发人员的角度来讲，类加载器还可以划分的更细致一点：
- 启动类记载器（Bootstrap ClassLoader）:负责将存放在$Java_Home\lib目录中的，或者被-Xbootclasspath参数所指定的路径中的，并且是虚拟机识别的类库加载到虚拟机内存中。启动类加载器无法被Java程序直接引用，用户在编写自定义类加载器时，如果需要把加载请求委派给引导类加载器，那直接使用null代替即可。
- 扩展类加载器（Extension ClassLoader）：这个加载器由sun.misc.Lancher$ExtClassLoader实现，它负责加载$Java_Home\lib\ext目录中的，或者被java.ext.dirs系统变量所指定的路径中的所有类库，开发者可以直接使用扩展类加载器。
- 应用程序类加载器（Application ClassLoader）：这个类加载器由sun.misc.Lancher$AppClassLoader实现。由于这个类加载器是ClassLoader中的getSystemClassLoader()方法的返回值，所以一般也称为系统类加载器，它负责加载用户路径（classpath）上所指定的类库，开发者可以直接使用这个类加载器，如果应用程序中没有自定义过类加载器，一般情况下这个就是程序中默认的类加载器。

## 双亲委派模型工作流程
如果一个类加载器收到了类加载的请求，它首先不会自己去尝试加载这个类，而是把这个请求委派给父类加载器去完成，每一个层次的类加载器都是如此，因此所有的类加载器请求最终都应该传送到顶层的启动类加载器中，只有当父加载器反馈无法完成这个加载请求（它的搜索范围中没有找到所需的类）时，子加载器才会尝试自己去加载。使用双亲委派模型来组织类加载器之间的关系，有一个显而易见的好处就是Java类随着它的类加载器一起具备了一种带有优先级的层次关系。例如类java.lang.Object，它存放在rt.jar中，无论哪一个类加载器要加载这个类，最终都是委派给处于模型最顶端的启动类加载器进行加载，因此Object类在程序的各种类加载器环境中都是同一个类。
![images](https://github.com/FantasyLakers/my-lessons/blob/master/java/%E5%8F%8C%E4%BA%B2%E5%A7%94%E6%B4%BE%E6%A8%A1%E5%9E%8B%E5%9B%BE.jpg)

### 双亲委派模型的破坏
- 双亲委派模型的第一次被破坏发生在双亲委派模型出现之前---即JDK1.2发布之前。由于双亲委派模型后出现，而类加载器和抽象类java.lang.ClassLoader则在JDK1.0时代就已存在，面对已经存在的用户自定义类加载器的实现代码，Java设计者引入双亲委派模型型不得不做出妥协。为了向前兼容，JDK1.2之后的java.lang.ClassLoader添加了一个新的protected方法findClass()，在此之前，用户去继承java.lang.ClassLoader的唯一目的就是重写loadClass方法，因为虚拟机在进行类加载的时候会调用加载器的私有方法loadClassInternal()，而这个方法的唯一逻辑就是去调用自己的loadClass()。JDK1.2以后已不提倡用户再去覆盖loadClass()方法，而应当把自己的类加载逻辑写道findClass()方法中，在loadClass()方法的逻辑里如果父类加载失败，则会调用自己的findClass()方法来完成加载，这样就可以保证新写出来的类加载器是符合双亲委派规则的。
- 双亲委派模型的第二次被破坏是由这个模型的缺陷所导致的。双亲委派模型很好的解决了各个类加载器的基础类的统一问题（越基础的类由越上层的加载器进行加载），基础类之所以称为基础，是因为它们总是作为被用户代码调用的API，但世事往往没有绝对的完美，如果基础类又要回调用户的代码，那该怎么办？一个典型的例子便是JNDI服务，JNDI现在已经是Java的标准服务，它的代码由启动类去加载，但JNDI的目的就是对资源进行集中管理和查找，它需要调用由独立厂商实现并部署在应用程序的ClassPath下的JNDI接口提供者（SPI，Service Provider Interface）的代码，但启动类加载器不可能“认识”这些代码，那该怎么办？为了解决这个问题，Java设计团队引入了一个不太优雅的设计：线程上下文类加载器（thread context classloader）。这个类加载器可以通过java.lang.Thread类的setContextClassLoader()方法进行设置，如果创建线程时还未设置，他会从父线程中继承一个，如果在应用程序的全局范围内都没有设置过的话，那这个类加载器默认就是应用程序类加载器。有了线程上下文类加载器，就可以做一些舞弊的事情了，JNDI服务使用这个线程上下文你加载器去加载所需要的SPI代码，也就是父类加载器请求子类加载器去完成类加载的动作，这种行为实际上就是打通了双亲委派模型的层次结构来逆向使用类加载器，实际上已经违背了双亲委派模型的一般性原则。Java中所有涉及SPI的加载动作基本上都采用了这种方式，例如JNDI、JDBC、JCE、JAXB和JBI等。
- 双亲委派模型的第三次被破坏是由于用户对程序动态性的追求而导致的，这里所说的动态性指的是---代码热替换（Hotswap）、模块热部署（hot deployment）等。
- OSGI的类加载：在OSGI环境下，类加载器不再是双亲委派模型中的树状结构，而是进一步发展为更加复杂的网状结构，当收到类加载请求时，OSGI将按照下面的顺序进行类搜索：
	- 将以java.*开头的类委派给父类加载器加载
	- 否则，将委派列表名单内的类委派给父类加载器加载
	- 否则，将Import列表中的类委派给Export这个类的Bundle的类加载器加载
	- 否则，查找当前Bundle的Classpath，使用自己的类加载器加载
	- 否则，查找类是否在自己的Fragment Bundle中，如果在，则委派给Fragment Bundle的类加载器加载
	- 否则，查找Dynamic Import列表的Bundle，委派给对应的Bundle的类加载器加载
	- 否则，类查找失败


## 22、java8中HashMap的变化
[Java8和Java7中HashMap的变化](https://www.cnblogs.com/jajian/p/10385063.html)

## 23、什么是AQS
[AQS的全称是AbstractQueuedSynchronizer：抽象队列同步器。](https://www.cnblogs.com/waterystone/p/4920797.html)

## 24、java8中ConcurrentHashMap的变化
[Java8之前ConcurrentHashMap的原理分析](https://blog.csdn.net/jiahao1186/article/details/83689241)
[Java8中的ConcurrentHashMap的原理分析](https://blog.csdn.net/u010723709/article/details/48007881)

## 25、Java设计原则
- 单一原则一个类只负责一项职责。
- 里氏替换原则
	* 子类可以实现父类的抽象方法，但是不能覆盖父类的非抽象方法
	* 子类中可以增加自己特有的方法
	* 当子类的方法重载父类的方法时，方法的形参要比父类方法的输入参数更宽松
	* 当子类的方法实现父类的抽象方法时，方法的返回值要比父类更严格
- 依赖倒置原则：高层模块不应该依赖低层模块，二者都应该依赖其抽象；抽象不应该依赖细节，细节应该依赖抽象。例如有一个方法tellStory，它的参数为Book实体类，而不是接口或抽象类，这是不应该的。
	```java
	public void tellStory(Book book){
		// tellStory
	}
	```
- 接口隔离原则：客户端不应该依赖他不需要的接口，一个类对另一个类的依赖应该建立在最小的接口上
- 迪米特法则（最少知道原则）：一个类要尽量的封装自己，一个类只与自己的朋友类打交道（朋友类：朋友类是成员变量或者方法参数，非朋友类一般都是局部变量）
- 开闭原则：对扩展开放，对修改闭合


## 26、动态代理
java.lang.reflect.Proxy：Proxy提供用于创建动态代理类和实例的静态方法，它还是由这些方法创建的所有动态代理类的超类。   
java.lang.reflect.InvocationHandler：是代理实例的调用处理程序实现的接口。对代理实例调用方法时，将对方法调用进行编码并将其指派到它的调用处理程序的 `invoke` 方法。  

```java
/**
 * 被代理接口
 */
public interface AccountService {
	
	void queryAccountBal();
	
}

/**
 * 被代理类
 */
public class DefaultAccountServiceImpl implements AccountService{

	@Override
	public void queryAccountBal() {
		System.out.println("my account bal is 100.10$");
	}

}



/**
 * 代理类，必须要继承InvocationHandler，并且被代理类必须实现某个接口
 */
public class ProxyHandler implements InvocationHandler{
	
	private Object obj;
	
	public ProxyHandler(Object obj){
		this.obj = obj;
	}
	
	public Object invoke(Object proxy, Method method, Object[] args)
			throws Throwable {
		System.out.println("before invoke");
		// 这里的obj其实是AccountService的动态代理类。这样可以在执行真正的DefaultAccountServiceImpl的queryAccountBal方法前后加入与业务逻辑无关的处理，比如事务控制，记录日志等等
		method.invoke(obj, args);
		System.out.println("after invoke");
		return null;
	}

}


public class TestProxy {

	public static void main(String[] args) throws Throwable {
		// 被代理类
		AccountService ps = new DefaultAccountServiceImpl();
		// 代理类
		InvocationHandler handler = new ProxyHandler(ps);
		// Proxy.newProxyInstance生成代理对象
		AccountService service = (AccountService) Proxy.newProxyInstance(AccountService.class.getClassLoader(), new Class[]{AccountService.class} , handler);
		service.queryAccountBal();
		
	}

}

```

## 27、静态代理

```java
/**
 * 公共接口
 */
public interface PeopleService {
	void sing();
}

/**
 *	被代理类，实现PeopleService公共接口
 */
public class PeopleServiceImpl implements PeopleService{

	public void sing() {
		System.out.println("people sing");
	}
	
}

/**
 * 代理类，也实现了PeopleService公共接口。但是真正逻辑由PeopleServiceImpl完成
 */
public class PeopleProxy implements PeopleService{
	
	private PeopleService people;
	
	public PeopleProxy(PeopleService people){
		this.people = people;
	}
	
	public void sing() {
        // 可以在这里实现业务无关的逻辑，如事务、日志等等
		System.out.println("代理类开始");
		people.sing();
		System.out.println("代理类结束");
	}

}

/**
 * 测试静态代理
 */
public class Test {

	public static void main(String[] args) {

		PeopleService PeopleService = new PeopleServiceImpl();
		PeopleProxy proxy = new PeopleProxy(PeopleService);
		proxy.sing();
	}

}
```

## 28、RMI

远程方法调用（remote method invocation）：能够让在客户端Java虚拟机上的对象像调用本地方法一样调用另一个Java虚拟机中的方法。

RMI远程调用步骤：

- 客户端调用客户端辅助对象stub上的方法
- 客户端辅助对象stub打包调用信息（变量、方法名），通过网络发送给服务端辅助对象skeleton
- 服务端辅助对象skeleton将客户端辅助对象发送来的信息解包，找出真正被调用的方法以及该方法所在对象
- 调用真正服务对象上的真正方法，并将结果返回给服务端辅助对象skeleton
- 服务端辅助对象skeleton将结果打包，发送给客户端辅助对象stub
- 客户端辅助对象stub将结果解析，返回给调用者
- 客户端获取返回值

RMI示例----服务端代码

```java
/**
 * 远程方法接口，必须继承Remote接口
 */
public interface CifQueryService extends Remote {

	// 由于方法都是远程调用的，可能出现通讯异常，所以所有方法都要抛出异常
	public void queryCifInfo() throws RemoteException;
	
	public String queryCifName() throws RemoteException;
}

/**
 *	远程方法实现一，需继承UnicastRemoteObject类，serialVersionUID序列化字段
 */
public class CifQueryServiceImpl extends UnicastRemoteObject implements
		CifQueryService {

	private static final long serialVersionUID = -285874256933727869L;

	// 由于UnicastRemoteObject类的构造函数抛出了RemoteException，
	// 故其继承类不能使用默认构造函数，继承类的构造函数也必须抛出RemoteException
	protected CifQueryServiceImpl() throws RemoteException {
		super();
	}

	@Override
	public void queryCifInfo() throws RemoteException {
		System.out.println("瓦里安");
	}

	@Override
	public String queryCifName() throws RemoteException {
		return "安度因";
	}

}

/**
 *	远程方法实现二，需继承UnicastRemoteObject类，serialVersionUID序列化字段
 */
public class DefaultCifQueryServiceImpl extends UnicastRemoteObject  implements CifQueryService {

	private static final long serialVersionUID = -2858742569337112869L;

	// 由于UnicastRemoteObject类的构造函数抛出了RemoteException，
	// 故其继承类不能使用默认构造函数，继承类的构造函数也必须抛出RemoteException
	protected DefaultCifQueryServiceImpl() throws RemoteException {
		super();
	}

	@Override
	public void queryCifInfo() throws RemoteException {
		System.out.println("古伊尔");
	}

	@Override
	public String queryCifName() throws RemoteException {
		return "萨尔";
	}

}

/**
 * 注册远程方法
 */
public class TestRmiServer {

	public static void main(String[] args) {
		CifQueryService cifqueryService1 = null;
		CifQueryService cifqueryService2 = null;
		try {
			cifqueryService1 = new CifQueryServiceImpl();
			cifqueryService2 = new DefaultCifQueryServiceImpl();
			/**
			 * 生成远程对象注册表Registry的实例，并指定端口为6666（默认端口是1099）
			 * 注册中心
			 */
			LocateRegistry.createRegistry(6666);
			/**
			 * Naming类提供在对象注册注册表中存储和获得远程对远程对象引用的方法
			 * Naming类的每个参数都可以讲某个名称作为其一个参数
			 * host:port/name
			 * host：注册表所在的主机（远程或本地)，省略则默认为本地主机
			 * port：是注册表接受调用的端口号，省略则默认为1099，RMI注册表registry使用的著名端口
			 * name：是未经注册表解释的简单字符串
			 */
			Naming.bind("rmi://127.0.0.1:6666/CifQuery1", cifqueryService1);
			Naming.bind("rmi://127.0.0.1:6666/CifQuery2", cifqueryService2);
		} catch (Exception e) {
			e.printStackTrace();
		}
		System.out.println("success");
	}

}
```



RMI示例----客户端代码

```java
/**
 *	调用RMI远程方法。CifQueryService在客户端项目中也需要有一份一样的，包括路径名
 */
public class ClientTestRmi {

	public static void main(String[] args) throws MalformedURLException, RemoteException, NotBoundException {
		// 在rmi服务注册表中查找对象，并调用其方法
		CifQueryService service1 = (CifQueryService) Naming.lookup("rmi://127.0.0.1:6666/CifQuery1");
		service1.queryCifInfo();
		System.out.println(service1.queryCifName());
		
		CifQueryService service2 = (CifQueryService) Naming.lookup("rmi://127.0.0.1:6666/CifQuery2");
		service2.queryCifInfo();
		System.out.println(service2.queryCifName());
	}

}
```



RMI的优点：

- 使用简单：只需要按照规范定义自己的服务对象即可

- 面向对象：RMI可将完整的对象作为参数和返回值进行传递，而不仅仅是预定义的数据类型。也就是说，可以将类似Java Hash表这样的复杂类型作为一个参数进行传递。

- 分布式：对象传递功能使你可以在分布式计算中充分利用面向对象技术的强大功能

- 安全性：RMI使用Java内置的安全机制保证下载执行程序时用户系统的安全。

  

RMI的缺点：

- 注册中心和服务提供者必须在同一台机器上，不支持分布式部署的要求
- 服务提供者宕机后，注册中心完全感知不到，没有服务可用性的检测机制
- 不支持重试机制
- 序列化效率不高
- 客户端每次请求前都要去注册中心获取最新的服务信息
- 只支持Java语言
- 所有的服务均要从注册中心获取，注册中心挂掉后，所以服务均不可用
- 服务端采用的是BIO的模式，效率上不如NIO



## 29、RPC

远程过程调用（remote procedure call）

调用步骤：

- 消费方client以调用本地方法的形式调用服务
- client stub接收到调用后负责将方法、参数等组装成能够进行网络传输的消息体
- client stub找到服务地址，并将消息发送到服务端
- server stub收到消息后进行解码，并调用本地服务
- 本地服务将结果返回给server stub，stub将结果发送至消费方
- client stub接收到消息，并进行解码
- 消费方得到最终结果


RPC的目标就是要把中间的步骤封装起来，让用户对这些细节透明化。要实现这些，就需要解决以下问题：

- 服务的注册与发现
- 网络通信
- 序列化和反序列化


RMI和RPC的区别：
- 方法调用方式不同：RMI调用方法，RMI中是通过在客户端的Stub对象作为远程接口进行远程方法的调用。每个远程方法都具有方法签名。如果一个方法在服务器上执行，但是没有相匹配的签名被添加到这个远程接口(stub)上，那么这个新方法就不能被RMI客户方所调用。
RPC调用函数，RPC中是通过网络服务协议向远程主机发送请求，请求包含了一个参数集和一个文本值，通常形成“classname.methodname(参数集)”的形式。这就向RPC服务器表明，被请求的方法在“classname”的类中，名叫“methodname”。然后RPC服务器就去搜索与之相匹配的类和方法，并把它作为那种方法参数类型的输入。这里的参数类型是与RPC请求中的类型是匹配的。一旦匹配成功，这个方法就被调用了，其结果被编码后通过网络协议发回。
- 适用语言范围不同： RMI只用于Java，支持传输对象。RPC是基于C语言的，不支持传输对象，是网络服务协议，与操作系统和语言无关。
- 调用结果的返回形式不同： RMI是面向对象的，Java是面向对象的，所以RMI的调用结果可以是对象类型或者基本数据类型。RPC的结果统一由外部数据表示(External Data Representation,XDR)语言表示，这种语言抽象了字节序类和数据类型结构之间的差异。只有由XDR定义的数据类型才能被传递，可以说RMI是面向对象方式的Java RPC。

## 30、虚拟机参数
-Xms：虚拟机堆区内存初始内存大小，通常为操作系统可用内存的1/64大小即可。-Xms1024m    
-Xmx：虚拟机堆区内存最大内存大小，通常为操作系统可用内存的1/4大小。-Xmx2048m    
-XX:MinHeapFreeRatio：设置堆空间的最小空间比例。当堆空间的空闲内存小于这个数值时，jvm便会扩展堆空间  
-XX:MaxHeapFreeRatio：设置堆空间的最大空间比例。当堆空间的空闲内存大于这个数值时，jvm便会缩小堆空间  
-XX:PermSize：表示非堆区初始内存分配大小，即永久区。-XX:PermSize=256m    
-XX:MaxPermSize：表示对非堆区分配的内存的最大上限。-XX:MaxPermSize=512m    
-Xss：设置线程栈的大小  
-XX:NewSize=n 设置年轻代大小  
-XX:NewRatio=n 设置年轻代和年老代的比例。假如值为3，表示年轻代和年老代比值为1:3  
-XX:+PrintGCDetails：打印gc日志  
-XX:+PrintHeapAtGC：在gc前后，都输出详细的堆信息 

## 31、深克隆、浅克隆

浅克隆：指拷贝对象时仅仅拷贝对象本身（包括对象中的基本变量），而不拷贝包含的引用指向的对象。

深克隆：不仅拷贝对象本身，而且拷贝对象包含的引用指向的所有对象。

```java
public class Wheel implements Cloneable{
	// 数量
	private int count;
	// 品牌
	private String type;

	public int getCount() {
		return count;
	}

	public void setCount(int count) {
		this.count = count;
	}

	public String getType() {
		return type;
	}

	public void setType(String type) {
		this.type = type;
	}

	@Override
	public String toString() {
		return "Wheel [count=" + count + ", type=" + type + "]";
	}
	
	  protected Object clone() throws CloneNotSupportedException{
		  return super.clone();
	  }
}

public class Car implements Cloneable {
	// 车名字
	private String name;
	// 车品牌
	private String type;
	// 价格
	private double amount;
	// 轮胎
	private Wheel wheel;

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public String getType() {
		return type;
	}

	public void setType(String type) {
		this.type = type;
	}

	public Wheel getWheel() {
		return wheel;
	}

	public void setWheel(Wheel wheel) {
		this.wheel = wheel;
	}

	public double getAmount() {
		return amount;
	}

	public void setAmount(double amount) {
		this.amount = amount;
	}

	@Override
	public String toString() {
		return "Car [name=" + name + ", type=" + type + ", amount=" + amount + ", wheel=wheelCount " + wheel.getCount()
				+ " wheelType " + wheel.getType() + "]";
	}

	  protected Object clone() throws CloneNotSupportedException{
		  
		  /**
		   * 深克隆
		   */
//		  Car car = (Car) super.clone();
//		  Wheel wheel = (Wheel) car.getWheel().clone();
//		  car.setWheel(wheel);
//		  return car;
		  
		  /**
		   * 浅克隆
		   */
		  return super.clone();
	  }
}

// 测试方法
public void testcopy() throws Exception {
		
		Car car = new Car();
		car.setAmount(1999.12d);
		car.setType("benz");
		car.setName("小张的车");
		
		Wheel wheel = new Wheel();
		wheel.setCount(4);
		wheel.setType("米其林");
		car.setWheel(wheel);

		/**
		 *  复制Car对象
		 */
		Car copyCar = (Car) car.clone();
		copyCar.setAmount(2000.00d);
		car.setType("bmw");
		copyCar.setName("二手车");
		
		copyCar.getWheel().setType("复制米其林");
		
		/**
		 * 可以看到浅克隆和深克隆时对象的基本属性值都复制了，且互不影响
		 */
		System.out.println(car);
		System.out.println(copyCar);
		
		/**
		 * 浅克隆：指拷贝对象时仅仅拷贝对象本身（包括对象中的基本变量），而不拷贝包含的引用指向的对象。
		 * 深克隆：不仅拷贝对象本身，而且拷贝对象包含的引用指向的所有对象。
		 */
		System.out.println(car == copyCar);
		System.out.println(car.getWheel() == copyCar.getWheel());
	}

// 浅克隆时的测试结果
Car [name=小张的车, type=bmw, amount=1999.12, wheel=wheelCount 4 wheelType 复制米其林]
Car [name=二手车, type=benz, amount=2000.0, wheel=wheelCount 4 wheelType 复制米其林]
false
true

// 深克隆的测试结果
Car [name=小张的车, type=bmw, amount=1999.12, wheel=wheelCount 4 wheelType 米其林]
Car [name=二手车, type=benz, amount=2000.0, wheel=wheelCount 4 wheelType 复制米其林]
false
false
```





## 32、线程状态，Blocked和Waiting的区别

- NEW：Thread state for a thread which has not yet started
- RUNNABLE：A thread in the runnable state is executing in the Java virtual machine but it may be waiting for other resources from the operating system such as processor.
- BLOCKED：A thread in the blocked state is waiting for a monitor lock to enter a synchronized block/method or reenter a synchronized block/method after calling
- WAITING：A thread in the waiting state is waiting for another thread to perform a particular action
  - Object.wait()
  - Thread.join()
- TIMED_WAITING：A thread is in the timed waiting state due to calling one of the following methods with a specified positive waiting time
  - Thread.sleep
  - Object.wait with timeout
  - Thread.join with timeout
  - LockSupport.parkNanos
  - LockSupport.parkUntil
- TERMINATED：Thread state for a terminated thread. The thread has completed execution



### Blocked和Waiting的区别

WAITING状态的线程是主动放弃cpu，并等待其它线程唤醒它。

BLOCKED状态的线程是被动进入阻塞状态的，该线程进入一个同步代码块时，但拿不到同步代码块的锁。
