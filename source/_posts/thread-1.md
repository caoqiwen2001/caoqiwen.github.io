---
title: Java多线程（-） Thread的基础概念
date: 2018-02-05 15:43:50
tags:
  -  Java  
categories:  
  -  Java多线程集合 
type: tags
---
### Thread的概念
Java中多线程占了重要的一环，无论是在面试中还是在阅读源码的过程中多有着许多重要的体现。使用多线程可以发挥多核CPU的优势，多个线程将在多个核上运行，可以提高系统的吞吐率。

#### 线程的状态
1. 新建状态  
当我们new一个线程的时候，该线程就处于新建状态。
1. 就绪状态  
当线程新建以后，调用start方法使得线程进入就绪状态,等待cpu的调度。  
1. 阻塞状态    
线程正在等待进入监视器锁，此刻线程就是等待状态。
1. 等待状态  
线程调用object的wait方法（不带参数）、不带超时的Thread的join()方法，LockSupport的park()方法会进入等待状态，此刻，线程释放锁，等待另一个线程调用notify或者notifyAll方法进行唤醒。
1. 超时等待状态  
线程调用object带参数的wait方法、Thread的join()方法、Thread的sleep()方法、LockSupport的parkNanos()方法、LockSupport的parkUntil()方法，有点意思的是，如果在指定时间内，有线程唤醒，则不会等待时间到提前执行后面代码，可见，超时等待也会释放锁。
1. 终止状态  
线程调用终止或者run方法执行结束，线程处于终止状态，不能继续运行。

#### 线程中常用知识点
1. currentThread()
currentThread()是**正在执行线程对象的引用**，即由哪个线程调用的。
1. this.getName()  
getName()这个方法表示当前正在执行的线程。
this.getName()！=Thread.currentThread().getName()，这是两个概念。  
来看段代码：
```
class CountOperate extends Thread {

    static {
        System.out.println("Thread.currentThread().getName()=" + Thread.currentThread().getName());
    }

    public CountOperate() {
        System.out.println("CountOperate---begin");
        System.out.println("Thread.currentThread().getName()=" + Thread.currentThread().getName());
        System.out.println("this.getName()=" + this.getName());
        System.out.println("CountOperate---end");
    }

    @Override
    public void run() {
        System.out.println("run---begin");
        System.out.println("Thread.currentThread().getName()=" + Thread.currentThread().getName());
        System.out.println("this.getName()=" + this.getName());
        System.out.println("run---end");
    }
}

public class Run3_getName {
    public static void main(String[] args) {
        CountOperate c = new CountOperate();
        Thread t1 = new Thread(c);
        t1.setName("A");
        t1.start();
    }
}
```
可以看到输出为：

```
Thread.currentThread().getName()=main
CountOperate---begin
Thread.currentThread().getName()=main
this.getName()=Thread-0
CountOperate---end
run---begin
Thread.currentThread().getName()=A
this.getName()=A
run---end
```
（1）静态块代码是main线程执行的。  
（2）构造函数由main线程执行的，当前线程的名字为Thread-0。  
（3）run方法由A线程执行的，当前线程为A。  
所以Thread.currentThread().getName()不一定就是A这个线程。

#### 守护线程
守护线程：Java中的两种线程，包括守护线程和用户线程，守护线程是一种为用户线程服务的线程，它的消亡是伴随着进程中没有线程了，那么它会自动销毁。来看一个简单例子：

```
public class Run19_daemon {
    public static void main(String[] args) {
        MyThread19 myThread19 = new MyThread19();
        myThread19.setDaemon(true);
        myThread19.start();
        MyThread20 myThread20 = new MyThread20();
        new Thread(myThread20).start();
        try {
            Thread.sleep(5000);
            System.out.println("主线程结束了");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    static class MyThread19 extends Thread {
        private int i = 0;

        @Override
        public void run() {
            // super.run();
            try {
                while (true) {
                    i++;
                    System.out.println("i is the number:" + i);
                    Thread.sleep(1000);
                }
            } catch (Exception ex) {
                ex.printStackTrace();
            }
        }
    }

    static class MyThread20 implements Runnable {

        @Override
        public void run() {
            try {
                Thread.sleep(9000);
                System.out.println("子线程结束了，守护线程也消失了");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```
运行结果如下：

```
i is the number:1
i is the number:2
i is the number:3
i is the number:4
i is the number:5
主线程结束了
i is the number:6
i is the number:7
i is the number:8
i is the number:9
子线程结束了，守护线程也消失了
```
可以发现，只有等子线程结束了，守护线程也结束了。即进程中没有多余的线程。

#### interrupt方法
interrupt方法无法中断正在执行的线程，除非该线程正在被阻塞，来看段代码：

```
class MyThread7 extends Thread {
    @Override
    public void run() {
        super.run();
        for (int i = 0; i < 500000; i++) {
            System.out.println("i=" + (i + 1));
        }
    }
}
```

```
public class Run7_interrupt01 {
    public static void main(String[] args) {
        try {
            MyThread7 myThread7 = new MyThread7();
            myThread7.start();
            Thread.sleep(1000);
            myThread7.interrupt();
        } catch (InterruptedException e) {
            System.out.println("main catch");
            e.printStackTrace();
        }

    }
}
```
代码本意是想中断正在执行的MyThread7中的for循环代码，结果却一直在打印。

```
i=383497
i=383498
i=383499
i=383500
i=383501
i=383502
i=383503
i=383504
i=383505
i=383506
.......后续就没写出来了
```
说明**没有被阻塞的线程，调用interrupt方法并不会中断当前线程**。

### 总结
线程的基本概念包括以上这些内容，掌握这些对于以后研究高并发框架和源码有着很重要的作用。

