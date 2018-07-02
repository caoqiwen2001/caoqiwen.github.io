---
title: ArrayList源码分析
date: 2017-12-28 08:54:05
tags:
  -  Java  
categories:  
  - Java集合框架  
type: tags
---
### ArrayList源码分析
ArrayList 是我们最常见的集合框架了，顾名思义，ArrayList以数组形式实现的组合，它是基于数组实现的。
它基本属性包括以下几点：

关注 | 结论
---|---
是否为空 | 允许
是否重复数据 | 允许
是否有序 | 有序
是否线程安全 | 非线程安全

最常用的几个方法莫过于添加、删除、判断是否相等等几个判断。首先来看看源码，一般关心的是它的默认容量，源码给出了具体的容量：

```
 /**
     * Default initial capacity.
     */
    private static final int DEFAULT_CAPACITY = 10;
```
#### 添加元素
再来看看它的add方法是怎样的：

```
  public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }
```
可以看到是往elementData这个数组中塞进数据了。
重点来看下ensureCapacityInternal这个方法：

```
 private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```
发现每次扩容的数据大小为原来数据容量的一半加原来的总容量的和，最大的容量为0x7fffffff。每次扩容的容量大小的限制是JDK开发人员经过慎重考虑才这样决定的（从内存空间和程序的运行效率来考虑）。确定了容量之后调用copyof方法将数据复制到新的数组里面，elementData会指向新的数组地址。

#### 删除元素
ArrayList的删除元素按照下标删除或者按照元素删除，它们调用的源码如下所示：

```
   public E remove(int index) {
        rangeCheck(index);

        modCount++;
        E oldValue = elementData(index);

        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work

        return oldValue;
    }
```
可以看到：删除数据是直接将后面的数据复制到前一个位置。最后一个元素的位置为null，好让垃圾回收机制回收不必要的垃圾。
#### 查找元素
ArrayList中的indexOf方法可以查找对应的元素，这个方法也用的比较多。

```
   public int indexOf(Object o) {
        if (o == null) {
            for (int i = 0; i < size; i++)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = 0; i < size; i++)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }
```
如果元素为null，则找到对应位置为null的index返回，如果数组中没有包含对应的元素，则直接返回-1。
#### 关于数据的序列化
ArrayList中实现了Serializable接口，源码中elementData是用transient进行修饰的。它的意思是不希望数组被序列化，因为数组中有些位置可能是空的，这些无用的数据是没必要进行序列化的。在ArrayList用writeObject方法。

```
  private void writeObject(java.io.ObjectOutputStream s)
        throws java.io.IOException{
        // Write out element count, and any hidden stuff
        int expectedModCount = modCount;
        s.defaultWriteObject();

        // Write out size as capacity for behavioural compatibility with clone()
        s.writeInt(size);

        // Write out all elements in the proper order.
        for (int i=0; i<size; i++) {
            s.writeObject(elementData[i]);
        }

        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
    }
```
接着需要从流中读取对应的数据，代码是这样子的：

```
    private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {
        elementData = EMPTY_ELEMENTDATA;

        // Read in size, and any hidden stuff
        s.defaultReadObject();

        // Read in capacity
        s.readInt(); // ignored

        if (size > 0) {
            // be like clone(), allocate array based upon size not capacity
            int capacity = calculateCapacity(elementData, size);
            SharedSecrets.getJavaOISAccess().checkArray(s, Object[].class, capacity);
            ensureCapacityInternal(size);

            Object[] a = elementData;
            // Read in all elements in the proper order.
            for (int i=0; i<size; i++) {
                a[i] = s.readObject();
            }
        }
    }
```
首先定义一个空的elementData数组，通过ObjectOutputStream的readObject方法将数据写回数组。  
这种方法减少了序列化文件的大小。相应的序列化的速度也可以加快。
#### 总结
- ArrayList底层是数组实现，是一种随机访问模式，查找速度非常快
- 删除元素的时候涉及到数组元素的复制，如果元素非常多，会非常消耗性能。






















