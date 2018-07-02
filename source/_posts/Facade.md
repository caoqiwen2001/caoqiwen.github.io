---
title: Java设计模式-外观模式
date: 2017-12-22 15:12:48
tags:
  - Java
categories:
  - 设计模式
---

### 外观模式
外观模式可以表示一个类依赖多个不同的模块，使上层代码不必关注具体的实现细节，*个人认为它的主要作用在于封装代码，用于解耦。*

#### 具体的实例
从整体世界观来说，外观模式我们可以认为对世界的一种抽象认识，比如汽车，我们认为存在这样一个具体物体，但是它由很多个中物体组成，包括车轮、方向盘，座椅、引擎等部件构成，而其中的每个部件又是由很多种部件抽象组成。

#### 具体代码实现
首先实现三个具体模块代码，称之为ModuleA,ModuleB,ModuleC,每一个Module中都有自己的业务方法实现。

```
public class ModuleA {
     public void testModuleA(){
         System.out.println("system out testModuleA");
     }
}
```

```
public class ModuleB {
      public void testModuleB(){
          System.out.println("system out testModuleB");
      }
}

```

```
public class ModuleC {
    public void  testModuleC(){
        System.out.println("system out testModuleC");
    }
}
```
然后轮到外观模式出厂了：

```
public class Facade {
    private ModuleA moduleA = null;
    private ModuleB moduleB = null;
    private ModuleC moduleC = null;

    public Facade() {
        moduleA = new ModuleA();
        moduleB = new ModuleB();
        moduleC = new ModuleC();
    }
    public void test(){
        moduleA.testModuleA();
        moduleB.testModuleB();
        moduleC.testModuleC();
    }


    public static void main(String[] args){
        Facade facade=new Facade();
        facade.test();
    }
}
```

在Facade类中我实现了一个具体的类的实现，其中依赖了ModuleA，ModuleB，ModuleC，并调用了三个模块的方法，其中调用的逻辑可以自己自由发挥，根据你的业务而定，或者你也可以想象这样一个场景，我们在开车的时候，需要踩离合、加油门，这里就开车就是我们需要调用的方法，具体的开车步骤却被封装起来，外界看不到是怎么开车的，只看到车在动。

### 总结
- 外观模式可以根据自己的需求增加依赖进行封装，让子模块不依赖于父模块。
- 父模块不需要知道子模块是怎么实现的，只需要调子模块的方法即可。
- 可以隐藏实现的细节，把需要暴露的接口都集中于Facade类，方便客户端调用。但细节客户端并不知道。
