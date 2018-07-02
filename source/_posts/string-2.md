---
title: 再谈Java中的String
date: 2017-12-25 11:10:34
tags: 
  - Java
categories:
    - Java
type: tags
---
#### String类型是final的？
Java中的String是老生常谈的事情了，笔者比较疑惑的问题是String为啥是final的？
来看下这段代码：


```
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];
```
后来笔者经过分析和实践，得出结论：因为它是final所以不可被继承。其次，看到value数组也是final的，这个final的意思内存地址不可改变，但是它char数量里面的内容是可以改变的。  
String在给初始化常量值的时候是在常量池中分配地址空间给它的，比如 String c="hello";  c+="bbb";
此时的c是指向一个新开辟的地址空间的内容。  
再来说说我们关心字符串的追加方式问题。

```
  public static String appendStr(String c){
        c+="bbb";
        return c;
    }

    public static  StringBuilder  appendStrigBuilder(StringBuilder d){
        return  d.append("nihao");
    }
```

```
        String c="hello";
        String d=appendStr(c);
        System.out.println(c);
        StringBuilder builder=new StringBuilder("hello");
        appendStrigBuilder(builder);
        System.out.println(builder);
```

如果调用appendStr的时候发现原String内容的值并没有改变，笔者的分析是，当String作为参数传递给形参时，会传递一个地址的副本过去，此时，他们都指向常量池中的字符串，调用 c+="bbb";的时候，副本地址指向一个新的常量字符串地址"hellobbb",而原来c的字符串地址为final不能修改，即地址不能修改，这就是不可变性。
而对于StringBuilder的append方法，看下它的源码：

```
@Override
    public StringBuilder append(String str) {
        super.append(str);
        return this;
    }
```
看下基类的append方法：

```
  public AbstractStringBuilder append(String str) {
        if (str == null)
            return appendNull();
        int len = str.length();
        ensureCapacityInternal(count + len);
        str.getChars(0, len, value, count);
        count += len;
        return this;
    }
```

可以发现AbstractStringBuilder中的value数组不是final的，那它的地址是可以改变的。
重点可以看下ensureCapacityInternal这个方法，发现它调用了Arrays.copyOf这个方法的实现。

```
 public static char[] copyOf(char[] original, int newLength) {
        char[] copy = new char[newLength];
        System.arraycopy(original, 0, copy, 0,
                         Math.min(original.length, newLength));
        return copy;
    }
```
可以看到，*新开了一个char数*组，调用底层的System.arraycopy方法将原来的地址指向新的数组从而达到append的效果。
#### StringBuffer与StringBuilder
   StringBuffer是线程安全的，在并发的情况下采用StringBuffer，它的原理就是在方法的前面加了synchronized关键字加锁。
   
```
 @Override
    synchronized StringBuffer append(AbstractStringBuilder asb) {
        toStringCache = null;
        super.append(asb);
        return this;
    }
```
再看看String的concat方法，它的contact方法是效率极低的做法，因为char数组地址不可变，只能每次去new一个新的String返回。

```
  public String concat(String str) {
        int otherLen = str.length();
        if (otherLen == 0) {
            return this;
        }
        int len = value.length;
        char buf[] = Arrays.copyOf(value, len + otherLen);
        str.getChars(buf, len);
        return new String(buf, true);
    }
```
可以看到，每次返回的时候都是一个新的对象，如果遇到字符串拼接非常多的场景，每次都是要去申请内存，效率会非常低。所以，在拼接字符串的时候，尽量不用String.contact方法。

#### 总结
- String是不可变的，指的是地址不可变
- 字符串多的时候追加尽量用StringBuilder，多线程下用StringBuffer。

















