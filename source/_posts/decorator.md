---
title: Java设计模式-装饰器模式  
date: 2017-12-27 09:22:26   
tags:
  - Java
categories:
  - 设计模式
type: tags
---
### 装饰器模式
装饰器主要给模块添加额外的功能，可以通过派生或者组合来生成。如果通过直接修改对应类的来实现并不可取，造成很多冗余代码，在平常写代码的时候，尽可能用组合来实现。

装饰器实现的主要主要角色：
1. 抽象构件角色    
   一个抽象接口，以规范准备接受附加责任角色。
2. 具体构件角色  
   定义一个将要接受附加责任的类。
1. 装饰角色  
   持有构件角色实例，并定义与一个抽象构件接口的实例。
1. 具体装饰角色
   给构件角色贴上对应的责任。  
总结一句话：保持接口，增强性能。

#### 装饰器的具体实现
最近荒野行动这款吃鸡游戏很流行，国内盛行的抄袭风又开始了，如小米枪战，腾讯吃鸡手游，都是圈钱的机器，笔者就以这个举例来实现，首先我们需要一个具体吃鸡的接口：

```
/**
 * Created by caoqiwen on 2017/12/26.
 * 模拟吃鸡
 */
public interface Chicken {
     void eatChicken();
}
```
那么在进入游戏时，给个吃鸡的提示，可以给一个具体的提示如“大吉大利，今晚吃鸡”，给具体的构件角色：

```
public class ConChicken implements  Chicken{
    @Override
    public void eatChicken() {
            System.out.println("大吉大利，今晚吃鸡");
    }
}
```
大家都知道，在进入游戏点击的时候，肯定是少不了广告的，如腾讯公司可能会中插一段视频推销自己的最新游戏产品，而小米公司可能会推销自己最新出了几款新手机，笔者就以两家的广告营销为例作为增强吃鸡的功能。

```
public class MiChicken  implements  Chicken{

    private Chicken chicken;
    public  MiChicken(Chicken chicken){
        super();
        this.chicken=chicken;
    }
    @Override
    public void eatChicken() {
        System.out.println("小米吃鸡游戏放点小米官网的广告");
        chicken.eatChicken();

    }
}
```

```
public class TenCentChicken implements Chicken{
    private Chicken chicken;
    public TenCentChicken(Chicken chicken){
        super();
        this.chicken=chicken;
    }
    @Override
    public void eatChicken() {
        System.out.println("腾讯吃鸡游戏放点马化腾的广告");
        chicken.eatChicken();

    }
}
```
等玩家点击游戏打开APP时，此时可以这样调用：

```
       Chicken chicken=new MiChicken(new ConChicken());
        chicken.eatChicken();

        Chicken chicken2=new TenCentChicken(new ConChicken());
        chicken2.eatChicken();
```
此时输出为：  

```
小米吃鸡游戏放点小米官网的广告  
大吉大利，今晚吃鸡  
腾讯吃鸡游戏放点马化腾的广告  
大吉大利，今晚吃鸡
```
小米公司和腾讯达成协议，要在腾讯游戏中植入小米手机的营销广告，此时，我们可以这样调用代码进行增强：

```
Chicken chicken1=new TenCentChicken(new MiChicken(new ConChicken()));
        chicken1.eatChicken();
```
此时输入为：

```
腾讯吃鸡游戏放点马化腾的广告
小米吃鸡游戏放点小米官网的广告
大吉大利，今晚吃鸡
```

#### 装饰模式与代理模式的对比
笔者认为这两个设计模式的实现是非常类似的，只不过两个模式的侧重点不同，代理模式是委托实现该类的功能，屏蔽自身类，而装饰模式则是对原类方法的功能进行增强，在I/O流中就用到了很多装饰模式。

#### 总结
- 装饰模式通过继承和引用来扩展功能，它可以提供更多的灵活性。
- 通过不同的具体装饰器以及装饰排列组合，设计师可以创造出不同的组合来满足不同的业务需求。




    

