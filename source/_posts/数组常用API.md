---
title: 数组常用API
thumbnail: 'http://oayt7zau6.bkt.clouddn.com/basic.jpg'
date: 2017-05-21 15:01:11
updated: 2017-05-21 15:01:11
layout:
comments:
categories: javaSE
tags: 集合体系
permalink:
---
### 数组注意事项
对于数组的定义和一般使用技巧,我们肯定都是比较熟悉了.这里主要介绍一下数据的一些好用API以及注意事项
#### 数组的基本使用---创建数组
- 当创建一个数字型的数组时,所有元素都初始化为0
- 当创建boolean类型的数组时,所有元素都初始化为false
- 当创建一个对象数组时,所有元素都初始化为null,例如:
```java
//创建一个包含十个字符串的数组,但是每一个的初始化字符串都是null
String[] str = new String[10]
```
#### 数组工具类常用API---Arrays
<font color='blue'>static <T> T[] copyOf(T[] original, int newLength) </font>---将original数组拷贝newLength个到一个新的数组中,方法的返回值是一个新的数组
<font color='blue'>static void sort(type[] a)</font>---将数组进行排序,使用的是快排,效率比较高
<font color='blue'>static int binarySearch(type[] a, type key)</font>---在数组a中进行二分查找key,如果查找成功则返回下标值,否则返回一个负数
<font color='blue'>static void fill(type[] a, type v)</font>---将数组的所有数据元素赋值为v
<font color='blue'>boolean equals(type[] a, type[] b)</font>---如果两个数组大小相同,并且下标相同的元素都相同返回true,否则返回false
