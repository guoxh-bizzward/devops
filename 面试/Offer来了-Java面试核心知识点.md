[TOC]



# JVM

## JVM 运行机制

jvm是运行Java字节码的虚拟机,包括

* 字节码指令集
* 程序寄存器
* 虚拟机栈
* 虚拟机堆
* 方法区
* 垃圾回收区

Java程序的具体运行过程

1. java源文件被编译器编译成字节码文件
2. JVM将字节码文件编译成相应操作系统的机器码
3. 机器码调用相应操作系统的本地方法库执行相应的方法

Java虚拟机包括

* 类加载器子系统(Class loader SubSystem) 将编译好的.class文件加载到JVM中
* 运行时数据区(Runtime Data Area) 存储在JVM运行过程中产生的数据,包括程序计数器,方法区,本地方法区,虚拟机栈和虚拟机堆
* 执行引擎 包括即时编译器和垃圾回收器,即时编译器用于将java字节码编译成具体的机器码,垃圾回收器用于回收再运行过程中不再使用的对象
* 本地接口库(Native Interface Libary).本地接口库通过调用本地方法库(Native Method Libary)与操作系统交互.

## 多线程

在JVM后台运行的线程主要有以下几个:

* 虚拟机线程(JVM Thread): 虚拟机线程在JVM到达安全点(SafePoint)时出现;
* 周期性任务线程:通过定时器调线程来实现周期性操作的执行
* GC线程:GC线程支持JVM中不同的垃圾回收活动
* 编译器线程:编译器线程在运行时将字节码动态编译成本地平台机器码,是JVM跨平台的具体体现
* 信号分发器线程:接收发送到JVM信号并调用JVM方法

## JVM的内存区域

JVM内存区域分为线程私有区域(程序计数器,虚拟机栈,本地方法区),线程共享区域(虚拟机堆,方法区)和直接内存

直接内存又叫做堆外内存,它不是JVM运行时数据区的一部分.但是在并发编程中被频繁使用.JDK的NIO模块提供的基于channel和buffer的I/O操作方式就是基于堆外内存实现的,NIO通过调用native函数直接在操作系统上分配堆外内存,然后使用DirectoryByteBuffer对象作为这块内存的引用对内存进行操作

```
Java进程可以通过堆外内存技术避免在Java堆和Native堆中来回复制数据带来的资源占用和性能消耗,因此堆外内存在高并发应用场景下被广泛应用(Netty,Flink,HBase,Hadoop都有使用堆外内存) 
```



## JVM 的运行时内存

JVM运行时内存也叫做JVM堆,从GC的角度分为 新生代,老年代和永久代.新生代默认占1/3空间,老年代默认占2/3空间,永久代只占用很少的堆空间.

新生代又分为 Eden区(8/10),ServivorFrom(1/10)和ServivorTo(1/10)空间

* Eden: Java新创建的对象会 首先被存放到Eden区.如果新创建的对象属于大对象,直接分配到老年代.在Eden区内存空间不足的时候会触发MinortGC,对新生代进行一次垃圾回收.

```
大对象的定义根据具体的jvm版本,堆大小和垃圾回收策略有关,可以通过XX:PretenureSizeThreshold设置其大小.一般是2KB~128KB
```

* ServivorTo:保留上一次MinorGC时的幸存者
* ServivorFrom:将上一次MinorGC时的幸存者作为这一次MinorGC的被扫描者.

新生代的GC过程叫做Minor GC.采用复制算法实现.

* 把eden区和servivorFrom区中存活对象复制到servivorTo区.如果某对象的年龄到达老年代标准,则将其复制到老年代,同时把这些对象的年龄加1,如果servivorTo区内存不够,则也直接将其复制到老年代,如果对象属于大对象,也直接将其复制到老年代

```
对象晋升老年代的标准由 XX:MaxTenuringThreshold设置,默认是15
```

* 清空eden和servivorFrom区域的对象
* 将servivorTo区和servivorFrom区对换,原来的servivorTo区成为下一次GC时的ServivorFrom区

老年代主要存放有长生命周期的对象和大对象.老年代的GC过程叫做Major GC.MajorGC使用标记清除算法,该算法首先会扫描所有对象并标记存活对象,然后回收未被标记的对象,并释放内存空间.

```
在进行majorGC前,jvm会先进行一次Minor GC,如果MinorGC过后仍然出现老年代空间不足或者无法找到足够大的连续空间分配给新创建的大对象时,会触发Major GC进行垃圾回收.
Major会扫描所有老年代对象再回收,所以MajorGC的耗时比较长.同时MajorGC使用的标记清除算法容易产生内存碎片.在没有内存空间分配的时候,会报out of memory.
```

永久代指内存的永久保存区域,主要存放Class和meta(元数据)信息.Class在类加载时被放入永久代.GC不会在运行期间对永久代内存进行回收.所以永久代会随着Class的文件增加而增加,Class文件加载过多时会抛出Out of Memory.

```
在java8中,永久代被元数据区(元空间)取代.元数据区的作用和永久代类似,最大的区别是:
元数据区没有使用虚拟机的内存,而是直接使用操作系统的本地内存.因此,元空间的大小不受JVM内存限制,只和操作系统的内存有关.
在java8中,JVM将类的元数据放入本地内存(natvive memory)中,将常量池和类的静态变量放入java堆中.这样JVM能够加载多少元数据信息就不再由JVM的最大可用内存(MaxPermSize)空间决定,而是由操作系统的实际可用内存空间绝顶
```



## 垃圾回收与算法

### 如何确定垃圾

java采用引用计数器和可达性分析来确定对象是否可回收.

* 引用计数器方式容易产生循环依赖问题.
* 可达性分析 具体做法是首先定义一些GC Roots对象,然后以这些GC Roots对象作为起点向下搜索,如果在GC Roots和一个对象之间没有可达路径,则称该对象不可达.

```
不可达对象需要经过至少两次标记才能判定其是否可以被回收,如果在两次标记后该对象仍然是不可达的,则将被垃圾回收期回收.
```

### 常用垃圾回收算法

* 标记清除(Mark-Sweep)

分为标记和清除两个部分.在标记阶段标记所有需要回收的对象,在清除阶段清除可回收的对象并释放其所占的内存空间.

缺点:会引起内存碎片化问题,会引起大对象无法获得连续可用空间的问题

* 复制算法(Copying)

为了解决标记清除算法内存碎片化问题而设计的.

复制算法的内存清理效率高而易于实现,但是由于同一时刻只有一个内存区域可用,即可用的内存被压缩到原来的一半.因此存在大量的内存浪费.如果系统中有大量长时间存活对象时,在两个内存区域之间复制会影响系统效率.因此该算法只在对象"朝生夕死"状态下效率最高.

* 标记整理(Mark-Compact)

标记整理算法结合了标记清除算法和赋值算法的优点,其标记节点和标记清除算法节点相同,在标记完成后将存活对象移动到内存的另一端,然后清除该端对象并释放内存.

* 分代收集(Generational Collecting)

针对不同的对象类型,JVM采用了不同的垃圾回收算法,该算法被称为分代收集算法.

大部分JVM在新生代采用了复制算法;在老年代采用标记清除算法.

## Java中的4种引用类型

java对象的操作是通过该对象的引用(Reference)实现的

* 强引用: 有强引用的对象一定为可达性状态,所以不会被垃圾回收机制回收.因此,强引用是导致内存泄漏(Memory Link)的主要原因

* 软引用:通过SoftReference类实现,在系统内存空间不足时该对象将被回收;

* 弱引用:通过WeakReference类实现,如果对象只有弱引用,则在垃圾回收过程中一定会被回收;

* 虚引用:虚引用通过PhantomReference类实现,虚引用和引用队列联合使用,主要用于跟踪对象的垃圾回收状态.

## 分代收集算法和分区收集算法

### 分代收集算法

* 新生代 与复制算法
* 老年代与标记整理算法

### 分区收集算法

分区算法将整个堆空间划分为连续的大小不同的区域,对每个小区域都单独进行内存使用和垃圾回收,这样做的好处是可以根据每个小区域内存的大小灵活使用和释放内存.

```
分区收集算法可以根据系统可接受的停顿时间,每次都快速回收若干个小区域的内存 ,以缩短垃圾回收的停顿时间,最后以多次并行累加的方式逐步完成整个内存区域的垃圾回收.
```

## 垃圾收集器

### 针对新生代提供的垃圾收集器

* Serial 单线程,复制算法

基于复制算法实现,是一个单线程收集器.在它进行垃圾收集时,必须单暂停其他所有工作线程,知道垃圾收集结束.

Serial 是Java虚拟机运行在Client模式下新生代的默认垃圾收集器



* ParNew 多线程,复制算法

ParNew是Serial的多线程实现;在垃圾收集过程中会暂停其他所有工作线程,是Java虚拟机运行在Server模式下新生代的默认垃圾收集器

```
ParNew垃圾收集器默认开启与CPU同等数量的线程进行垃圾回收.在Java启动时可以通过 -XX:ParallelGCThreads参数调节ParNew垃圾收集器的工作线程数.
```



* Parallel Scavenge 多线程,复制算法

为了提高新生代垃圾收集效率而设计的垃圾收集器,基于多线程复制算法实现,在系统吞吐量上有很大的优化.可以更高效的利用CPU尽快完成垃圾回收任务.

Parallel Scavenge 通过自适应调节策略提高系统吞吐量,提供三个参数用于调节,控制垃圾回收的停顿时间和吞吐量

XX:MaxGCPauseMillis 控制最大垃圾收集停顿时间

XX:GCTimeRatio 控制吞吐量大小

UseAdaptiveSizePolicy 控制自适应调节策略开启与否



### 针对老年代提供的垃圾收集器

* Serial Old: Serial垃圾收集器的老年代实现

单线程,标记整理算法.

Serial Old是JVM运行在Client模式下的老年代的默认垃圾收集器



* Parallel Old 多线程,标记整理算法

Parallel Old在设计上优先考虑系统吞吐量,其次考虑停顿时间等因素.如果系统对吞吐量的要求比较高,可以使用新生代  Parallel Scavenge  + 老年代 Parallel Old.



* CMS

主要目的是达到最短的垃圾回收停顿时间,基于线程的标记清理算法实现,以便在多线程并发环境下以最短的垃圾收集停顿时间提高系统的稳定性.

CMS的工作机制相对复杂,垃圾回收过程包含如下4个步骤

1. 初始标记
2. 并发标记
3. 重新标记
4. 并发清除

```
 CMS垃圾收集器在和用户线程一起工作时(并发标记和并发清除)不需要暂停用户线程,有效缩短了垃圾回收时系统的停顿时间,同时由于CMS垃圾收集器和用户线程一起工作,因此其并行和效率也有很大提升.
```



### 针对不同区域的G1分区收集算法

G1 (Garbage First)

G1相对于CMS有两个突出改进

* 基于标记整理算法,不产生内存碎片
* 可以精确的控制停顿时间,在不牺牲吞吐量的前提下实现短停顿垃圾回收;

## Java 网络编程模型

### 阻塞I/O 

阻塞I/O模型的工作流程为:在用户线程发出I/O请求后,内核会检查数据是否就绪,此时用户线程会一直堵塞等待内存数据就绪,在内存数据就绪后,内核将数据复制到用户线程,并返回I/O执行结果到用户线程,此时用户线程解除阻塞状态并开始处理数据.

```
典型的阻塞I/O模型的例子是
data = socket.read();
如果内核数据没有就绪,socket线程就会一直阻塞在read()中等待内核数据就绪.
```

### 非阻塞I/O

非阻塞I/O模型是指用户线程在发起一个I/O操作后,无需阻塞便可以马上得到内核的一个结果.

```
在非阻塞I/O模型中,用户线程需要不断的轮训内核数据是否就绪,在内存数据还未就绪时,用户线程可以处理其他任务,在内核数据就绪后可以立即获取数据并进行相应的操作.
典型的非阻塞I/O模型一般如下
while(true){
	data = socket.read();
	if(data == true){
		//获取并处理内核数据
		break;
	}else{
		//内核数据还未就绪,处理其他任务
	}
}
```

### 多路复用I/O模型

多路复用I/O模型是多线程并发编程用的较多的模型,Java NIO就是基于多路复用I/O模型实现的.在多路复用I/O模型中会有一个被称为Selector的线程不断轮询多个Socket状态,只有在Socket状态有读写事件时,才会通知用户线程进行I/O读写操作.

多路复用I/O模型在连接数众多且消息体不大的情况下有很大的优势.**经过优化后的16核32G服务器能承载约10万台设备连接**

非阻塞I/O是在每个用户线程中都进行Socket状态检查,而在多路复用I/O模型中是在系统内核中进行Socket状态检查的,这也是多路复用I/O模型比非阻塞I/O模型效率高的原因

```
多路复用I/O模型在事件响应体(消息体)很大时,selector线程就会成为性能瓶颈,导致后续的事件迟迟得不到处理,影响下一轮的事件轮训.在实际应用中,在多路复用方法体内一般不建议做复杂逻辑运算,只做数据的接收和转发,将具体业务操作转发给后面的业务线程处理.
```

### 信号驱动I/O模型

在用户发起一个I/O请求操作时,系统会为该请求对应的socket注册一个信号函数,然后用户线程可以继续执行其他业务逻辑,在内核数据就绪后,系统会发送一个信号到用户线程,用户线程接收到信号后,会在信号函数中调用的I/O读写操作完成实际的I/O请求操作.

### 异步I/O模型



## JVM 类加载机制

### 类加载器

* 启动类加载器  JAVA_HOME/lib 或 Xbootclasspath
* 扩展类加载器 JAVA_HOME/ext/lib 或者 使用 java.ext.dirs
* 应用程序类加载器 加载用户路径 classpath上的类库

### 双亲委派

保障类的唯一性和安全性

# Java基础

### 集合

* List
  * ArrayList
  * LinkedList
  * Vector
* Queue
  * ArrayBlockingQueue
  * LinkedBlockingQueue
  * PriorityBlockingQueue
  * DelayQueue
  * SychronousQueue
  * LinkedTransferQueue
  * LinkedBlockingDueue
* Set
  * HashSet
  * TreeSet
  * LinkedSet
* Map
  * HashMap
  * HashTable
  * TreeMap

### 异常分类以及处理

Throwable

​	Error

​	Exception

​		checked Exception SQLException IOException FileNotFound...

​		RuntimeException eg NullPointer ClassCast ArrayIndexOutBounds...

### 反射机制

Java反射创建对象的两种方式

1. 使用newInstance创建该Class对象的对应实例,需要Class对象有默认的无参构造函数
2. 先使用Class对象获取指定的constructor对象,再调用constructor对象的newInstance方法创建class对象.

### 注解

@Target

@Retention

@Documented

@Inherited

### 内部类

静态内部类

成员内部类

局部内部类

匿名内部类

### 泛型

泛型标记和泛型限定 E T K V N ?

* E Element 在集合中使用,表示集合中存放的元素
* T Type 表示Java类中的基本类和自定义类型
* K Key 表示键 比如Map中的Key
* V value 表示值
* N Number 数值类型
* ? 不确定的Java类型
* List<? extends T> 上限 表示类都是 T的子类
* List<? super T> 下限 表示类都是 T的父类

类型擦除



### 序列化

对象序列化保存的是对象的状态,即它的成员变量,所以类中的静态变量不会保存;

# Java 并发编程

## Java线程的创建方式

继承Thread类

实现Runnable接口

通过ExecutorService和Callable<Class>实现有返回值的线程;

基于线程池;

## 线程池工作原理

JVM先根据用户的参数创建一定数量的可运行的线程任务,并将其放入队列中,在线程创建后启动这些任务,如果线程数量超过最大线程数量,则超出数量的线程排队等候,在有任务执行完毕后,线程池调度器会发现有可用线程,进入再次从队里中取出任务并执行;

线程池的主要作用是线程服用,线程资源管理,控制操作系统的最大并发数;

Java线程池主要由以下4个核心组件组成

* 线程池管理器 用于创建并管理线程池
* 工作线程 线程池中执行具体任务的线程
* 任务接口 用于定义工作线程的调度和执行策略,只有线程实现了该接口,线程中的任务才能够被线程池调度
* 任务队列 存放待处理的任务

Java中的线程池是通过Executor框架实现的,在该框架中用到了Executor,Executors,ExecutorService,ThreadPoolExecutor,Callabe,Future,FuterTask这几个核心类.

线程池拒绝策略

* AbortPolicy 直接抛出异常,拒绝执行
* CallerRunsPolicy 如果被丢弃的线程任务未关闭,则执行该线程任务(由当前线程执行被丢弃的任务,会导致主线程阻塞);
* DiscardOldestPolicy 移除线程队列中最早的一个线程任务,并尝试提交当前任务
* DiscardPolicy 丢弃当前的线程任务而不做任何处理
* 自定义拒绝策略

### 常用线程池

* `newCachedThreadPool`

* `newFixedThreadPool`

* `newScheduledTheadPool`

* `newSingleThreadExecutor`

* `newSingleThreadScheduledExecutor`

* `newWorkStealingPool`

### 线程的生命周期

* `New` 新建状态

* `Runable` 就绪状态
* `Running` 运行态
* `Blocked` 阻塞状态

运行中的线程会主动或被动的放弃CPU使用权并暂停运行,此时该线程转为阻塞状态,直到再次进入可运行状态,才有机会竞争到CPU使用权并转为运行态.阻塞的状态分为三种

1. 等待阻塞 运行态线程调用o.wait方法,JM会把该线程放入等待队列(Waiting Queue)中,线程转为阻塞状态
2. 同步阻塞 在运行状态的线程尝试获取正在被其他线程占用的对象锁时,JVM会把该线程放入锁池(Lock Pool)中,此时线程转为阻塞状态
3. 其他阻塞 运行状态线程执行sleep(),join()或者发出IO请求时,JVM会把该线程转为阻塞状态,直到sleep超时,join()等待线程终止或超时,或者IO处理完毕,线程才重新弄转为可运行状态.

* Dead 死亡

1. 线程正常结束 run方法或者call方法执行完毕;
2. 线程异常退出 抛出Error 或者未捕获异常
3. 手动结束 调用线程对象的stop方法手动结束运行中的线程(该方式会瞬间释放线程占用的同步对象锁,导致锁混乱和死锁,不推荐使用)

### 线程的基本方法

* wait 在调用wait方法后会释放对象的锁,所以wait方法一般被用于同步方法或同步代码块中,线程会进入 WAITING 状态

* sleep sleep方法不会释放当前占有的锁,会导致线程进入 TIME-WAITING状态

* yield 线程让步 调用 yield方法会使当前线程让出(释放)CPU执行时间片,与其他线程一起竞争CPU时间片.

* interrupt 线程中断 该方法用于向线程发出一个终止通知信号,会影响该线程内部的一个中断标识,这个线程本身并不会因为调用了Interrupt方法而改变状态(阻塞,终止等).状态的具体变化需要等待接收到中断标识的程序的最终处理结果来判定.对interrupt方法的理解要注意以下4个核心点

  * 调用interrupt方法并不会中断一个正在运行的线程,也就是说处于running状态的线程并不会因为中断而终止,仅仅改变了内部维护的中断标示位而已.具体的jdk源码如下

  ```
  public static boolean interrupted(){
  	return currentThread().isInterrupted(true);
  }
  public boolean isInterrupted(){
  	return isInterrupted(false);
  }
  ```

  * 若因为调用sleeep方法而使线程处理time_waiting状态,则这时调用interrupt方法会抛出interruptedException,使线程提前结束Time_waiting状态
  * 许多声明抛出interruptException的方法比如Thread.sleep(),在抛出异常前都会清除中断标示位,所以在抛出异常后调用isInterrupted方法将会返回false
  * 中断状态是线程固有的一个标示位,可以通过此标示位安全的终止线程.比如在想终止一个线程时,可以先调用该线程的interrupt方法,然后在线程的run方法中根据该线程的interrupted方法的返回状态值安全终止线程

  ```
  
  ```

  

* join 线程加入 join方法用于等待其他线程终止,如果在当前线程中调用一个线程的join方法,则当前线程转为阻塞状态,等到另一个线程结束,当前线程再由阻塞状态转为就绪状态,等待获取CPU使用权.线程执行完成后,会触发notify操作,这块的实现是在native方法(hotspot 源码)中实现的

```
//很多情况下,主线程生成并启动了子线程,需要等到子线程返回结果并收集和处理再退出.这时就要用到join方法,具体的使用方法如下
System.out.println("子线程运行开始");
ChildThread childThread = new ChildThread();
childThread.join();//等待子线程childThread执行结束
System.out.println("子线程结束,开始运行主线程");
```



* notify 线程唤醒 Object对象方法.用于唤醒在此对象监视器上等待的一个线程,如果所有线程都在此对象上等待,则会选择唤醒其中一个线程,选择是随机的.

```
我们通常调用其中一个对象的wait方法在此对象监视器上等待,直到当前线程放弃此对象上的锁定,才能继续执行被唤醒的线程,被唤醒的线程将以常规方式与此对象上主动同步的其他线程竞争.类似的方法还是notifyAll,用于唤醒在监听器上等待的所有线程.
```

* setDaemon 后台守护线程 也叫服务线程 这种线程有一个特性即为用户提供公共服务,在没有用户线程可服务时会自动离开

```

```

* sleep wait 区别
  * sleep属于Thread类,wait属于Object类
  * sleep方法暂停执行指定时间,让出CPU给其他线程,但其监控状态依然保持,在指定时间过后又会自动恢复运行状态
  * 在调用sleep方法的过程中,线程不会释放对象锁
  * 在调用wait方法时,线程会放弃对象锁,进入等待此对象的等待锁池,只有针对此对象调用notify方法后,该线程才能进入对象锁池获取对象锁,并进入运行状态
* start 和run方法的区别
  * start属于启动线程,真正实现了多线程.在调用了线程的start方法后,线程会在后台执行,无须等待run方法体的代码执行完毕,就可以继续执行下面的代码
  * 通过调用thread类的start方法启动一个线程时,此线程处于就绪状态,并没有运行
  * run方法也叫做线程体,包含了要执行的线程的逻辑代码,在调用run犯法后,线程就进入运行状态,开始运行run方法中的代码.在run方法运行结束后,该线程终止,CPU再调度其他线程.

#### 终止线程的4种方式

1. 正常结束
2. 使用退出标志退出线程
3. 使用interrupt方法终止线程
4. 使用stop方法终止线程,不安全不推荐

## Java 中的锁

Java中的锁主要用于保障多并发线程情况下数据的一致性.

锁从乐观和悲观的角度分为乐观锁和悲观锁,从获取资源的公平性角度分为公平锁和非公平锁,从是否共享的角度分为共享锁和独占锁,从锁的状态角色可以分为偏向锁,轻量级锁和重量级锁.

在JVM中还巧妙设计了自旋锁以更快的使用CPU资源.

### 乐观锁

大部分是通过CAS(Compare And Swap,比较和交换)实现

### 悲观锁

大部分是通过AQS(Abstract Queued Synchronized,抽象的队列同步器)架构实现

```
AQS定义了一套多线程访问共享资源的同步框架,许多同步类的实现都依赖于他,例如常用的Synchronized,ReentrantLock,Semphore,CountDownLatch等.该框架下的锁会先尝试以CAS乐观锁去获取锁,如果获取不到,则会转为悲观锁(比如RetreenLock)
```

### 自旋锁

自旋锁因为如果持有锁的线程在很短的时间内释放资源,那么那些等待竞争的线程就不需要做内核和用户态之间的切换进入阻塞,挂起状态,只需要等一等(自旋),在等待持有锁的线程释放锁后立即获取锁,这样就避免了用户线程在内核状态的切换上导致的锁时间消耗.

线程在自旋的时候会占用CPU,在线程长时间自旋获取不到锁时,会产生CPU浪费,甚至有时候线程永远无法获取锁而导致CPU资源被永久占用,所以需要设定一个自旋等待的最大时间.在线程执行的时间超过自旋等待的最大时间后,线程会退出自旋模式并释放其持有的锁.

自旋锁的优缺点

* 优点 自旋锁可以减少CPU上下文切换,对于占用锁的时间非常短或锁竞争不激烈的代码块来说性能大幅度提升,因为自旋的CPU耗时明显少于线程阻塞,挂起,再唤醒时两次CPU上下文切换所用的时间.

* 缺点 在持有锁的线程占用锁时间或者锁的竞争过于激烈时,线程在自旋过程中会长时间获取不到锁资源,将引起CPU浪费.所以在系统中复杂锁依赖的情况下不适合采用自旋锁.

自旋锁的时间阈值

JDK不同的版本所采用的自旋周期不同,JDK1.5为固定时间,JDK1.6引入适应性自旋锁.

适应性自旋锁的自旋时间不再是固定,而是由上一次在同一个锁上的自旋时间及锁的拥有者的状态来决定的,可基本认为一个线程上下文切换的时间就是最佳时间.

### Synchronized

synchronized属于独占式的悲观锁,同时属于可重入锁.

Java中每个对象都有个monitor对象,加锁就是在竞争monitor对象.对代码块加锁是通过在前后分别加monitorenter和monitorexit指令实现的,对方法是否加锁是通过一个标记位来判断的.

#### synchronized的作用范围

* synchronized作用于成员变量和非静态方法时,锁住的是对象的实例,即this对象
* synchronized作用于静态方法时,锁住的是Class实例,因为静态方法属于Class而不属于对象
* synchronized作用于一个代码块时,锁住的是所有代码块中配置的对象.

#### synchronized的实现原理

在synchronized内部包括ContentionList,EntryList,WaitSet,OnDeck,Owner,!Owner 6个区域,每个区域的数据都代表锁的不同状态

* ContentionList 锁竞争队列,所有请求锁的线程都被放在竞争队列中
* EntryList 竞争候选者列表 在ContentionList中有资格成为候选者来竞争锁资源的线程被移动到EntryList中
* WaitSet 等待集合,调用wait方法后被阻塞的线程将被放在WaitSet集合中 (notify/notifyAll唤醒的是这里的线程)
* OnDeck 竞争候选者 在同一时刻最多只有一个线程在竞争锁资源,该线程的状态为OnDeck
* Owner 竞争到锁资源的线程被称为Owner状态线程
* !Owner 在Owner线程释放锁后,会从Owner变成 !Owner

synchronized在收到新的锁请求时会首先自旋,如果通过自旋也没有获取到锁资源,则将被放入锁竞争队列ContentionList中.该方式对已经进入队列的线程是不公平的,因此synchronized是非公平锁.另外,自旋获取锁的线程也可以直接抢占OnDeck线程的锁资源.

```
为了防止锁竞争时ContentionList尾部的元素被大量的并发线程CAS访问而影响性能,Owner线程在释放锁资源的时候将ContentionList中的部分线程移动到EntryList中,并制定EntryList中的某个线程(一般是最先进入的线程)为OnDeck线程.Onwer并不会直接把锁传递个OnDeck线程,而是把锁竞争的权利交给Ondeck.让Ondeck去重新竞争锁.在Java中把该行为成为"竞争切换",该行为牺牲了公平性,但提高了性能.
获取到锁资源的Ondeck线程会变成Owner线程,而未获取到锁资源的线程仍然停留在EntryList中.
Owner线程在被wait方法阻塞后,会被转移到WaitSet队列中,直到某个时刻被notify/notifyAll唤醒,会再次进入EntryList中.ContentionList,EntryList,WaitSet中的线程均为阻塞状态,该阻塞是由操作系统完成(在Linux内核下采用pthread_mutex_lock内核函数完成)
```



synchronized是一个重量级锁,需要调用操作系统的相关接口,性能较低,给线程加锁的时间有可能超过获取锁后具体逻辑代码的操作时间;

JDK1.6对synchronized做了很多优化,引入了适应自旋,锁消除,锁粗化,轻量级锁以及偏向锁等以提高效率.锁可以从偏向锁升级到轻量级锁,再升级到重量级锁.这个升级过程叫做锁膨胀.

在JDK1.6中默认开启了偏向锁和轻量级锁,可以通过`-XX:UseBiasedLocking` 禁用偏向锁.

### ReentrantLock

ReentrantLock继承Lock接口并实现了在接口中定义的方法.是一个可重入的独占锁.ReentrantLock通过自定义队列同步器(AQS)来实现锁的获取和释放.(实际实现是通过实现了AbstractQueuedSynchronizer的Sync内部类)

```
独占锁是指该锁在同一时刻只能被一个线程获取,而获取锁的其他线程只能在同步队列中等待;
可重入锁时指该锁能够支持一个线程对同一资源执行多次加锁操作;
```

ReentrantLock支持公平锁和非公平锁.

ReentrantLock不仅支持synchronized对锁的操作功能,还提供了诸如可响应中断锁,可轮询锁请求,定时锁等避免多线程死锁的情况.

ReentrantLock有显示的操作过程,何时加锁,何时释放锁都在程序员的控制之下.

```

```

注意:获取锁和释放锁的次数要相同

如果释放锁的次数>获取锁的次数,报 java.lang.IllegalMonitorStateException;

如果释放锁的次数<获取锁的次数,该线程会一直持有锁,其他线程将无法获取锁资源.

#### ReentrantLock如何避免死锁

响应中断,可轮询锁,定时锁

* 响应中断 在synchronized中如果有一个线程尝试获取锁,则结果是要么获取锁继续执行,要么保持等待.ReentrantLock还提供了可响应中断的可能,即在等待锁的过程中,线程可以根据需要取消对锁的请求.
* 可轮询锁 通过`boolean tryLock()`获取锁.如果有可用锁,则获取该锁并返回true,如果无可用锁,则立即返回false
* 定时锁 通过`boolean tryLock(long time,TimeUnit timeUnit)`获取定时锁.如果在给定的时间内获取到了可用锁,且当前线程未被中断,则获取到锁并返回true.如果在给定的时间内获取不到可用锁,将禁用当前线程,并且在发生以下三种情况之前,该线程将一直处于休眠状态
  * 当前线程获取到了可用锁并返回true
  * 当前线程在进入此方法时设置了该线程的中断状态,或者单签线程在获取锁时被中断,则抛出InterruptedException,并清除当前线程的中断状态
  * 当前线程获取锁的时间超过指定的等待时间,则将返回false.如果设定的时间小于等于0,则该方法将完全不用等待;

#### Lock接口的主要方法

* void lock()
* boolean tryLock()
* tryLock(long timeout,TimeUnit timeUnit)
* void unlock
* Condition newCondition()
* getHoldCount()
* getQueueLength()
* getWaitQueueLength(Condition condition)
* hasWaiters(Condition condition)
* hasQueuedThread(Thread thread)
* hasQueuedTHreads()
* isFair()
* isHeldByCurrentThread()
* isLock()
* lockInterruptibly()

#### tryLock lock 和 lockInteruptibly



### Synchronized VS ReentrantLock





### Semaphore



### CountDownLatch



### AtomicInteger



### 可重入锁



### 公平锁 非公平锁



### 读写锁 ReadWriteLock



### 共享锁和独占锁



### 重量级锁和轻量级锁



### 偏向锁



### 分段锁



### 同步锁和死锁



### 如何进行锁优化





### 线程上下文切换



### Java 阻塞队列



### Java并发关键字



### 多线程共享数据



### ConcurrentHashMap并发



### Java 线程调度



### 进程调度算法



### CAS



### ABA问题



### AQS

# 数据结构



# 算法

# 网络与负载均衡

# 数据库与分布式事务

# 分布式缓存的原理和应用

缓存分为进程级缓存和分布式缓存，进程级缓存是指Map List等结构实现的存储；分布式缓存则是通过Ehcache,redis，menCached实现。

## Ehcached



## Redis

支持多种数据类型 String，Hash,List，Set，Set,BitMap,HyperLogLog,Geospatial.

内置复制脚本，Lua脚本，LURU驱动事件，事务和不同级别的磁盘持久化

通过redis哨兵和Cluster模式提供高可用

### Redis数据类型

### Redis管道

在分布式环境下,Redis的性能瓶颈主要体现在网络延迟上.

Redis的管道技术指在服务端未响应时,客户端可以继续向服务端发送请求,最终一次性读取所有服务端的响应.

### Redis事务

Redis的事务操作分为 开启事务->命令入队列->执行事务.

* 事务开启 客户端执行Multi开启事务
* 提交请求 客户端提交命令到事务
* 任务入队列 redis将客户端请求放入事务队列中等待执行
* 入队状态反馈 服务端返回QURUD,说明命令已经进入事务队列
* 执行命令 客户端通过Exec执行事务
* 事务执行错误 在redis事务中如果某条命令执行错误,则其他命令可以继续执行,不会回滚;可以通过通过watch监控事务执行的状态并处理命令执行错误的异常情况
* 执行结果反馈 服务端向客户端返回事务执行的结果

事务相关的命令有 Multi,Exec,Discard,Watch,Unwatch

* Multi 标记一个事务块的开始
* Exec 执行所有事务块内的命令
* Discard 取消事务,放弃执行事务块内的所有命令
* Watch 监视一个(或者多个)key,如果事务执行之前这个(或者这些)key被其他命令改动,那么这个事务将被打断
* Unwatch 取消watch命令对所有key的监视

```
    public void transactionSet(Map<String, Object> commandList) {
        //1 开启事务权限
        redisTemplate.setEnableTransactionSupport(true);
        try {
            //2. 开启事务
            redisTemplate.multi();
            //3. 执行事务命令
            for (Map.Entry<String, Object> entry : commandList.entrySet()) {
                String mapKey = entry.getKey();
                Object mapValue = entry.getValue();

                redisTemplate.opsForValue().set(mapKey, mapValue);
            }
            // 4. 成功就提交
            redisTemplate.exec();
        } catch (Exception e) {
            //5. 失败就回滚
            redisTemplate.discard();
        }
    }
```



### Redis发布,订阅

### Redis集群数据复制的原理

* 一个从数据库在启动后,会向主数据发送SYNC命令
* 主数据库在接收SYNC命令后开会开始在后台保存快照(RDB持久化过程),并将保存快照期间接收到的命令缓存起来.在该持久化过程中会生成一个.rdb快照文件.
* 在主数据库快照执行完成后,redis会将快照文件和所有缓存的命令以.rdb快照文件的形式发送给从数据库
* 从数据库接收到主数据库的.rdb快照文件后,载入该快照文件到本地
* 从数据库执行载入后的.rdb快照文件,将数据写入内存.以上过程被称为复制初始化
* 在复制初始化结束后,主数据库在每次收到写命令时都会将命令同步给从数据库,从而保证主从数据库的数据一致

### Redis持久化

redis持久化有两种 AOF RDB

* RDB (redis database)
* AOF(Append of File)

### Redis集群模式及工作原理

redis集群有三种模式 主从,哨兵,集群;

## 分布式缓存设计的核心问题

分布式缓存设计的核心问题是以哪种方式进行缓存预热和缓存更新,以及如何优雅的解决缓存雪崩,缓存穿透,缓存降级等问题;

### 缓存预热

缓存预热一般有系统启动加载,定时加载等方式;

### 缓存更新

缓存更新是指在数据发生变化后及时将变化后的数据更新到缓存中.常见的缓存更新策略有以下4种.

* 定时更新
* 过期更新
* 写请求更新
* 读请求更新

### 缓存淘汰策略

* FIFO
* LRU
* LFU

### 缓存雪崩

* 请求加锁
* 失效更新
* 设置不同的失效时间

### 缓存穿透

* 布隆过滤器
* cache null 策略

### 缓存降级

* 写降级
* 读降级

# 设计模式

设计模式的7个原则

* 单一职责原则
* 开闭原则
* 里式代换原则
* 依赖倒转原则
* 接口隔离原则
* 合成/聚合复用原则
* 迪米特法则

设计模式按照功能和使用场景可以分为三类

* 创建型

工厂模式,抽象工厂模式,单例模式,创建型模式,原型模式

* 结构型

适配器模式,桥接模式,组合模式,装饰器模式,外观模式,享元模式,代理模式

* 行为型

责任链模式,命令模式,解释器模式,迭代器模式,中介者模式,备忘录模式,观察者模式

状态模式,策略模式,模板模式,访问者模式  