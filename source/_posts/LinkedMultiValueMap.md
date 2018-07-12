---
title: LinkedMultiValueMap源码解析
date: 2018-07-08 11:14:59
tags:
  - Java  
categories:  
  - Java集合框架  
type: tags
---
### LinkedMultiValueMap源码解析
最近在看HttpEntity的源码的时候，发现有一个类用到了MultiValueMap这个类，该类属于spring框架中自带的类型，首先看看它的基类实现：

```
public interface MultiValueMap<K, V> extends Map<K, List<V>> {
    @Nullable
    V getFirst(K var1);

    void add(K var1, @Nullable V var2);

    void addAll(K var1, List<? extends V> var2);

    void addAll(MultiValueMap<K, V> var1);

    void set(K var1, @Nullable V var2);

    void setAll(Map<K, V> var1);

    Map<K, V> toSingleValueMap();
}

```
发现它的类型就是继承自Map,只不过在它的基础上做了一层封装，便于后续的管理调用，再来看看LinkedMultiValueMap的实现：

```
public class LinkedMultiValueMap<K, V> implements MultiValueMap<K, V>, Serializable, Cloneable {
    private static final long serialVersionUID = 3801124242820219131L;
    private final Map<K, List<V>> targetMap;

    public LinkedMultiValueMap() {
        this.targetMap = new LinkedHashMap();
    }

    public LinkedMultiValueMap(int initialCapacity) {
        this.targetMap = new LinkedHashMap(initialCapacity);
    }

    public LinkedMultiValueMap(Map<K, List<V>> otherMap) {
        this.targetMap = new LinkedHashMap(otherMap);
    }

    @Nullable
    public V getFirst(K key) {
        List<V> values = (List)this.targetMap.get(key);
        return values != null ? values.get(0) : null;
    }

    public void add(K key, @Nullable V value) {
        List<V> values = (List)this.targetMap.computeIfAbsent(key, (k) -> {
            return new LinkedList();
        });
        values.add(value);
    }

    public void addAll(K key, List<? extends V> values) {
        List<V> currentValues = (List)this.targetMap.computeIfAbsent(key, (k) -> {
            return new LinkedList();
        });
        currentValues.addAll(values);
    }

    public void addAll(MultiValueMap<K, V> values) {
        Iterator var2 = values.entrySet().iterator();

        while(var2.hasNext()) {
            Entry<K, List<V>> entry = (Entry)var2.next();
            this.addAll(entry.getKey(), (List)entry.getValue());
        }

    }

    public void set(K key, @Nullable V value) {
        List<V> values = new LinkedList();
        values.add(value);
        this.targetMap.put(key, values);
    }

    public void setAll(Map<K, V> values) {
        values.forEach(this::set);
    }

    public Map<K, V> toSingleValueMap() {
        LinkedHashMap<K, V> singleValueMap = new LinkedHashMap(this.targetMap.size());
        this.targetMap.forEach((key, value) -> {
            singleValueMap.put(key, value.get(0));
        });
        return singleValueMap;
    }

    public int size() {
        return this.targetMap.size();
    }

    public boolean isEmpty() {
        return this.targetMap.isEmpty();
    }

    public boolean containsKey(Object key) {
        return this.targetMap.containsKey(key);
    }

    public boolean containsValue(Object value) {
        return this.targetMap.containsValue(value);
    }

    @Nullable
    public List<V> get(Object key) {
        return (List)this.targetMap.get(key);
    }

    @Nullable
    public List<V> put(K key, List<V> value) {
        return (List)this.targetMap.put(key, value);
    }

    @Nullable
    public List<V> remove(Object key) {
        return (List)this.targetMap.remove(key);
    }

    public void putAll(Map<? extends K, ? extends List<V>> map) {
        this.targetMap.putAll(map);
    }

    public void clear() {
        this.targetMap.clear();
    }

    public Set<K> keySet() {
        return this.targetMap.keySet();
    }

    public Collection<List<V>> values() {
        return this.targetMap.values();
    }

    public Set<Entry<K, List<V>>> entrySet() {
        return this.targetMap.entrySet();
    }

    public LinkedMultiValueMap<K, V> deepCopy() {
        LinkedMultiValueMap<K, V> copy = new LinkedMultiValueMap(this.targetMap.size());
        this.targetMap.forEach((key, value) -> {
            copy.put(key, (List)(new LinkedList(value)));
        });
        return copy;
    }

    public LinkedMultiValueMap<K, V> clone() {
        return new LinkedMultiValueMap(this);
    }

    public boolean equals(Object obj) {
        return this.targetMap.equals(obj);
    }

    public int hashCode() {
        return this.targetMap.hashCode();
    }

    public String toString() {
        return this.targetMap.toString();
    }
}

```
可以看到，它的本质上还是一个Map，只不过存放的value是List类型的而已。  
其中addAll 方法有多个重载，支持实现MultiValueMap接口类型的和参数为K,V类型的方法。  
而它的add函数运用了lamda表达式，当值为null的时候，会new一个LinkedList去填充，里面的默认属性为空。
其他的实现方法都与Map的操作不相伯仲。
### 总结  
如果在遇到有单个key和多个value的情况下，可以采用LinkedMultiValueMap这种方法来实现我们的需求。