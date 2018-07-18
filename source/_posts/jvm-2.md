---
title: JVM（相关知识点整理二）
date: 2018-07-18 21:23:04
tags:
  - Java虚拟机
categories:
    - Java虚拟机
type: tags
---
### JVM相关知识点二  
知道了相关的JVM算法之后，我们来了解一下垃圾回收器以及每个垃圾回收器的特点。
1. Serial收集器  
    该收集器是最基本，发展历史最悠久的一个收集器，它采用复制算法的单线程收集器，单线程默认只会使用一个CPU或者一条线程去收集垃圾，另一方面也意味着它进行垃圾收集时必须暂用其他线程的工作。到现在为止，它依然是虚拟机运行在Client模式下的默认新生代收集器。因为是单线程没有与其他线程的交互工作，专门做垃圾回收自然可以获得最高的单线程收集效率。
1. ParNew收集器  
   它是Serial收集器的多线程版本，除了使用多条线程进行垃圾回收以外，其余行为还包括Serial收集器可用的所有参数。ParNew收集器在单CPU的环境中绝对不会比Serial收集器有更好的效果，甚至还存在线程交互的开销，该收集器在通过超线程技术实现的两个CPU的环境中都不能百分之百地保证可以超越Serial收集器，直接的来说，这款收集器现在用的比较少了。
2. Parallel Scavenge收集器  
    它是一个新生代的收集器，采用复制算法，而且是并行的多线程收集器，它的目标是达到一个可控制的吞吐量，<font color="red">所谓的吞吐量就是CPU用于运行用户代码的时间与CPU总消耗时间的比值。</font>
    假如虚拟机总共运行了100分钟，其中垃圾回收一分钟，那吞吐量就是99%。Parallel Scavenge收集器还有一个参数-XX:+UseAdaptiveSizePolicy,当这个参数打开以后，就不需要手工指定新生代的大小了，虚拟机会根据当前系统的运行情况收集性能监控信息，动态调整这些参数以提供最合适的停顿时间或者最大的吞吐量。
1. Serial Old收集器  
   它是Serial收集器的老年代版本，同样是一个单线程的收集器，使用“标记-整理算法”。这个收集器的主要意义在于给Client模式下的虚拟机使用。如果在Server模式下，有两大用途：一种用途是在JDK1.5 以及之前的版本与Parallel Scavenge收集器搭配使用。另一种用途就是作为CMS收集器的后备预案。
2. Parallel Old收集器   
   它是Parallel Scavenge收集器的老年代版本，使用多线程和“标记-整理”算法，这个收集器在JDK1.6中才开始提供使用。
1. CMS收集器
   它是一种以获取最短回收停顿时间为目标的收集器。目前很大部分Java应用集中在互联网站或者B/S系统的服务端上，这类应用尤其重视服务的响应速度。希望系统的停顿时间越短，给用户带来比较好的体验。它的整个过程分为四个步骤：
- [ ]   初始标志
- [ ] 并发标记
- [ ] 重新标志
- [ ] 并发清除  
整个过程中耗时比较长的并发标记和并发清除过程是和用户线程一起工作，但它占用了一部分CPU资源，<font color="red">整理的吞吐量降低。它无法处理浮动的垃圾，收集结束时会大量空间碎片的产生。</font>    
1. G1收集器  
G1收集器是当今收集器技术发展的最前沿成果之一。它主要有以下几个特点。
- [ ] 并行与并发，G1能充分利用多CPU，多核环境下的硬件优势，使用多个CPU来缩短Stop-The-World停顿时间
- [ ] 分代收集：与其他收集器一样，分代概念在G1中依然得以保留。虽然G1可以不需要其他收集器配合就能独立管理整个GC堆，但它能够采用不同的方式去处理新创建的对象和已经存活了一段时间。
- [ ]  空间整合：与CMS的"标志-清理”算法不同，G1从整体来看基于"标记-清理”算法来实现的。从局部上来看是基于“复制”算法来实现的，但无论如何，这两种算法都意味着G1运作期间不会产生内存空间碎片，收集后能提供规整的可用内存。
- [ ] - 可预测的停顿：这是G1相对于CMS的另一大优势，降低停顿时间是G1和CMS共同的关注点，但G1除了追求停顿外，还能建立可预测的停顿时间模型。  
G1收集器之所以能建立可预测的停顿时间模型，是因为它可以有计划地避免在整个java堆中进行垃圾回收，G1跟踪各个Region里面垃圾堆积的价值大小，在后台维护一个优先列表，每次根据允许收集时间，优先回收价值最大的Region。 
#### 如何查看日记
查看JVM日记就是看每次GC后的堆内存的相关显示，来看一段分布在Eden区上的代码：
#### 测试Eden区堆分布

```
package com.brianway.learning.java.jvm.memory;

public class EdenAllocationTest {

    private static final int _1MB = 1024 * 1024;

    /**
     * 测试新生代对象分配在Eden区上
     * 虚拟机参数  -verbose:gc -XX:+PrintGCDetails   -Xms20M -Xmx20M -Xmn10M -XX:SurvivorRatio=8
     * @param args
     */
    public static void main(String[] args) {
        byte[] allocation1 = new byte[2 * _1MB];
        byte[] allocation2 = new byte[2 * _1MB];
        byte[] allocation3 = new byte[2 * _1MB];
        byte[] allocation4 = new byte[4* _1MB];

    }
}


```
这里用的虚拟机参数为新生代10m,老年代10m，from和to区按照1:1进行划分，各占1M空间。收集器类型为Parallel Scavenge。接下来看看输出的虚拟机日记：

```
[GC (Allocation Failure) [PSYoungGen: 6150K->712K(9216K)] 6150K->4808K(19456K), 0.0136399 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
Heap
 PSYoungGen      total 9216K, used 7095K [0x00000000ff600000, 0x0000000100000000, 0x0000000100000000)
  eden space 8192K, 77% used [0x00000000ff600000,0x00000000ffc3be08,0x00000000ffe00000)
  from space 1024K, 69% used [0x00000000ffe00000,0x00000000ffeb2020,0x00000000fff00000)
  to   space 1024K, 0% used [0x00000000fff00000,0x00000000fff00000,0x0000000100000000)
 ParOldGen       total 10240K, used 4096K [0x00000000fec00000, 0x00000000ff600000, 0x00000000ff600000)
  object space 10240K, 40% used [0x00000000fec00000,0x00000000ff000020,0x00000000ff600000)
 Metaspace       used 3446K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 376K, capacity 388K, committed 512K, reserved 1048576K
```
(Allocation Failure)表示新生代内存不够触发的GC, ** 6150K->712K(9216K** 表示新生代堆内存从6150K降到712K,新生代的总内存为9216K。**6150K->4808K(19456K)** 表示堆内总内存从6159K降到4808K，堆的总内存为19456K。  
从这里可以看出堆之前都是分布在新生代上的，GC之后，有一部分放入到老年代去了，老年代占的空间为4096K。也就是说新生代，后面分布的4M直接就放到新生代中了。
#### 大容量对象直接分布在老年代中

```
/**
 * 虚拟机参数为-XX:+PrintGCDetails -XX:+UseSerialGC -Xms20M -Xmx20M -Xmn10M -XX:SurvivorRatio=8 -XX:PretenureSizeThreshold=3145728
 */
public class OldAllocationTest {
    private static final int _1MB = 1024 * 1024;


    public static void main(String[] args) {
        byte[] allocation1 = new byte[4* _1MB];
    }
}
```
这里使用的收集器是UseSerialGC，因为只有它才能设定分配的最大值，通过这个参数 -XX:PretenureSizeThreshold来设置对应的值。来看下输入的日记：

```
 def new generation   total 9216K, used 1886K [0x00000000fec00000, 0x00000000ff600000, 0x00000000ff600000)
  eden space 8192K,  23% used [0x00000000fec00000, 0x00000000fedd7b20, 0x00000000ff400000)
  from space 1024K,   0% used [0x00000000ff400000, 0x00000000ff400000, 0x00000000ff500000)
  to   space 1024K,   0% used [0x00000000ff500000, 0x00000000ff500000, 0x00000000ff600000)
 tenured generation   total 10240K, used 4096K [0x00000000ff600000, 0x0000000100000000, 0x0000000100000000)
   the space 10240K,  40% used [0x00000000ff600000, 0x00000000ffa00010, 0x00000000ffa00200, 0x0000000100000000)
 Metaspace       used 3106K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 341K, capacity 388K, committed 512K, reserved 1048576K
```
可以看到4096K直接分配到老年区了。
### 总结
通过以上的知识点，大概了解了JVM收集器的垃圾回收器的几种基本类型及如何查看垃圾日记，这为我们后续虚拟机的调优奠定了基础。




