---
title: JVM
date: 2020-02-27 19:27:36
tags: java
category: java
---

# Java内存区域

## 前提

本文讲的基本都是以Sun HotSpot虚拟机为基础的，Oracle收购了Sun后目前得到了两个【Sun的HotSpot和JRockit(以后可能合并这两个),还有一个是IBM的IBMJVM】

![](jvm0.png)

## Java内存模型

Java程序内存的分配是在JVM虚拟机内存分配机制下完成。

Java内存模型（Java Memory Model,JMM）就是一种符合内存模型规范的，屏蔽了各种硬件和操作系统的访问差异的，保证了Java程序在各种平台下对内存的访问都能保证效果一致的机制及规范。

简而言之，JMM是jvm的一种规范，定义了JVM的内存模型。她屏蔽了各种硬件和操作系统的访问差异，不是直接访问硬件内存，相对安全很多，它的主要目的是解决由于多线程通过共享内存进行通信时，存在的本地内存数据不一致，编译器会对代码指令重排序，处理器会对代码乱序执行等带来的问题。可以保证并发编程场景中的原子性、可见性和有序性。

Java数据区域分为为五大数据区域。这些区域各有各的用途，创建及销毁时间。

![](jvm1.png)

## 五大内存区域详解

### 程序计数器

程序计数器是一块很小的内存空间，它是线程私有的，可以认作为当前线程的行号指示器。

为什么需要程序计数器？

我们知道对于一个处理器（如果是多核cpu那就是一核），在一个确定的时刻都只会执行一条线程中的指令，一条线程中有多个指令，为了线程切换可以恢复到正确执行位置，每个线程都需要有一个独立的程序计数器，不同线程之间的程序计数器互不影响，独立存储。

如果线程执行的是个java方法，那么计数器记录虚拟机字节码指令的地址。如果为native方法，那么计数器为空。这块内存区域是虚拟机规范中唯一没有OutOfMemoryError的区域。

### Java栈（虚拟机栈）

同计数器一样也为线程私有，生命周期与之相同，就是我们平时说的栈，栈描述的是Java方法执行的内存模型。

每个方法被执行的时候都会创建一个栈帧用于存储局部变量表，操作栈，动态链接，方法出口等信息。每一个方法被调用的过程就对应一个栈帧在虚拟机栈中从入栈到出栈的过程。栈遵循先进后出的原则。

	栈帧：是用来存储数据和部分过程结果的数据结构。
	栈帧的位置：内存->运行时数据区->某个线程对应的虚拟机栈->here
	栈帧大小确定时间：编译期确定，不受运行期数据影响。

平时说的栈一般指局部变量表部分。

局部变量表：一片连续的内存空间，用来存放方法参数，以及方法内定义的局部变量，存放着编译期间已知的数据类型（八大基本类型和对象引用reference）,returnAddress类型。它的最小的局部变量表空间单位为Slot,虚拟机没有指明Slot的大小，但在jv'm中，long和double类型数据明确规定为64位，这两个类型占2个Slot,其他基本类型固定占用一个Slot.

reference类型：与基本类型不同的是它不等同本身，即使是String，内部也是char数组组成，它可能是指向一个对象起始位置指针，也可能指向一个代表对象的句柄或其他与该对象有关的位置。

returnAddress:指一条字节码指令的地址。

![](jvm.png)

![](jvm3.png)



需要注意的是，局部变量表所需要的内存空间在编译期完成分配，当进入一个方法时，这个方法在栈中需要分配多大的局部变量空间是完全确定的，在方法运行期间不会改变局部变量表大小。

java虚拟机栈可能出现的两种类型的异常：

	1、线程请求的栈深度大于虚拟机允许的栈深度，将抛出StackOverflowError

	2、虚拟机栈空间可以动态扩展，当动态扩展是无法申请到足够的空间时，抛出OutOfMemory

### 本地方法栈

本地方法栈是与虚拟机栈发挥的作用十分相似，区别是虚拟机栈执行的是java方法（也就是字节码）服务，而本地方法栈则为虚拟机使用到的native方法服务，可能底层调用的c或c++,我们打开jdk安装目录可以看到也有很多c编写的文件，可能就是native方法所调用的c代码

### 堆

对于大多数应用来说，堆是java虚拟机管理内存最大的一块内存区域，因为堆存放的对象是线程共享的，所以多线程的时候也需要同步机制。

注意:它是所有线程共享的，它的目的是存放对象实例。同时它也是GC所管理的主要区域，因此常被称为GC堆，又由于现在收集器常使用分代算法，Java堆中还可以细分为新生代和老年代，再细致点还有Eden(伊甸园)空间之类的不做深究。

根据虚拟机规范，Java堆可以存在物理上不连续的内存空间，就像磁盘空间只要逻辑是连续的即可。它的内存大小可以设为固定大小，也可以扩展。

当前主流的虚拟机如HotPot都能按扩展实现(通过设置 -Xmx和-Xms)，如果堆中没有内存完成实例分配，而且堆无法扩展将报OOM错误(OutOfMemoryError)

web应用程序new出的对象放在Eden区（8/10），当放满了后会自动开启一个线程执行minor gc

![](jvm10.png)

### 方法区

方法区同堆一样，是所有线程共享的内存区域，为了区分堆，又被称为非堆。

用于存储已被虚拟机加载的类信息、常量、静态变量，如static修饰的变量加载类的时候就被加载到方法区中。

运行时常量池

是方法区的一部分，class文件除了有类的字段、接口、方法等描述信息之外，还有常量池用于存放编译期间生成的各种字面量和符号引用。

在老版jdk，方法区也被称为永久代【因为没有强制要求方法区必须实现垃圾回收，HotSpot虚拟机以永久代来实现方法区，从而JVM的垃圾收集器可以像管理堆区一样管理这部分区域，从而不需要专门为这部分设计垃圾回收机制。不过自从JDK7之后，Hotspot虚拟机便将运行时常量池从永久代移除了。】

jdk8真正开始废弃永久代，而使用元空间(Metaspace)

![](jvm4.png)

# GC

## GC简介
GC(Garbage Collection)：即垃圾回收器，诞生于1960年MIT的Lisp语言，主要是用来回收，释放垃圾占用的空间。

java GC泛指java的垃圾回收机制，该机制是java与C/C++的主要区别之一，我们在日常写java代码的时候，一般都不需要编写内存回收或者垃圾清理的代码，也不需要像C/C++那样做类似delete/free的操作。

## 为什么需要学习GC

对象的内存分配在java虚拟机的自动内存分配机制下，一般不容易出现内存泄漏问题。但是写代码难免会遇到一些特殊情况，比如OOM神马的。。尽管虚拟机内存的动态分配与内存回收技术很成熟，可万一出现了这样那样的内存溢出问题，那么将难以定位错误的原因所在。

### 哪些内存要回收

java内存模型中分为五大区域已经有所了解。我们知道程序计数器、虚拟机栈、本地方法栈，由线程而生，随线程而灭，其中栈中的栈帧随着方法的进入顺序的执行的入栈和出栈的操作，一个栈帧需要分配多少内存取决于具体的虚拟机实现并且在编译期间即确定下来【忽略JIT编译器做的优化，基本当成编译期间可知】，当方法或线程执行完毕后，内存就随着回收，因此无需关心。

而Java堆、方法区则不一样。方法区存放着类加载信息，但是一个接口中多个实现类需要的内存可能不太一样，一个方法中多个分支需要的内存也可能不一样【只有在运行期间才可知道这个方法创建了哪些对象需要多少内存】，这部分内存的分配和回收都是动态的，gc关注的也正是这部分的内存。

Java堆是GC回收的“重点区域”。堆中基本存放着所有对象实例，gc进行回收前，第一件事就是确认哪些对象存活，哪些死去[即不可能再被引用]

### 堆的回收区域

	 为了高效的回收，jvm将堆分为三个区域
	 1.新生代（Young Generation）NewSize和MaxNewSize分别可以控制年轻代的初始大小和最大的大小
	 2.老年代（Old Generation）
	 3.永久代（Permanent Generation）【1.8以后采用元空间，就不在堆中了】

## 判断对象是否存活算法

1.引用计数算法
早期判断对象是否存活大多都是以这种算法，这种算法判断很简单，简单来说就是给对象添加一个引用计数器，每当对象被引用一次就加1，引用失效时就减1。当为0的时候就判断对象不会再被引用。
优点:实现简单效率高，被广泛使用与如python何游戏脚本语言上。
缺点:难以解决循环引用的问题，就是假如两个对象互相引用已经不会再被其它其它引用，导致一直不会为0就无法进行回收。

2.可达性分析算法
目前主流的商用语言[如java、c#]采用的是可达性分析算法判断对象是否存活。这个算法有效解决了循环利用的弊端。
它的基本思路是通过一个称为“GC Roots”的对象为起始点，搜索所经过的路径称为引用链，当一个对象到GC Roots没有任何引用跟它连接则证明对象是不可用的。

![](gc0.png)

可作为GC Roots的对象有四种

①虚拟机栈(栈桢中的本地变量表)中的引用的对象，就是平时所指的java对象，存放在堆中

②方法区中的类静态属性引用的对象，一般指被static修饰引用的对象，加载类的时候就加载到内存中

③方法区中的常量引用的对象

④本地方法栈中JNI（native方法)引用的对象

要真正宣告对象死亡需经过两个过程。

1.可达性分析后没有发现引用链

2.查看对象是否有finalize方法，如果有重写且在方法内完成自救[比如再建立引用]，还是可以抢救一下，注意这边一个类的finalize只执行一次，这就会出现一样的代码第一次自救成功第二次失败的情况。[如果类重写finalize且还没调用过，会将这个对象放到一个叫做F-Queue的序列里，这边finalize不承诺一定会执行，这么做是因为如果里面死循环的话可能会使F-Queue队列处于等待，严重会导致内存崩溃，这是我们不希望看到的。]

## 垃圾收集算法

1.标记/清除算法【最基础】

2.复制算法

3.标记/整理算法

jvm采用`分代收集算法`对不同区域采用不同的回收算法。

### 新生代采用复制算法

新生代中因为对象都是"朝生夕死的"，【深入理解JVM虚拟机上说98%的对象,不知道是不是这么多，总之就是存活率很低】，适用于复制算法【复制算法比较适合用于存活率低的内存区域】。它优化了标记/清除算法的效率和内存碎片问题，且JVM不以5:5分配内存【由于存活率低，不需要复制保留那么大的区域造成空间上的浪费，因此不需要按1:1【原有区域:保留空间】划分内存区域，而是将内存分为一块Eden空间和From Survivor、To Survivor【保留空间】，三者默认比例为8:1:1，优先使用Eden区，若Eden区满，则将对象复制到第二块内存区上。但是不能保证每次回收都只有不多于10%的对象存货，所以Survivor区不够的话，则会依赖老年代年进行分配】。

GC开始时，对象只会存于Eden和From Survivor区域，To Survivor【保留空间】为空。

GC进行时，Eden区所有存活的对象都被复制到To Survivor区，而From Survivor区中，仍存活的对象会根据它们的年龄值决定去向，年龄值达到年龄阈值(默认15是因为对象头中年龄占4bit，新生代每熬过一次垃圾回收，年龄+1)，则移到老年代，没有达到则复制到To Survivor。

![](jvm11.png)

### 老年代采用标记/清除算法或标记/整理算法

由于老年代存活率高，没有额外空间给他做担保，必须使用这两种算法。

## 枚举根节点算法

GC Roots 被虚拟机用来判断对象是否存活

可达性分析算法需考虑

1.如果方法区几百兆，一个个检查里面的引用，将耗费大量资源。

2.在分析时，需保证这个对象引用关系不再变化，否则结果将不准确。【因此GC进行时需停掉其它所有java执行线程(Sun把这种行为称为‘Stop the World’)，即使是号称几乎不会停顿的CMS收集器，枚举根节点时也需停掉线程】

解决办法:实际上当系统停下来后JVM不需要一个个检查引用，而是通过OopMap数据结构【HotSpot的叫法】来标记对象引用。

虚拟机先得知哪些地方存放对象的引用，在类加载完时。HotSpot把对象内什么偏移量什么类型的数据算出来，在jit编译过程中，也会在特定位置记录下栈和寄存器哪些位置是引用，这样GC在扫描时就可以知道这些信息。【目前主流JVM使用准确式GC】

OopMap可以帮助HotSpot快速且准确完成GC Roots枚举以及确定相关信息。但是也存在一个问题，可能导致引用关系变化。

这个时候有个safepoint(安全点)的概念。

HotSpot中GC不是在任意位置都可以进入，而只能在safepoint处进入。 GC时对一个Java线程来说，它要么处在safepoint,要么不在safepoint。

safepoint不能太少，否则GC等待的时间会很久

safepoint不能太多，否则将增加运行GC的负担

安全点主要存放的位置

1:循环的末尾 

2:方法临返回前/调用方法的call指令后 

3:可能抛异常的位置

## Minor GC、Major GC、FULL GC

Minor GC:在年轻代Young space(包括Eden区和Survivor区)中的垃圾回收称之为 Minor GC,Minor GC只会清理年轻代.

Major GC:Major GC清理老年代(old GC)，但是通常也可以指和Full GC是等价，因为收集老年代的时候往往也会伴随着升级年轻代，收集整个Java堆。所以有人问的时候需问清楚它指的是full GC还是old GC。

Full GC:full gc是对新生代、老年代、永久代【jdk1.8后没有这个概念了】统一的回收。
jvm调优  目的减少full gc次数，full gc的时候会stw,停掉所有线程，影响用户体验的

# Linux查看某个服务JVM的GC和堆内存使用情况

## 使用 jps 命令查看配置了JVM的服务

Jps(Java Virtual Machine Process Status Tool)是JDK 1.5提供的一个显示当前所有java进程pid的命令，简单实用，非常适合在linux/unix平台上简单察看当前java JVM进程的一些简单情况。

	jps
	列出pid和java主类名

![](jvm5.png)

	jps -l
	列出pid和java主类全称
	[root@airthink ~]# jps
	23393 jar
	5401 jenkins.war
	29675 Bootstrap
	31692 Bootstrap


	[root@airthink ~]# jps -l
	23393 bubu-0.0.1-SNAPSHOT.jar
	5401 /usr/lib/jenkins/jenkins.war
	24746 sun.tools.jps.Jps
	29675 org.apache.catalina.startup.Bootstrap
	31692 org.apache.catalina.startup.Bootstrap

	jps -lm
	列出皮带、主类全称和应用程序参数
	[root@airthink ~]# jps -lm
	23393 bubu-0.0.1-SNAPSHOT.jar --server.port=7070
	5401 /usr/lib/jenkins/jenkins.war --logfile=/var/log/jenkins/jenkins.log --webroot=/var/cache/jenkins/war --daemon --httpPort=9090 --debug=5 --handlerCountMax=100 --handlerCountMaxIdle=20
	29675 org.apache.catalina.startup.Bootstrap start
	24908 sun.tools.jps.Jps -lm
	31692 org.apache.catalina.startup.Bootstrap start

	jps -v
	列出pid和JVM参数
	
	[root@airthink ~]# jps -v
	23393 jar
	24964 Jps -Denv.class.path=.:/usr/local/jdk1.8.0_162/lib:/usr/local/jdk1.8.0_162/jre/lib: -Dapplication.home=/usr/local/jdk1.8.0_162 -Xms8m
	5401 jenkins.war -Dcom.sun.akuma.Daemon=daemonized -Djava.awt.headless=true -DJENKINS_HOME=/var/lib/jenkins
	29675 Bootstrap -Djava.util.logging.config.file=/airthink/oauth-apache-tomcat-8.5.46/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Djdk.tls.ephemeralDHKeySize=2048 -Djava.protocol.handler.pkgs=org.apache.catalina.webresources -Dorg.apache.catalina.security.SecurityListener.UMASK=0027 -Dignore.endorsed.dirs= -Dcatalina.base=/airthink/oauth-apache-tomcat-8.5.46 -Dcatalina.home=/airthink/oauth-apache-tomcat-8.5.46 -Djava.io.tmpdir=/airthink/oauth-apache-tomcat-8.5.46/temp
	31692 Bootstrap -Djava.util.logging.config.file=/airthink/apache-tomcat-8.0.50/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Djdk.tls.ephemeralDHKeySize=2048 -Djava.protocol.handler.pkgs=org.apache.catalina.webresources -Dignore.endorsed.dirs= -Dcatalina.base=/airthink/apache-tomcat-8.0.50 -Dcatalina.home=/airthink/apache-tomcat-8.0.50 -Djava.io.tmpdir=/airthink/apache-tomcat-8.0.50/temp

## 查看某个进程JVM的GC使用情况

	 jstat -gc 71614 5000

	 jstat -gc 进程号 刷新时间

![](jvm6.png)

	S0C：年轻代中第一个survivor（幸存区）的容量 (字节) 
	S1C：年轻代中第二个survivor（幸存区）的容量 (字节) 
	S0U：年轻代中第一个survivor（幸存区）目前已使用空间 (字节) 
	S1U：年轻代中第二个survivor（幸存区）目前已使用空间 (字节) 
	EC：年轻代中Eden（伊甸园）的容量 (字节) 
	EU：年轻代中Eden（伊甸园）目前已使用空间 (字节) 
	OC：Old代的容量 (字节) 
	OU：Old代目前已使用空间 (字节)
	YGC：从应用程序启动到采样时年轻代中gc次数 
	YGCT：从应用程序启动到采样时年轻代中gc所用时间(s) 
	FGC：从应用程序启动到采样时old代(全gc)gc次数 
	FGCT：从应用程序启动到采样时old代(全gc)gc所用时间(s) 
	GCT：从应用程序启动到采样时gc用的总时间(s) 

## 查看堆内存使用情况

	jmap -heap 进程号

![](jvm7.png)

![](jvm8.png)

	Heap Configuration:   #堆配置情况 

	MinHeapFreeRatio         = 40 #堆最小使用比例
	MaxHeapFreeRatio         = 70 #堆最大使用比例
	MaxHeapSize              = 482344960 (460.0MB)  #堆最大空间
	NewSize                  = 10485760 (10.0MB) #新生代初始化大小
	MaxNewSize               = 160759808 (153.3125MB) #新生代可使用最大容量大小
	OldSize                  = 20971520 (20.0MB) #老生代大小
	NewRatio                 = 2 #新生代比例
	SurvivorRatio            = 8 #新生代与suvivor的占比
	MetaspaceSize            = 21807104 (20.796875MB) #元数据空间初始大小
	CompressedClassSpaceSize = 1073741824 (1024.0MB)  #类指针压缩空间大小, 默认为1G
	MaxMetaspaceSize         = 17592186044415 MB #元数据空间的最大值, 超过此值就会触发 GC溢出( JVM会动态地改变此值)
	G1HeapRegionSize         = 0 (0.0MB) #区块的大小

	Heap Usage:
	New Generation (Eden + 1 Survivor Space): #新生代大小
	   capacity = 14876672 (14.1875MB) #区块最大可使用大小
	   used     = 2722520 (2.5963973999023438MB) #区块已使用内存
	   free     = 12154152 (11.591102600097656MB) #区块空闲内存
	   18.30059841340859% used #区块使用比例
	Eden Space: # Eden区空间
	   capacity = 13238272 (12.625MB)
	   used     = 2630736 (2.5088653564453125MB)
	   free     = 10607536 (10.116134643554688MB)
	   19.87220084313119% used
	From Space:  #Survivor0区
	   capacity = 1638400 (1.5625MB)
	   used     = 91784 (0.08753204345703125MB)
	   free     = 1546616 (1.4749679565429688MB)
	   5.60205078125% used
	To Space: #Survivor1区
	   capacity = 1638400 (1.5625MB)
	   used     = 0 (0.0MB)
	   free     = 1638400 (1.5625MB)
	   0.0% used
	tenured generation: #老年代
	   capacity = 33013760 (31.484375MB)
	   used     = 26392512 (25.16986083984375MB)
	   free     = 6621248 (6.31451416015625MB)
	   79.94397487593052% used

## linux服务器或tomcat项目启动，进行jvm参数调优设置

首先执行命令：free -h，查询当前的内存占用情况

	[root@airthink ~]# free -h
	              total        used        free      shared  buff/cache   available
	Mem:           1.8G        1.1G        111M        680K        553M        498M
	Swap:            0B          0B          0B

开始进行优化，执行命令：top，查看各个应用的内存占用情况，选取内存占用过高的pid进程；

![](jvm9.png)

然后获取pid号31692，根据pid查询对应的进程以及项目路径，执行命令：
	ps -aux |grep -v grep|grep 31692

	[root@airthink ~]# ps -aux |grep -v grep|grep 31692
	root     31692  0.1 17.2 2575436 324396 ?      Sl   Feb24  16:53 /usr/local/jdk1.8.0_162/jre/bin/java -Djava.util.logging.config.file=/airthink/apache-tomcat-8.0.50/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Djdk.tls.ephemeralDHKeySize=2048 -Djava.protocol.handler.pkgs=org.apache.catalina.webresources -Dignore.endorsed.dirs= -classpath /airthink/apache-tomcat-8.0.50/bin/bootstrap.jar:/airthink/apache-tomcat-8.0.50/bin/tomcat-juli.jar -Dcatalina.base=/airthink/apache-tomcat-8.0.50 -Dcatalina.home=/airthink/apache-tomcat-8.0.50 -Djava.io.tmpdir=/airthink/apache-tomcat-8.0.50/temp org.apache.catalina.startup.Bootstrap start

定位到项目跟路径之后，开始设置项目启动jvm内存占用，不同项目可分配不同的内存

	如果是springboot项目jar启动，则在启动的时候指定jvm的内存分配：
	java -Xms128m -Xmx256m -jar xxx.jar --server.port =8080

	如果是tomcat项目启动，则在bin目录下，执行命令：vim catalina.sh，然后在顶部加上：
	JAVA_OPTS="-Xms128m -Xmx256m"



