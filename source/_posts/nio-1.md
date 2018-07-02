---
title: NIO（一）缓冲区  
date: 2018-01-30 17:23:56  
tags:  
  - Java  
categories:  
  - NIO    
type: tags  
---
### I/O的基本概念
1. 同步和异步的概念：
   所谓的同步就是在发出一个请求的时候，如果没有得到结果，就不返回。即调用者主动等待返回结果。做完一件事情在做另一件事情。  
   所谓的异步：调用之后直接返回结果，一般通过回调函数来处理这个应用。可以同时做多件事情。
2. 阻塞和非阻塞的概念：
   阻塞：在调用没有得到结果之前，当前线程会被挂起。调用线程得到结果之后才会返回。  
   非阻塞：不能得到结果之前，该调用不会阻塞住当前的线程。

### 缓冲区
缓冲区其实就是一块数组，然后对这一块数组进行一系列的操作，它包括以下几个常用的方法： 

1分配空间给缓冲区  
2.往缓冲区中添加数据调用put方法  
3.调用flip方法切换为读状态           
4.读取完毕调用clear方法重新设置位置为0。

### 常用方法解析
1.clear方法不清除数据，只是改变当前limit值

```
  public final Buffer clear() {
        position = 0;
        limit = capacity;
        mark = -1;
        return this;
    }
```
2.flip方法允许输出

```
 public final Buffer flip() {
        limit = position;
        position = 0;
        mark = -1;
        return this;
    }
```
3.rewind方法使posotion方法置为0
```
 public final Buffer rewind() {
        position = 0;
        mark = -1;
        return this;
    }
```

4.判断两个缓冲区是否相等的方法，代码如下：

```
    public boolean equals(Object ob) {
        if (this == ob)
            return true;
        if (!(ob instanceof CharBuffer))
            return false;
        CharBuffer that = (CharBuffer)ob;
        if (this.remaining() != that.remaining())
            return false;
        int p = this.position();
        for (int i = this.limit() - 1, j = that.limit() - 1; i >= p; i--, j--)
            if (!equals(this.get(i), that.get(j)))
                return false;
        return true;
    }
```
要满足几下条件才能算相等。    
1、对象的类型要相同  
2、剩余的空间要相同  
3、从position到limit区间的数据要相等才能确认数据是否相等。


#### 字节流缓冲区
字节流缓冲区是开发中经常用到的一个缓冲区，下面来看下它的源代码：

```
  public static ByteBuffer allocate(int capacity) {
        if (capacity < 0)
            throw new IllegalArgumentException();
        return new HeapByteBuffer(capacity, capacity);
    }
```
可以看到它是通过HeapByteBuffer的构造来实现的，而HeapByteBuffer是继承自ByteBuffer，它是分配在堆上面的，由JVM负责垃圾回收。
还有一个allocateDirect的静态方法，它是在虚拟机的内存外分配一块缓冲区，可以通过指令设置对该区域进行设置，但该方法一般用在周期长,文件大的传输过程中，一般情况下不建议使用。

```
    public static ByteBuffer allocateDirect(int capacity) {
        return new DirectByteBuffer(capacity);
    }
```
DirectByteBuffer 方法默认是一个构造方法，可以看到调用unsafe.allocateMemory方法返回内存的一个基地址，以后的操作都是基于这个地址进行的。因为它每次回收依赖于Full GC,回收效率不高，可能会导致内存溢出。
```
   DirectByteBuffer(int cap) {                   // package-private

        super(-1, 0, cap, cap);
        boolean pa = VM.isDirectMemoryPageAligned();
        int ps = Bits.pageSize();
        long size = Math.max(1L, (long)cap + (pa ? ps : 0));
        Bits.reserveMemory(size, cap);

        long base = 0;
        try {
            base = unsafe.allocateMemory(size);
        } catch (OutOfMemoryError x) {
            Bits.unreserveMemory(size, cap);
            throw x;
        }
        unsafe.setMemory(base, size, (byte) 0);
        if (pa && (base % ps != 0)) {
            // Round up to page boundary
            address = base + ps - (base & (ps - 1));
        } else {
            address = base;
        }
        cleaner = Cleaner.create(this, new Deallocator(base, size, cap));
        att = null;

    }
```






