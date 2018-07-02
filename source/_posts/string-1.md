---
title: Java的String源码解析
date: 2017-01-03 18:04:06
tags: 
  - Java
categories:
    - Java
type: tags
---

  ## java的属性 ## 
   
```
 private final char value[];  //final类型的char数组，则赋值之后是不能改变的。

    /** Cache the hash code for the string */
    private int hash; // Default to 0 //哈希值

    /** use serialVersionUID from JDK 1.0.2 for interoperability */
    private static final long serialVersionUID = -6849794470754667710L;
```
 ### 常用的构造函数

```
//默认给一个空间
 public String() {
        this.value = new char[0];
    }
    
  public String(String original) {
        this.value = original.value;
        this.hash = original.hash;
    }    
    //支持char数组初始化
    public String(char value[]) {
        this.value = Arrays.copyOf(value, value.length);
    }  
    //支持StringBuilder初始化
    public String(StringBuilder builder) {
        this.value = Arrays.copyOf(builder.getValue(), builder.length());
    }
    

```
 ###  常用到的函数
```
//返回char数组中对应的字符
  public char charAt(int index) {
        if ((index < 0) || (index >= value.length)) {
            throw new StringIndexOutOfBoundsException(index);
        }
        return value[index];
    }
```


```
//输入一个String返回一个char数组
    public char[] toCharArray() {
        // Cannot use Arrays.copyOf because of class initialization order issues
        char result[] = new char[value.length];
        System.arraycopy(value, 0, result, 0, value.length);
        return result;
    }
```
## 附上java中考到的一个算法题。
求String字符串中重复的字符算法题

```
	public static Map<Character, Integer> allTotalSum(String str) {

		char[] chars=str.toCharArray();//把String转换成char数组
		
		HashMap<Character, Integer> hash = new HashMap<Character, Integer>();

		for (int i = 0; i < chars.length; i++) {
			if (!hash.containsKey(chars[i])) {
				hash.put(chars[i], 1);
			} else {
				int count = hash.get(chars[i]) + 1;
				hash.put(chars[i], count);
			}
		}
		return hash;
	}

```

    













