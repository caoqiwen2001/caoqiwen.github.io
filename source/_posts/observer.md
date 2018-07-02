---
title: Java设计模式-观察者模式
date: 2017-12-21 19:04:24
tags:
  - Java
categories:
  - 设计模式
---
### 观察者模式定义
观察者模式：可以简单的理解为订阅发布，让多个观察者对象同时监听某一个主题对象，当主题对象的状态发生改变的时候，会通知其他监听该主题的观察者对象，能够使他们自动更新他们。
观察者模式包含以下几个对象：

- 抽象主题角色  
 抽象主题角色有一个列表来维护观察角色，与观察角色有依赖关系。
- 具体主题角色  
  当具体主题角色改变一个角色时，会通知所有的观察者角色列表。
- 抽象观察角色  
  抽象观察者角色主要包含一个接口，用来提供给上层实现类。
- 具体观察角色  
  抽象类的接口实现，实现具体的接口，以便主题的状态与本身协调。
### 观察者实例的实现
抽象主题角色，可以增加观察者，删除观察者，通知观察者功能。它是一个抽象类，有对Observer的依赖。

```

public abstract class Subject {
    private  static List<Observer> list = null;
    static {
        list = new ArrayList<>();
    }

    public void attch(Observer observer) {
        list.add(observer);
        System.out.println("Attached an observer");
    }

    public void detach(Observer observer) {
        list.remove(observer);
        System.out.println("Detached an observer");
    }

    public void notifyAllObserver(String newState) {
        for (int i = 0; i < list.size(); i++) {
            list.get(i).update(newState);
        }
    }
}

```
具体的主题角色实现主要为改变状态时通知观察者，可以看到他会主动调起方法去通知所有的观察者

```
public class ConcreteSubject extends  Subject{
    private String state;
    public String getState() {
        return state;
    }
    public void setState(String state) {
        this.state = state;
    }

    public void change(String newState){
        state=newState;
        System.out.println("主体状态为:"+state);
        this.notifyAllObserver(state);
    }
}
```

需要定义一个观察者接口实现：

```
public interface Observer {
    void update(String state);
}
```
具体实现观察者接口的实现：

```
public class ConcreteSubject extends  Subject{
    private String state;
    public String getState() {
        return state;
    }
    public void setState(String state) {
        this.state = state;
    }

    public void change(String newState){
        state=newState;
        System.out.println("主体状态为:"+state);
        this.notifyAllObserver(state);
    }
}
```
### 总结
观察者模式在系统中运用的比较多的地方主要是用来解耦，使得模块的代码更加清晰，更易读，便于扩展，使代码更加容易维护，修改代码尽量修改少数部分。至于在什么场景中使用，还需要结合具体的场景进行分析。
















