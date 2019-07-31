
<span id="menu"></span>

<!-- TOC -->

- [1. 性能调优](#1-性能调优)
    - [1.1. 性能调优概述](#11-性能调优概述)
    - [1.2. 操作系统性能监控](#12-操作系统性能监控)
    - [1.3. JVM](#13-jvm)
        - [1.3.1. JVM内存模型](#131-jvm内存模型)
        - [1.3.2. JVM的内存空间](#132-jvm的内存空间)
            - [1.3.2.1. 对象内存布局](#1321-对象内存布局)
            - [1.3.2.2. 对象访问定位](#1322-对象访问定位)
        - [1.3.3. 垃圾回收算法](#133-垃圾回收算法)
            - [1.3.3.1. 对象回收判定](#1331-对象回收判定)
            - [1.3.3.2. 对象引用分类](#1332-对象引用分类)
            - [1.3.3.3. 标记清除算法](#1333-标记清除算法)
            - [1.3.3.4. 复制算法](#1334-复制算法)
            - [1.3.3.5. 标记整理算法](#1335-标记整理算法)
            - [1.3.3.6. 分代收集算法](#1336-分代收集算法)
        - [1.3.4. 垃圾收集器](#134-垃圾收集器)
            - [1.3.4.1. Serial收集器](#1341-serial收集器)
            - [1.3.4.2. ParNew收集器](#1342-parnew收集器)
            - [1.3.4.3. Parallel Scavenge收集器](#1343-parallel-scavenge收集器)
            - [1.3.4.4. Serial Old收集器](#1344-serial-old收集器)
            - [1.3.4.5. Parallel Old收集器](#1345-parallel-old收集器)
            - [1.3.4.6. CMS收集器](#1346-cms收集器)
            - [1.3.4.7. G1收集器](#1347-g1收集器)
        - [1.3.5. 内存分配和回收策略](#135-内存分配和回收策略)
        - [1.3.6. 垃圾收集器相关参数介绍](#136-垃圾收集器相关参数介绍)
        - [1.3.7. 性能监控与故障处理工具](#137-性能监控与故障处理工具)
            - [1.3.7.1. JDK命令行工具](#1371-jdk命令行工具)
            - [1.3.7.2. Jdk可视化工具](#1372-jdk可视化工具)
        - [1.3.8. JVM性能调优](#138-jvm性能调优)
        - [1.3.9. 类文件结构](#139-类文件结构)
        - [1.3.10. 类加载器](#1310-类加载器)

<!-- /TOC -->
# 1. 性能调优
<a href="#menu" style="float:right">目录</a>

## 1.1. 性能调优概述

## 1.2. 操作系统性能监控
<a href="#menu" style="float:right">目录</a>

## 1.3. JVM
<a href="#menu" style="float:right">目录</a>

### 1.3.1. JVM内存模型
<a href="#menu" style="float:right">目录</a>
![](https://img2018.cnblogs.com/blog/163758/201811/163758-20181101131229284-1189515543.png)

### 1.3.2. JVM的内存空间
* 堆内存
    * 新生代
        * Eden区
        * From Survivor区
        * To Survivor区
    * 老年代
* 方法区
* 栈内存(线程私有)
    * java虚拟机栈
    * 本地方法栈
* 程序计数器（线程私有）



* **堆内存（Heap）**
    * 对于大多数应用来说，Java 堆（Java Heap）是Java 虚拟机所管理的内存中最大的一块。Java 堆是被所有线程共享的一块内存区域，在虚拟机启动时创建。
    * 此内存区域的唯一目的就是存放对象实例，几乎所有的对象实例都在这里分配内存。
    * 堆内存是所有线程共有的，可以分为两个部分：年轻代和老年代。
    * 下图中的Perm代表的是永久代，但是注意永久代并不属于堆内存中的一部分，同时jdk1.8之后永久代已经被移除。
![](https://img2018.cnblogs.com/blog/163758/201811/163758-20181101131302208-1666214046.png)
    * 新生代 ( Young ) 与老年代 ( Old ) 的比例的值为 1:2 ( 该值可以通过参数 –XX:NewRatio 来指定 )
    * 默认的，Eden : from : to = 8 : 1 : 1 ( 可以通过参数 –XX:SurvivorRatio 来设定 )，即： Eden = 8/10 的新生代空间大小，from = to = 1/10 的新生代空间大小。

* **方法区（Method Area）**
    * 方法区也称"永久代"，它用于存储虚拟机加载的类信息、常量、静态变量、是各个线程共享的内存区域。
    * 在JDK8之前的HotSpot JVM，存放这些”永久的”的区域叫做“永久代(permanent generation)”。永久代是一片连续的堆空间，在JVM启动之前通过在命令行设置参数-XX:MaxPermSize来设定永久代最大可分配的内存空间，默认大小是64M（64位JVM默认是85M）。
    * 随着JDK8的到来，JVM不再有 永久代(PermGen)。但类的元数据信息（metadata）还在，只不过不再是存储在连续的堆空间上，而是移动到叫做“Metaspace”的本地内存（Native memory。
    * 方法区是一种规范，永久代和元空间只是实现方式
    * 由于永久代使用应用内存，很可能导致OOM，因此更换为元空间，可以无限制使用本地内存
* **虚拟机栈(JVM Stack)**
    * 描述的是java方法执行的内存模型：每个方法被执行的时候都会创建一个"栈帧",用于存储局部变量表(包括参数)、操作栈、方法出口等信息。每个方法被调用到执行完的过程，就对应着一个栈帧在虚拟机栈中从入栈到出栈的过程。
    * 本地方法栈(Native Stack)
    * 本地方法栈（Native Method Stacks）与虚拟机栈所发挥的作用是非常相似的，其区别不过是虚拟机栈为虚拟机执行Java方法（也就是字节码）服务，而本地方法栈则是为虚拟机使用到的Native方法服务。

* **程序计数器（PC Register）**
    *  程序计数器是用于标识当前线程执行的字节码文件的行号指示器。多线程情况下，每个线程都具有各自独立的程序计数器，所以该区域是非线程共享的内存区域。
    * 当执行java方法时候，计数器中保存的是字节码文件的行号；当执行Native方法时，计数器的值为空。

* **直接内存**
    * 直接内存并不是虚拟机内存的一部分，也不是Java虚拟机规范中定义的内存区域。jdk1.4中新加入的NIO，引入了通道与缓冲区的IO方式，它可以调用Native方法直接分配堆外内存，这个堆外内存就是本机内存，不会影响到堆内存的大小。

* **JVM内存参数设置**             
    * -Xms设置堆的最小空间大小。
    * -Xmx设置堆的最大空间大小。
    * -Xmn:设置年轻代大小
    * -XX:NewSize设置新生代最小空间大小。
    * -XX:MaxNewSize设置新生代最大空间大小。
    * -XX:PermSize设置永久代最小空间大小。
    * -XX:MaxPermSize设置永久代最大空间大小。
    * -Xss设置每个线程的堆栈大小
    * -XX:+UseParallelGC:选择垃圾收集器为并行收集器。此配置仅对年轻代有效。即上述配置下,年轻代使用并发收集,而年老代仍旧使用串行收集。
    * -XX:ParallelGCThreads=20:配置并行收集器的线程数,即:同时多少个线程一起进行垃圾回收。此值最好配置与处理器数目相等。

* 内存泄露
    * 强引用所指向的对象不会被回收，可能导致内存泄漏，虚拟机宁愿抛出OOM也不会去回收他指向的对象
    * 分类
        * 常发性内存泄漏。发生内存泄漏的代码会被多次执行到，每次被执行的时候都会导致一块内存泄漏。 
        * 偶发性内存泄漏。发生内存泄漏的代码只有在某些特定环境或操作过程下才会发生。常发性和偶发性是相对的。对于特定的环境，偶发性的也许就变成了常发性的。所以测试环境和测试方法对检测内存泄漏至关重要。 
        * 一次性内存泄漏。发生内存泄漏的代码只会被执行一次，或者由于算法上的缺陷，导致总会有一块仅且一块内存发生泄漏。比如，在类的构造函数中分配内存，在析构函数中却没有释放该内存，所以内存泄漏只会发生一次。 
        * 隐式内存泄漏。程序在运行过程中不停的分配内存，但是直到结束的时候才释放内存。严格的说这里并没有发生内存泄漏，因为最终程序释放了所有申请的内存。但是对于一个服务器程序，需要运行几天，几周甚至几个月，不及时释放内存也可能导致最终耗尽系统的所有内存。所以，我们称这类内存泄漏为隐式内存泄漏
* 内存溢出
    * 系统已经不能再分配出你所需要的空间
    * 内存泄露将导致内存溢出
    * 内存溢出分析
        * 内存中加载的数据量过于庞大，如一次从数据库取出过多数据； 
        * 集合类中有对对象的引用，使用完后未清空，使得JVM不能回收； 
        * 代码中存在死循环或循环产生过多重复的对象实体； 
        * 使用的第三方软件中的BUG； 
        * 启动参数内存值设定的过小
    * 解决方法
        * 修改JVM启动参数，直接增加内存。(-Xms，-Xmx参数一定不要忘记加。)
        * 检查错误日志，查看“OutOfMemory”错误前是否有其 它异常或错误。        
        * 使用内存查看工具动态查看内存使用情况　
        * 对代码进行走查和分析，找出可能发生内存溢出的位置。
            * 检查对数据库查询中，是否有一次获得全部数据的查询。一般来说，如果一次取十万条记录到内存，就可能引起内存溢出。这个问题比较隐蔽，在上线前，数据库中数据较少，不容易出问题，上线后，数据库中数据多了，一次查询就有可能引起内存溢出。因此对于数据库查询尽量采用分页的方式查询。
            * 检查代码中是否有死循环或递归调用。
            * 检查是否有大循环重复产生新对象实体。
            * 检查对数据库查询中，是否有一次获得全部数据的查询。一般来说，如果一次取十万条记录到内存，就可能引起内存溢出。这个问题比较隐蔽，在上线前，数据库中数据较少，不容易出问题，上线后，数据库中数据多了，一次查询就有可能引起内存溢出。因此对于数据库查询尽量采用分页的方式查询。
            * 检查List、MAP等集合对象是否有使用完后，未清除的问题。List、MAP等集合对象会始终存有对对象的引用，使得这些对象不能被GC回收。 
* 异常
    * OutOfMemoryError
        * 堆内存不足，无法分配新的内存
    * StackOverflowError
        * 递归调用导致方法深度过高
#### 1.3.2.1. 对象内存布局
* HotSpot对象头
    * 用于存储对象自身运行时数据
    * 类型指针，即对象指向类元数据的指针
        * 通过这个指针确定对象是哪个类的实例
        * 如果Java对象是一个Java数组，那么对象头中还必须有一块用于记录数组长度的数据

HotSpot对象头 Mark Word

|存储内容|标志位|状态|
|---|---|---|
|对象哈希码，对象分代年龄|01|未锁定|
|指向锁记录的指针|00|轻量级锁定|
|指向重量级锁的指针|10|膨胀(重量级锁定)|
|空，不需要记录信息|11|GC标志|
|偏向线程ID，偏向时间戳，对象分代年龄 |01|可偏向|

Mark Word有32bit,25bit对象哈希码，4bit存储对象分代年龄，2bit用于存储锁标志位，1bit固定为0。

#### 1.3.2.2. 对象访问定位

* 句柄访问
    * 使用句柄访问方式，java堆将会划分出来一部分内存去来作为句柄池，reference中存储的就是对象的句柄地址。而句柄中则包含对象实例数据的地址和对象类型数据（如对象的类型，实现的接口、方法、父类、field等）的具体地址信息。
    * 使用句柄访最大的好处是reference中存储着稳定的句柄地址，当对象移动之后（垃圾收集时移动对象是非常普遍的行为），只需要改变句柄中的对象实例地址即可，reference不用修改。
* 直接指针访问(hotspot使用)
    * 如果使用指针访问，那么java堆对象的布局中就必须考虑如何放置访问类型的相关信息（如对象的类型，实现的接口、方法、父类、field等），而reference中存储的就是对象的地址。
    * 使用指针访问的好处是访问速度快，它减少了一次指针定位的时间开销，由于java是面向对象的语言，在开发中java对象的访问非常的频繁，因此这类开销积少成多也是非常可观的，反之则提升访问速度。

### 1.3.3. 垃圾回收算法
<a href="#menu" style="float:right">目录</a>

#### 1.3.3.1. 对象回收判定

**引用计数法**
* 给对象添加一个引用计数器，引用一次则计数器+1,引用失效计数器-1，当计数器为0的时候，说明没有地方引用，垃圾收集器可以将它进行回收
* 缺点：无法解决循环引用

**可达性分析算法**
* 以GC ROOTS为起始点，从这些节点开始向下搜索，搜索所走过的路径称为引用链，当一个对象到GC ROOTS没有引用链相连时，说明不可达，也说明没有任何引用。
* GC Roots对象
    * 虚拟机栈中引用的对象
    * 方法区中类静态对象引用的对象
    * 方法区中常量引用的对象
    * 本地方法栈中引用的对象

#### 1.3.3.2. 对象引用分类
**强引用**
* Object obj = new Object();
* 只要强引用存在，就不会被垃圾回收
* 对于Map,List中存放的对象是强引用，因此一般通过虚引用和弱引用来缓存数据

**软引用**
* 通过SoftReference来实现 
* 内存不足时才会回收，回收之后内存不足将抛出OOM异常
* 可以通过get来获取对象实例
* 用于缓存热数据

**弱引用**
* 通过WeakReference来实现
* 只要发生垃圾回收，将会被回收
* 可以通过get来获取对象实例
* 用于缓存冷数据

**虚引用**
* 通过PhantomReference来实现
* 无法通过虚引用来获取对象的实例
* 虚引用和前面的软引用、弱引用不同，它并不影响对象的生命周期
* 如果一个对象与虚引用关联，则跟没有引用与之关联一样，在任何时候都可能被垃圾回收器回收。
* 虚引用必须和引用队列关联使用，当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会把这个虚引用加入到与之 关联的引用队列中

#### 1.3.3.3. 标记清除算法
* 先标记可回收的对象空间，在标记完成之后进行统一的回收
* 缺点
    * 效率问题，标记和清除两个过程的效率都不高
    * 空间问题，清除后将产生内存碎片，不利于二次使用

#### 1.3.3.4. 复制算法
* 内存按容量分为两个区块，每次只使用一个区块用于内存分配
* 垃圾回收时，将存活的对象复制到另一个区块，按顺序存放
* 复制完成后，一次性清理之前的区块
* 新创建对象将在另一个区块中分配
* 优点
    * 不产生碎片内存
* 缺点
    * 空间利用率不高，每次只能有一块区域分配内存。
    * 复制效率不高

#### 1.3.3.5. 标记整理算法
* 标记对象，然后让存活的对象往一边移动，最后一次性清理掉端边界以外的内存。

#### 1.3.3.6. 分代收集算法
* 将内存分为老年代和新生代
* 新创建的对象在新生代进行内存分配，经过多次垃圾回收之后仍然存活的对象将被放到老年代
* 新生代的对象一般生命周期短，大部分都会被回收掉，因此每次垃圾收集只有很少的对象存活，因此使用复制算法效率比较高
* 老年代的对象经过多次回收仍然存活，说明生命周期长，不容易被回收。因此每次垃圾回收只有少量的对象被回收，因此使用标记清除/标记整理算法效率比较高。


### 1.3.4. 垃圾收集器
<a href="#menu" style="float:right">目录</a>

* HotSpot虚拟机的垃圾收集器
    * 年轻代
        * Serial收集器
        * ParNew收集器
        * Parallel Scavenge收集器
        * G1收集器
    * 老年代
        * CMS收集器
        * Serial Old收集器
        * Parallel Old收集器
        * G1收集器

* 并发和并行
    * 并发:多条垃圾收集线程并行工作，此时用户线程处于等待状态
    * 并发:用户线程和垃圾收集线程同时进行，此时用户线程也可以工作，垃圾收集线程在另一个CPU工作
* stop the world
    * 由于执行垃圾回收，用户线程无法执行，将会导致不可预知的错误，比如响应缓慢，任务超时等
    * 垃圾收集器应当尽量避免发生这种情况
    
#### 1.3.4.1. Serial收集器
<a href="#menu" style="float:right">目录</a>
* 进行垃圾收集时，将会暂停其他工作线程，直到回收完成
* 这将导致出现"stop the world"问题，应用代码会发生不可预知的问题
* 桌面应用场景，分配内存不多，可以使用该垃圾收集器
* client 模式中比较好的选择

#### 1.3.4.2. ParNew收集器
<a href="#menu" style="float:right">目录</a>

* Serial收集器的多线程版本
* Server环境下比较好的新生代收集器
* 与CMS(老年代收集器)很好的配合
* 单CPU环境下，由于存在线程切换，因此效率可能会比Serial收集器低
* 参数配置
    * 配置-XX:+UseConcMarkSweepGC将默认新生代使用ParNew收集器
    * 也可以通过 -XX:+UseParNewGC进行配置
    * 通过-XX：ParallelGCThreads限制线程数


#### 1.3.4.3. Parallel Scavenge收集器
<a href="#menu" style="float:right">目录</a>

* 使用复制算法和多线程方式实现
* 目标是达到一个可控制的吞吐量，吞吐量=用户运行代码时间/(用户运行代码时间+垃圾收集时间)
* 参数配置
    * 控制垃圾收集最大停顿时间，-XX:MaxGCPauseMillis
        * 设置过小，将发生频繁的垃圾收集行为，因为每次只能收集很少的一部分，导致吞吐量降低
    * 设置吞吐量大小:-XX:GCTimeRation (0-100)
        


#### 1.3.4.4. Serial Old收集器
<a href="#menu" style="float:right">目录</a>

* 老年代单线程收集算法，使用标记整理
* 将会发生stop the world 问题

#### 1.3.4.5. Parallel Old收集器
<a href="#menu" style="float:right">目录</a>

* Parallel Scavenge收集器的老年代版本
* 使用标记整理算法

#### 1.3.4.6. CMS收集器
<a href="#menu" style="float:right">目录</a>

* 以获取最短停顿时间为目标的收集器，能够给用户带来更好的响应速度
* 标记清除算法
* 垃圾收集过程
    * 初始标记
        * 需要 stop the world
        * 标记GC Roots能之间关联到的对象 
    * 并发标记
        * 需要 stop the world
        * 进行GC Roots Tracing 的过程
    * 重新标记
        * 修正并发标记期间由于用户线程工作而产生标记变动的那一部分对象的标记记录
        * 停顿时间比初始标记时间长，比并发标记时间短很多
    * 并发清除
* 问题
    * 对CPU资源敏感
    * 无法处理浮动垃圾（Floating Garbage）,可能出现Concurrent Mode Failure失败而导致另一次Fullgc.
    * 使用标记清除算法，会产生比较多的垃圾碎片
        * 碎片过多，老年代没有空间分配，将会触发FULL GC。
        * -XX:UseCMSCompactAtFullCollection（默认开启）
            * FullGC时同时进行内存碎片整理，同时将导致停顿时间变长
        * -XX:CMSFullGCsBeforeCompaction
            * 执行多少次FullGC后才会进行内存碎片整理，默认为0  


#### 1.3.4.7. G1收集器
<a href="#menu" style="float:right">目录</a>

* JDK7+ 默认的垃圾收集器
* 场景
    * 垃圾收集线程和应用线程并发执行，和CMS一样
    * 空闲内存压缩时避免冗长的暂停时间
    * 应用需要更多可预测的GC暂停时间
    * 不希望牺牲太多的吞吐性能
    * 不需要很大的Java堆
* 特点
    * 并行和并发
        * 充分利用多核来缩短stop the world时间
    * 分代收集
    * 空间整合
        * 从整体看是标记-整理算法，从局部看是基于复制算法
        * 收集后不产生内存碎片
    * 可预测的停顿
        * 让使用者指定Mms的时间片段内，垃圾收集的时间不超过Mms.

**内存结构**
* 以往的垃圾回收算法，如CMS，使用的堆内存结构如下：
![](https://upload-images.jianshu.io/upload_images/2184951-f6a73e5ef608cfa8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)
* 新生代：eden space + 2个survivor
* 老年代：old space
* 持久代：1.8之前的perm space
* 元空间：1.8之后的metaspace
这些space必须是地址连续的空间。

* 在G1算法中，采用了另外一种完全不同的方式组织堆内存，堆内存被划分为多个大小相等的内存块（Region），每个Region是逻辑连续的一段内存，结构如下：
![](https://upload-images.jianshu.io/upload_images/2184951-715388c6f6799bd9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)
* 每个Region被标记了E、S、O和H，说明每个Region在运行时都充当了一种角色，其中H是以往算法中没有的，它代表Humongous，这表示这些Region存储的是巨型对象（humongous object，H-obj），当新建对象大小超过Region大小一半时，直接在新的一个或多个连续Region中分配，并标记为H。

**Region**
堆内存中一个Region的大小可以通过-XX:G1HeapRegionSize参数指定，大小区间只能是1M、2M、4M、8M、16M和32M，总之是2的幂次方，如果G1HeapRegionSize为默认值，则在堆初始化时计算Region的实践大小，具体实现如下：
![](https://upload-images.jianshu.io/upload_images/2184951-c6194652e3232be2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

默认把堆内存按照2048份均分，最后得到一个合理的大小。

**GC模式**

G1中提供了三种模式垃圾回收模式，young gc、mixed gc 和 full gc，在不同的条件下被触发。

**young gc**
发生在年轻代的GC算法，一般对象（除了巨型对象）都是在eden region中分配内存，当所有eden region被耗尽无法申请内存时，就会触发一次young gc，这种触发机制和之前的young gc差不多，执行完一次young gc，活跃对象会被拷贝到survivor region或者晋升到old region中，空闲的region会被放入空闲列表中，等待下次被使用。



|参数|含义
|---|---|
|-XX:MaxGCPauseMillis|设置G1收集过程目标时间，默认值200ms
|-XX:G1NewSizePercent|新生代最小值，默认值5%
|-XX:G1MaxNewSizePercent|新生代最大值，默认值60%

**mixed gc**
当越来越多的对象晋升到老年代old region时，为了避免堆内存被耗尽，虚拟机会触发一个混合的垃圾收集器，即mixed gc，该算法并不是一个old gc，除了回收整个young region，还会回收一部分的old region，这里需要注意：是一部分老年代，而不是全部老年代，可以选择哪些old region进行收集，从而可以对垃圾回收的耗时时间进行控制。
那么mixed gc什么时候被触发？
先回顾一下cms的触发机制，如果添加了以下参数：
-XX:CMSInitiatingOccupancyFraction=80 
-XX:+UseCMSInitiatingOccupancyOnly

当老年代的使用率达到80%时，就会触发一次cms gc。相对的，mixed gc中也有一个阈值参数 -XX:InitiatingHeapOccupancyPercent，当老年代大小占整个堆大小百分比达到该阈值时，会触发一次mixed gc.
mixed gc的执行过程有点类似cms，主要分为以下几个步骤：

initial mark: 初始标记过程，整个过程STW，标记了从GC Root可达的对象
concurrent marking: 并发标记过程，整个过程gc collector线程与应用线程可以并行执行，标记出GC Root可达对象衍生出去的存活对象，并收集各个Region的存活对象信息
remark: 最终标记过程，整个过程STW，标记出那些在并发标记过程中遗漏的，或者内部引用发生变化的对象
clean up: 垃圾清除过程，如果发现一个Region中没有存活对象，则把该Region加入到空闲列表中

**full gc**
如果对象内存分配速度过快，mixed gc来不及回收，导致老年代被填满，就会触发一次full gc，G1的full gc算法就是单线程执行的serial old gc，会导致异常长时间的暂停时间，需要进行不断的调优，尽可能的避免full gc.

### 1.3.5. 内存分配和回收策略
<a href="#menu" style="float:right">目录</a>

* 大多数情况下，对象优先在Eden区中分配，当Eden中没有足够空间，虚拟机将发生一次minor GC.
* 大对象（需要大量连续内存空间的Java对象，比如长字符串和数组）直接进入老年代
* 长期存活的对象将进入老年代
    * 虚拟机给每一个对象定义了一个Age年龄计数器，每经过一次Minor GC.年龄增加1,超过阈值将被移动到老年代，默认是15岁。
* 动态对象年龄判定
    * 如果Survivor空间中相同年龄的对象大小的总和大于Survivor空间中总和的一半，则年龄大于或者和等于该年龄的对象则直接进入老年代，不受上面年龄阈值的限制
* 空间分配担保
    * 为什么要进行老年代担保
        * Minor GC最差的情况就是垃圾收集完所有的对象都存活，此时将超过 survivor空间，导致这些对象进入老年代，最终可能导致OOM
    * 在Minor GC之前，虚拟机会先检查老年代最大可用的连续空间是否大于新生代所有对象的空间。
        * 如果成立，那么Minor GC就确认是安全的
        * 如果不成立，那么Minor GC就是不安全的
            * 检查HandlerPromotionFailure是否允许担保失败
                * 如果允许，继续检查老年代最大的连续空间是否大于历次晋升到老年代对象的平均大小，如果大于，则进行Monor GC,尽管存在风险。
                * 如果不允许，那么则进行一次Full GC
                



### 1.3.6. 垃圾收集器相关参数介绍
<a href="#menu" style="float:right">目录</a>

### 1.3.7. 性能监控与故障处理工具
<a href="#menu" style="float:right">目录</a>

#### 1.3.7.1. JDK命令行工具
<a href="#menu" style="float:right">目录</a>

**javap**
* 反编译工具,可用来查看java编译器生成的字节码
    * -help 帮助
    * -l 输出行和变量的表
    * -public 只输出public方法和域
    * -protected 只输出public和protected类和成员
    * -package 只输出包，public和protected类和成员，这是默认的
    * -p -private 输出所有类和成员
    * -s 输出内部类型签名
    * -c 输出分解后的代码，例如，类中每一个方法内，包含java字节码的指令
    * -verbose 输出栈大小，方法参数的个数
    * -constants 输出静态final常量
    
**jps**
* 虚拟机进程状况工具

```
usage: jps [-help]
       jps [-q] [-m | -l|-v|-V] [<hostid>]

Definitions:
    <hostid>:      <hostname>[:<port>]
    
```

**jstat**
* 虚拟机统计信息监视工具
```
Usage: jstat -help|-options
       jstat -<option> [-t] [-h<lines>] <vmid> [<interval> [<count>]]

Definitions:
  <option>      An option reported by the -options option
  <vmid>        Virtual Machine Identifier. A vmid takes the following form:
                     <lvmid>[@<hostname>[:<port>]]
                Where <lvmid> is the local vm identifier for the target
                Java virtual machine, typically a process id; <hostname> is
                the name of the host running the target Java virtual machine;
                and <port> is the port number for the rmiregistry on the
                target host. See the jvmstat documentation for a more complete
                description of the Virtual Machine Identifier.
  <lines>       Number of samples between header lines.
  <interval>    Sampling interval. The following forms are allowed:
                    <n>["ms"|"s"]
                Where <n> is an integer and the suffix specifies the units as 
                milliseconds("ms") or seconds("s"). The default units are "ms".
  <count>       Number of samples to take before terminating.
  -J<flag>      Pass <flag> directly to the runtime system.
```
**jinfo**
* Java配置信息工具类
```
Usage:
    jinfo [option] <pid>
        (to connect to running process)
    jinfo [option] <executable <core>
        (to connect to a core file)
    jinfo [option] [server_id@]<remote server IP or hostname>
        (to connect to remote debug server)

where <option> is one of:
    -flag <name>         to print the value of the named VM flag
    -flag [+|-]<name>    to enable or disable the named VM flag
    -flag <name>=<value> to set the named VM flag to the given value
    -flags               to print VM flags
    -sysprops            to print Java system properties
    <no option>          to print both of the above
    -h | -help           to print this help message

```
**jmap**
Java内存映像工具
```
Usage:
    jmap [option] <pid>
        (to connect to running process)
    jmap [option] <executable <core>
        (to connect to a core file)
    jmap [option] [server_id@]<remote server IP or hostname>
        (to connect to remote debug server)

where <option> is one of:
    <none>               to print same info as Solaris pmap
    -heap                to print java heap summary
    -histo[:live]        to print histogram of java object heap; if the "live"
                         suboption is specified, only count live objects
    -clstats             to print class loader statistics
    -finalizerinfo       to print information on objects awaiting finalization
    -dump:<dump-options> to dump java heap in hprof binary format
                         dump-options:
                           live         dump only live objects; if not specified,
                                        all objects in the heap are dumped.
                           format=b     binary format
                           file=<file>  dump heap to <file>
                         Example: jmap -dump:live,format=b,file=heap.bin <pid>
    -F                   force. Use with -dump:<dump-options> <pid> or -histo
                         to force a heap dump or histogram when <pid> does not
                         respond. The "live" suboption is not supported
                         in this mode.
    -h | -help           to print this help message
    -J<flag>             to pass <flag> directly to the runtime system


```
**jhat**
* 虚拟机堆转储快照分析工具
```
Usage:  jhat [-stack <bool>] [-refs <bool>] [-port <port>] [-baseline <file>] [-debug <int>] [-version] [-h|-help] <file>

	-J<flag>          Pass <flag> directly to the runtime system. For
			  example, -J-mx512m to use a maximum heap size of 512MB
	-stack false:     Turn off tracking object allocation call stack.
	-refs false:      Turn off tracking of references to objects
	-port <port>:     Set the port for the HTTP server.  Defaults to 7000
	-exclude <file>:  Specify a file that lists data members that should
			  be excluded from the reachableFrom query.
	-baseline <file>: Specify a baseline object dump.  Objects in
			  both heap dumps with the same ID and same class will
			  be marked as not being "new".
	-debug <int>:     Set debug level.
			    0:  No debug output
			    1:  Debug hprof file parsing
			    2:  Debug hprof file parsing, no server
	-version          Report version number
	-h|-help          Print this help and exit
	<file>            The file to read

For a dump file that contains multiple heap dumps,
you may specify which dump in the file
by appending "#<number>" to the file name, i.e. "foo.hprof#3".

All boolean options default to "true"

```
**jstack**
* Java堆栈跟踪工具
```
Usage:
    jstack [-l] <pid>
        (to connect to running process)
    jstack -F [-m] [-l] <pid>
        (to connect to a hung process)
    jstack [-m] [-l] <executable> <core>
        (to connect to a core file)
    jstack [-m] [-l] [server_id@]<remote server IP or hostname>
        (to connect to a remote debug server)

Options:
    -F  to force a thread dump. Use when jstack <pid> does not respond (process is hung)
    -m  to print both java and native frames (mixed mode)
    -l  long listing. Prints additional information about locks
    -h or -help to print this help message


```
****


#### 1.3.7.2. Jdk可视化工具
<a href="#menu" style="float:right">目录</a>

* JConsole
* JVisualVM
![](https://github.com/lgjlife/Java-Study/blob/master/pic/jvm/monitor.jpg?raw=true)
![](https://github.com/lgjlife/Java-Study/blob/master/pic/jvm/thread.jpg?raw=true)
![](https://github.com/lgjlife/Java-Study/blob/master/pic/jvm/gc.jpg?raw=true)
### 1.3.8. JVM性能调优

### 1.3.9. 类文件结构

### 1.3.10. 类加载器


