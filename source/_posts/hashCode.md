---
title: hashCode的理解
date: 2017-12-18 14:48:55
tags:
  - Java
categories:
  - Java
type: tags
---
### hashCode的理解
   hashCode函数在Java中运用的比较多，它的存在主要是为了查找的快捷性。如果两个对象equals相等，那他们的hashCode也相等，如果hashCode相同，只能确认这两个值有冲突，不一定两个对象就相等，在重写equals方法时，应该尽量也重写hashCode方法。  
    比如生成一个对象的时候，如果不重写hashCode方法，如果将整个对象存在一个Set集合中，生成的hashCode不相同，但他们的equals方法是相同的，那么相当于存储了相同的对象了。  
    JDK源码中例如String,HashMap等都有自己的hashCode算法实现。  
    String的hashCode实现和equals方法：
    
   
```
  public int hashCode() {
        int h = hash;
        if (h == 0 && value.length > 0) {
            char val[] = value;

            for (int i = 0; i < value.length; i++) {
                h = 31 * h + val[i];
            }
            hash = h;
        }
        return h;
    }
```

```
    public boolean equals(Object anObject) {
        if (this == anObject) {
            return true;
        }
        if (anObject instanceof String) {
            String anotherString = (String)anObject;
            int n = value.length;
            if (n == anotherString.value.length) {
                char v1[] = value;
                char v2[] = anotherString.value;
                int i = 0;
                while (n-- != 0) {
                    if (v1[i] != v2[i])
                        return false;
                    i++;
                }
                return true;
            }
        }
        return false;
    }
```

Interger的HashCode方法和equals方法的实现：

```
  public boolean equals(Object obj) {
        if (obj instanceof Integer) {
            return value == ((Integer)obj).intValue();
        }
        return false;
    }
```

```
 public int hashCode() {
        return Integer.hashCode(value);
    }
    
```
  再看看hashMap中Entry的hashCode和quals方法
  
```
  public final boolean equals(Object o) {
            if (o == this)
                return true;
            if (o instanceof Map.Entry) {
                Map.Entry<?,?> e = (Map.Entry<?,?>)o;
                if (Objects.equals(key, e.getKey()) &&
                    Objects.equals(value, e.getValue()))
                    return true;
            }
            return false;
        }
```

```
public final int hashCode() {
            return Objects.hashCode(key) ^ Objects.hashCode(value);
        }
```

---
### hashCode的优点
一个良好的哈希算法可以减少冲突次数，增加寻址的效率。如HashMap中运用hashCode进行遍历就可以加快遍历的速度，筛去那些不在一个链表下的数据。
  
---
### 总结
一个对象如果重写equals方法尽量要重写它的hashCode方法。不重写可能会遇到一些未知的情况。









   
   









   
   