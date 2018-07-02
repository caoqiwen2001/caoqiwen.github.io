---
title: Java多线程（二） synchronized锁机制
date: 2018-02-08 15:43:50
tags:
  -  Java  
categories:  
  -  Java多线程集合 
type: tags
---
### synchronized锁
#### 数据脏读原因
在多个线程对同一个对象实例变量进行并发访问的时候，很容易引起数据脏读的问题，来看一个简单的并发脏读的实例：

```
public class HasSelfPrivateNum {
    private int num = 0;

    public void addI(String username) {
        try {
            if (username.equals("a")) {
                num = 100;
                System.out.println("a set over");
                Thread.sleep(2000);

            } else {
                num = 200;
                System.out.println("b set over");
            }
            System.out.println(username + " num= " + num);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```
接着实现两个线程对HasSelfPrivateNum的方法进行访问：

```
public class ThreadA extends Thread {
    private HasSelfPrivateNum numRef;
    public ThreadA(HasSelfPrivateNum numRef) {
        super();
        this.numRef = numRef;
    }

    @Override
    public void run() {
        super.run();
        numRef.addI("a");
    }
}
```

```
public class ThreadB extends Thread {
    private HasSelfPrivateNum numRef;
    public ThreadB(HasSelfPrivateNum numRef) {
        super();
        this.numRef = numRef;
    }

    @Override
    public void run() {
        super.run();
        numRef.addI("b");
    }
}
```
然后新建两个线程来进行调用：

```
public class Run2_private01 {
    public static void main(String[] args) {
        HasSelfPrivateNum numRef = new HasSelfPrivateNum();
        ThreadA threadA = new ThreadA(numRef);
        threadA.start();
        ThreadB threadB = new ThreadB(numRef);
        threadB.start();
    }
}
```
其结果输出如下：

```
a set over
b set over
b num= 200
a num= 200
```
结果a和b都输出为200，原因在于num这个字段是一个全局的，在numRef中是共用的，在多个线程对这个字段赋值之后，会引起数据的混乱。那么如何解决并发引起的数据混乱问题呢？加上synchronized关键字就可以了，稍微修改下代码：

```
public class HasSelfPrivateNum {
    private int num = 0;

    synchronized
    public void addI(String username) {
        try {
            if (username.equals("a")) {
                num = 100;
                System.out.println("a set over");
                Thread.sleep(2000);

            } else {
                num = 200;
                System.out.println("b set over");
            }
            System.out.println(username + " num= " + num);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```
其他初始化线程的代码不变，执行后结果如下：

```
a set over
a num= 100
b set over
b num= 200
```

#### 锁重入，支持继承
所谓重入，就是线程得到一个对象锁时，再次请求该对象的锁可以再次进入该对象方法。

```
public class Main {
    protected int i = 10;

    synchronized public void operateIinMain() {
        try {
            i--;
            System.out.println("main print i=" + i);
            Thread.sleep(100);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```
通过继承自Main类，来实现重入锁

```
public class Sub extends Main {
    synchronized public void operateIinSub() {
        try {
            while (i > 0) {
                i--;
                System.out.println("sub print i=" + i);
                Thread.sleep(100);
                this.operateIinMain();
                operateIinSub1();
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public synchronized void operateIinSub1() {
        int i = 0;
        while (true) {
            i++;
            if (i > 5) break;
            System.out.println("operateIinSub1 has begined");
        }
    }
}
```
可以看到operateIinSub方法不仅可以调用operateIinMain方法，也可以调用实例的operateIinSub1方法，这就是锁可重入。
#### 锁异常
在多线程遇到锁异常的时候会自动释放其所有的锁：
```
public class Service {
    synchronized public void testMethod() {
        if (Thread.currentThread().getName().equals("a")) {
            System.out.println("ThreadName=" + Thread.currentThread().getName() + " run beginTime=" + System.currentTimeMillis());

            int i = 1;
            while (i == 1) {
                if (("" + Math.random()).substring(0, 8).equals("0.123456")) {
                    System.out.println("ThreadName=" + Thread.currentThread().getName() + " run exceptionTime=" + System.currentTimeMillis());
                    Integer.parseInt("a");
                }
            }
        } else {
            System.out.println("Thread b run Time=" + System.currentTimeMillis());
        }
    }
}
```
需要定义两个线程类：

```
public class ThreadA extends Thread {
    private Service service;

    public ThreadA(String name, Service service) {
        super(name);
        this.service = service;
    }

    @Override
    public void run() {
        service.testMethod();
    }
}
```

```
public class ThreadB extends Thread {
    private Service service;

    public ThreadB(String name, Service service) {
        super(name);
        this.service = service;
    }

    @Override
    public void run() {
        service.testMethod();
    }
}
```
主线程去调用这两个线程使其中一个抛出异常：

```
public class Run6_exception {
    public static void main(String[] args) {
        try {
            Service service = new Service();
            ThreadA a = new ThreadA("a", service);
            a.start();
            Thread.sleep(500);
            ThreadB b = new ThreadB("b", service);
            b.start();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```
输出结果如下所示：

```
ThreadName=a run beginTime=1517811391726
ThreadName=a run exceptionTime=1517811393510
Thread b run Time=1517811393510
Exception in thread "a" java.lang.NumberFormatException: For input string: "a"
	at java.lang.NumberFormatException.forInputString(NumberFormatException.java:65)
	at java.lang.Integer.parseInt(Integer.java:580)
	at java.lang.Integer.parseInt(Integer.java:615)
	at com.brianway.learning.java.multithread.synchronize.example6.Service.testMethod(Service.java:15)
	at com.brianway.learning.java.multithread.synchronize.example6.ThreadA.run(ThreadA.java:16)
```
可以看到，抛出异常的同时线程B中的代码也开始执行了。
#### 锁方法块
有时直接锁方法会造成执行一个同步时间长的任务，那么另外一个线程必须等待较长的时间。可以通过锁代码块来提高运行效率：
1. 通过synchronized(this)同步代码块。
2. synchronized(非this对象)来同步代码实现。同一个时刻只能有一个线程能执行代码块中的代码。


### 总结
上面是线程中锁的初步运用，在一些高并发框架里面运用的非常多，遇到多个线程数据同步的问题优先考虑加锁。
