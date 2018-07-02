---
title: HashMap源码分析
date: 2018-01-09 12:57:50
tags:
  - Java  
categories:  
  - Java集合框架  
type: tags
---
### 前言
HashMap应该是Java集合框架中用的最多的一个集合框架了，也是每次面试面试官最喜欢问的集合框架之一。面试官喜欢问的问题我汇总如下：
1. 谈谈HashMap的初始化容量是多少？
2. HashMap的扩容机制是怎么样的？
3. HashMap的遍历有哪几种？哪种是速度比较快的？  
带着这些疑问，去解开HashMap的神秘面纱。
jdk1.8中的HashMap实现方式变成了数组+链表+红黑树的实现方式。当链表的长度超过8个node的时候采用红黑树的结构。
首先来看下源码：
#### put方法的实现

```
 final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```
首先来看下源码大致实现：
- 如果当前的table等于null，则初始化table，否则判断table[i]的首个元素是否和key一样，如果相同的话覆盖value。
- 如果table[i]是否为treeNode，即红黑树，则直接在数组插入键值对。
- 第三步table[i],判断链表长度是否大于8，大于8的话把整个tab和链表换为红黑树。
#### hash碰撞方法
分析源码的时候看到容量必须为2的幂次方，为什么不能是其他的数字呢，原来这里来计算该node在数组中的位置的，因为位运算比取余的速度更快。

```
 if ((p = tab[i = (n - 1) & hash]) == null) 
```
也可以这样写：

```
p=tab[i=hash%(n-1)==null]
```
但是这个效率太低了。至于初始容量16为什么，因为能减少碰撞的次数，加快查询的效率。16这个数字也正好是2的幂次方。

#### get方法获取
get方法主要通过是否通过命中节点来判断：源码大概如下：

```
   final Node<K,V> getNode(int hash, Object key) {
        Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & hash]) != null) {
            if (first.hash == hash && // always check first node
                ((k = first.key) == key || (key != null && key.equals(k))))
                return first;
            if ((e = first.next) != null) {
                if (first instanceof TreeNode)
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        return e;
                } while ((e = e.next) != null);
            }
        }
        return null;
    }
```
- 第一次通过桶的第一个元素判断是否相等，如果相等则直接返回。
- 如果没有命中，就去红黑树中获取，否则就去链表中获取。


#### hashMap的扩容机制
扩容就是重新计算容量，向hashMap对象里不停的添加元素，具体来看看它的源码实现：

```
 final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
        if (oldCap > 0) {
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold
        }
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;
        else {               // zero initial threshold signifies using defaults
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
            Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        if (oldTab != null) {
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    if (e.next == null)
                        newTab[e.hash & (newCap - 1)] = e;
                    else if (e instanceof TreeNode)
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { // preserve order
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            next = e.next;
                            if ((e.hash & oldCap) == 0) {
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        return newTab;
    }
```
可以看到jdk1.8的扩容是分成两个链表来进行的。对于最高位为0的数据不需要从新在hash一次，直接存放在newTab[j]中，而最高为为1的数据则放在newTab[j + oldCap] 中。这种写法非常巧妙。
#### 遍历的方法
HashMap可以采用keySet和entrySet来遍历，其中keySet是取到所有的key，之后在通过get取到value，而entrySet则保存了key和value的值，只要取一次就够了。
jdk1.8中可以使用map.Foreach来遍历对应的数据，其对应的实现也是调用迭代器进行遍历，通常在遍历的时候采用entrySet的实现就好了。
### 总结
hashMap集合框架中采用了大量的位运算操作，包括hash操作和扩容时候的再次hash操作。平时开发的时候可以注意这种写法提高效率。




