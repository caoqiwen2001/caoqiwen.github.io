---
title: Hibernate中的外键1对1测试
date: 2016-09-27 00:28:21
tags:
  - Java
  - hexo
categories:
  - Java
type: tags
---

 &emsp;首先创建一个Manager和一个Department类，因为是1对1的映射，所以要在每个类的字段中包含不同的类。即Manager中包含Department的引用，
Department包含Manager的引用。
然后最主要的就是配置的问题，由于eclipse已经帮我们配置好了，只需稍微修改下即可，注意的点是在配置属性的时候要对应上类中的字段名。来看个配置：
```
<?xml version="1.0"?>
<!DOCTYPE hibernate-mapping PUBLIC "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
"http://hibernate.sourceforge.net/hibernate-mapping-3.0.dtd">
<hibernate-mapping>
    <class name="com.hiberate.one2one.foreign.Department" table="DEPARTMENTS">
        <id name="deptID" type="java.lang.Integer">
            <column name="DEPT_ID" />
            <generator class="native" />
        </id>
        <property name="deptName" type="java.lang.String">
            <column name="DEPT_NAME" />
        </property>
        <many-to-one name="manager" class="com.hiberate.one2one.foreign.Manager"
        column="MGR_ID" unique="true"></many-to-one>
    </class>
</hibernate-mapping>
```
&emsp;
* 要特别注意的是在配置property的时候一定要对上java文件中的字段，否则就会报错。
* 配置many-to-one时需要将unique="true"。
