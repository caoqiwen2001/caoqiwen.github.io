---
title: Java设计模式之单例
date: 2017-12-14 14:30:07
tags:
  - hexo
categories:
  - 设计模式
type: tags
---

### 单例模式
单例模式是比较基础的一个设计模式了，在并发编程中需要考虑的是线程的安全性去实现对应的方法。


#### 饿汉式
所谓饿汉式代码在命名一个初始化的时候就将对象创建出来了，所以不会有多线程情况下数据的问题。

```
public class EagerSingleton {
    private static EagerSingleton instance = new EagerSingleton();

    private EagerSingleton() {

    }

    public static EagerSingleton getInstance() {
        return instance;
    }
}
```

#### 懒汉式
每次去在初始化的时候判断一下该对象是否为空，为空的时候去创建该对象。

```
public class LazySingleton {
    private static LazySingleton  instance=null;

    private LazySingleton(){

    }

    public  static  LazySingleton getInstance(){
        if (instance==null){
            instance=new LazySingleton();
        }
        return  instance;
    }
}
```
此种情况下，在多线程的情况下会造成数据的竞争而导致该对象在内存中分配两份对象，违背了单例的本意。  
如何解决多线程下存在的并发问题，采用双检锁的方法来实现。代码如下：

```
package com.caoqiwen.learning.design.example2;

/**
 * Created by caoqiwen on 2017/12/13.
 */
public class DoubleCheckLockSingleton {
    private volatile  static  DoubleCheckLockSingleton instance=null;
//    private  static  Object object=new Object();

    private DoubleCheckLockSingleton(){

    }

    /**
     * 线程安全的单例
     * @return
     */
    public  static  DoubleCheckLockSingleton getInstance(){
        if (instance==null){
            synchronized (DoubleCheckLockSingleton.class){
                if (instance==null){
                    instance=new DoubleCheckLockSingleton();
                }
            }
        }
        return instance;
    }

    public  void get(){
        synchronized (this){
            System.out.println("hello world");
        }
    }

}

```

上面这段代码中存在1个疑问的地方。
 为什么DoubleCheckLockSingleton 需要加volatile关键字？
 笔者到网上找了一些答案，大概分析都是这两种情况：  
1. 内存的可见性，让其他线程马上对instance可见。
2. 保证内存顺序的一致性，可以参考知乎上装个讨论[java 单例模式中双重检查锁定 volatile 的作用？](https://www.zhihu.com/question/56606703/answer/238878067)。  
实例化一个对象需要的步骤：  
- 禁止指令的重排序   
（1）分配内存空间  
（2）初始化对象  
（3）将内存空间的地址赋值给对应的引用  
但是由于操作系统可以对指令进行重排序，程序运行的顺序可能是这样子的：  
 （1）分配内存空间  
（2）将内存空间的地址赋值给对应的引用   
（3）初始化对象  
 如果不加volatile 关键字可能会导致该初始化对象没有正确初始化，造成错误。

#### 总结
关于加volatile关键字的作用在面试中也是面试管非常喜欢问的一个问题，好好了解它的作用。即以下两个作用
- 防止指令重排序
- 内存可见性。
