---
title: ThreadLocal案例及源码分析
date: 2018-03-03 10:36:00
tags:
  -  Java  
categories:  
  -  Java多线程集合 
type: tags
---

### ThreadLocal案例
ThreadLocal的作用主要是隔离线程中的数据，它针对每个线程独立设置值，不同线程之间的数据是相互隔离，不会相互影响，可以看一段测试代码：

```
public class ThreadLocalDemo {

    public static void main(String[] args) {
        Thread t1 = new Thread(new ThreadLocalTest());
        Thread t2 = new Thread(new ThreadLocalTest());
        Thread t3 = new Thread(new ThreadLocalTest());
        t1.start();
        t2.start();
        t3.start();
    }


    static class Tools {
        public static ThreadLocal<Integer> t1 = new ThreadLocal<>();
    }

    static class ThreadLocalTest implements Runnable {

        private static AtomicInteger ai = new AtomicInteger();

        @Override
        public void run() {
            try {
                for (int i = 0; i < 3; i++) {
                    Tools.t1.set(ai.addAndGet(1));
                    System.out.println(Thread.currentThread().getName() + "get value---->" + Tools.t1.get());
                }
            } catch (Exception ex) {
                ex.printStackTrace();
            }
        }
    }
}
```
其输出结果为：

```
Thread-1get value---->1
Thread-0get value---->2
Thread-1get value---->3
Thread-0get value---->4
Thread-1get value---->5
Thread-0get value---->6
Thread-2get value---->7
Thread-2get value---->8
Thread-2get value---->9
```
可以看到每次线程之间的值不会重复，不会互相干扰，这就是利用*数据隔离*的方法使多个线程之间的数据不会重复。

下面来研究一下源码的实现：

#### set方法

```
  public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }
```

```
  ThreadLocalMap getMap(Thread t) {
        return t.threadLocals;
    }
```
可以看到，它的大概流程是这样子的：

1. 取得当前线程
2. 获取当前线程的 ThreadLocal.ThreadLocalMap
3. 如果当前ThreadLocal.ThreadLocalMap为空，则创建ThreadLocal.ThreadLocalMap，否则就设置一个值。
源码里面有一个非常重要的函数set(ThreadLocal key, Object value)方法。来看看他的实现：

```
 private void set(ThreadLocal key, Object value) {
            Entry[] tab = table;
            int len = tab.length;
            int i = key.threadLocalHashCode & (len-1);

            for (Entry e = tab[i];
                 e != null;
                 e = tab[i = nextIndex(i, len)]) {
                ThreadLocal k = e.get();

                if (k == key) {
                    e.value = value;
                    return;
                }

                if (k == null) {
                    replaceStaleEntry(key, value, i);
                    return;
                }
            }

            tab[i] = new Entry(key, value);
            int sz = ++size;
            if (!cleanSomeSlots(i, sz) && sz >= threshold)
                rehash();
        }
```
1. 先获取ThreadLocal里面的具体位置
2. 判断当前位置上的ThreadLocal和本身的ThreadLocal是不是同一个，如果相同数据直接覆盖返回即可。
3. 有可能当前的ThreadLocal变成NULL了，因为Entry是ThreadLocal弱引用，有可能被垃圾回收了，此时，只需要将它的值覆盖即可。

#### get方法
get方法比较简单。

```
    public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null)
                return (T)e.value;
        }
        return setInitialValue();
    }
```
1. 获取当前运行的线程
2. 获取当前线程的ThreadLocalMap
3. 如果当前线程存在ThreadLocalMap，则通过map.getEntry方法获取值。
4. 不存在ThreadLocalMap则通过setInitialValue方法设置初始值。

#### remove方法

```
   public void remove() {
         ThreadLocalMap m = getMap(Thread.currentThread());
         if (m != null)
             m.remove(this);
     }
```
remove方法也比较简单，首先获取当前线程，如果线程不为NULL,则直接移除ThreadLocal。

### 总结
- ThreadLocal不是链表法实现的，而是通过开地址法实现的。
- 塞数据的时候如果数组某个位置没有值，则寻找它的下一个位置为空的位置赋值。
- get方法获取值也是同理，没有值往下一个位置找。
- ThreadLocal一般用在框架，servlet中，它是一种空间换时间的思想。


