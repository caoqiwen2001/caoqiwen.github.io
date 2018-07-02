---
title: LinkedList源码分析
date: 2018-01-02 19:39:08
tags:
  - Java  
categories:  
  - Java集合框架  
type: tags
---
### 前言
LinkedList也是用的非常多的一个集合框架，由于它的底层是采用双向链表实现的。双向链表既有头又有尾节点，它的尾节点的后一个节点是链表的头结点，链表头结点的前一个节点是尾节点。

#### 链表的结构
LinkedList中有一个内部类来标识它的内部存储单元：

```
   private static class Node<E> {
        E item;
        Node<E> next;
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
```

关注点| 结论
---|---
是否允许为空 |允许 
是否允许为重复数据 |允许
是否有序 |是
是否线程安全 |是

#### 添加元素

在尾部插入：
```
   /**
     * Links e as last element.
     */
    void linkLast(E e) {
        final Node<E> l = last;
        final Node<E> newNode = new Node<>(l, e, null);
        last = newNode;
        if (l == null)
            first = newNode;  //默认为首尾节点
        else
            l.next = newNode;   //把新的节点录入为末节点
        size++;
        modCount++;
    }
```
可以看到，如果头节点为空，则把新增加的节点设置为头节点，否则就在尾部插入新增节点。

```
List<Integer> lists = new LinkedList<Integer>();
lists.add(5);
lists.add(4);
```
 首先调用无参构造函数，之后添加元素5和4。其结果图如下：  
 ![image](https://note.youdao.com/yws/public/resource/812b564b763769a068df3b97ac8017ce/xmlnote/BDA1A1F424B1494197D24804E4CEA206/13058)
 
 ####  addAll方法
 
```
 public boolean addAll(int index, Collection<? extends E> c) {
        checkPositionIndex(index);

        Object[] a = c.toArray();
        int numNew = a.length;
        if (numNew == 0)
            return false;

        Node<E> pred, succ;
        if (index == size) {
            succ = null;
            pred = last;
        } else {
            succ = node(index);
            pred = succ.prev;
        }

        for (Object o : a) {
            @SuppressWarnings("unchecked") E e = (E) o;
            Node<E> newNode = new Node<>(pred, e, null);
            if (pred == null)
                first = newNode;
            else
                pred.next = newNode;
            pred = newNode;
        }

        if (succ == null) {
            last = pred;
        } else {
            pred.next = succ;
            succ.prev = pred;
        }

        size += numNew;
        modCount++;
        return true;
    }
```
其中node函数是找出后一个节点的位置。
 
```
    Node<E> node(int index) {
        // assert isElementIndex(index);

        if (index < (size >> 1)) {
            Node<E> x = first;
            for (int i = 0; i < index; i++)
                x = x.next;
            return x;
        } else {
            Node<E> x = last;
            for (int i = size - 1; i > index; i--)
                x = x.prev;
            return x;
        }
    }
```
如果index小于size的一半，则从链表的前面往后面找，如果index大于size的一半，则从链表的后面往前面找。这种写法增加了查找的速度。
#### 删除元素
   元素的删除有从头删除和尾删除，下面讨论从头删除：
   
```
  private E unlinkFirst(Node<E> f) {
        // assert f == first && f != null;
        final E element = f.item;
        final Node<E> next = f.next;
        f.item = null;
        f.next = null; // help GC
        first = next;
        if (next == null)
            last = null;
        else
            next.prev = null;
        size--;
        modCount++;
        return element;
    }
```
首先找到需要删除的节点的下一个节点，然后将当前节点置空并设置成null，然GC回收，然后将next的prev设置成null形成双链表。
#### 关于循环遍历的问题
在ArrayList中习惯用get方法进行遍历，在LinkedList中是不适用的，首先来看看源码：

```
    Node<E> node(int index) {
        // assert isElementIndex(index);

        if (index < (size >> 1)) {
            Node<E> x = first;
            for (int i = 0; i < index; i++)
                x = x.next;
            return x;
        } else {
            Node<E> x = last;
            for (int i = size - 1; i > index; i--)
                x = x.prev;
            return x;
        }
    }
```
可以看到遍历的时候会从首部或者尾部进行遍历，它的时间复杂度达到了二次方系数，如果要遍历的话用foreach或者迭代器去遍历，原因是迭代器每次都保存了上次的节点位置，不需要重新从头部或者尾部去遍历。











 








 

    



  