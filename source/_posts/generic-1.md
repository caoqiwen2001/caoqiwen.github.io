---
title: 浅谈Java中的泛型  
date: 2017-12-26 16:38:29
tags:
  - Java
categories:
  - Java
type: tags
---
### 浅谈Java中的泛型
泛型算是比较高级知识的了，但是作为一名合格的开发人员，泛型是必须要掌握的一门技术，熟悉掌握好它才能为以后的框架开发打好基础。  
在项目中泛型无处不在，只是可能我们关注的比较少而已。

```
  ArrayList<String> strings=new ArrayList<>();
        strings.add("123");
        strings.add("234");
        strings.add("444");
```
如上面这段代码就是我们常用的泛型方法，采用泛型主要是解决了以下几个问题：
1. 类型安全，类型错误在编译期间就确定了，更容易找到错误。
2. 消除了代码强制转换，增强了代码的可读性。
3. 提升了代码的性能。

####  泛型方法的定义
静态方法中泛型的定义：

```
static <T>  void  writeWithWildcard(List<? super T> list, T item) {
        list.add(item);
    }
```
静态方法中有T,如果我们去掉T会怎么样呢？它会报无法识别的错误，所以静态方法是必须加<T>的。  
而在常量方法中则不需要添加T，否则会报错。

####  泛型作为参数的用法

```
class Fruit {
}

class Apple extends Fruit {
}

class Jonathan extends Apple {
    
}

class Orange extends Fruit {
}
```
代码定义了一个基类Fruit和它的几个派生类，下面可以用泛型来测试一下。  

```
 static <T> T readExact(List<? extends  T> list) {
        return list.get(0);
    }
```

? extends  T是指***上界通配符***，它能保证所有类型至少是T类型或者是T类型的派生类型，在get的时候至少可以保证该类型是安全的。 以上面的水果为例，可以这样写：

```
 static List<Apple> apples = Arrays.asList(new Apple());
    static List<Fruit> fruit = Arrays.asList(new Fruit());
        static void f1() {
        Apple a = readExact(apples);
        Fruit f = readExact(fruit);
        f = readExact(apples);
    }
```
基类是什么类型，则取出来的类型不能低于基类，否则会报类型不匹配错误。  
而? super  T是指***下界通配符***，它决定了下界，能放入的类型只能是该类型及该类型的派生类。存入元素的时候是安全的。以上面的水果为例，测试添加水果类。

```
  static <T> void writeExact(List<? super T> list, T item) {
        list.add(item);
    }
```

```
 static List<Apple> apples = new ArrayList<Apple>();
    static List<Fruit> fruit = new ArrayList<Fruit>();
        static void f1() {
        writeExact(apples, new Apple());
        writeExact(fruit, new Apple()); 
   
    }
```
可以往列表中插入一系列的水果，它的下界是Fruit，即可以是Fruit和它派生类等一系列对象。如果T类型是Apple，则插入的对象只能是Apple和派生自Apple类型了。
#### 总结
- 频繁往外读取内容的，适合用上界extends
- 需要往内插入数据的，适合用下界super























