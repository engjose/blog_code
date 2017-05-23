---
title: BigDecimal
thumbnail: 'http://oayt7zau6.bkt.clouddn.com/basic.jpg'
date: 2017-05-21 12:00:33
updated: 2017-05-21 12:00:33
layout:
comments:
categories: javaSE
tags: BigDecimal数据类型
permalink:
---
### 大数据类型BigDecimal与Double
我们知道在浮点类型做运算的时候存在损失精度的问题,这是因为计算机底层是以二进制的方式存储数据的,所以我们在和金钱打交道的时候就应该避免采用Double,而应该用大数据类型BigDecimal.
### BigDecimal与Double的实验
#### 分别采用两种方式计算两个小数的加法运算:
```java
public static void testNaN() {
  int num1 = 10;
  float num2 = 10.1F;
  double num3 = 10.2;

  BigDecimal add1 = BigDecimal.valueOf(10.1).add(BigDecimal.valueOf(10.2));
  BigDecimal add2 = new BigDecimal("10.1").add(new BigDecimal("10.2"));
  BigDecimal add3 = BigDecimal.valueOf(num2).add(BigDecimal.valueOf(num3));
  Double add4 = num3 + num2;
  System.out.println(add1 + "****" + add2 + "****" + add3 + "****" + add4);
}
```
#### 执行结果
```java
20.3****20.3****20.300000381469727****20.300000381469726
```
#### 得出结论
浮点型的运算会存在误差问题,所以我们需要进行小数精确运算的时候应该采用BigDecimal,BigDecimal实现了任意精度的浮点型运算,保证了运算的正确性
#### 使用心得
我们在使用BigDecimal的时候应该采用构造方法,传入字符串的形式进行加减乘除计算
#### API的使用
<font color='blue'>add(other) </font>--- 和
<font color='blue'>subtract(other) </font>--- 差
<font color='blue'>multiply(other)</font> --- 积
<font color='blue'>divid(other)</font> --- 商
<font color='blue'>mod(oher) </font>--- 余
<font color='blue'>int compareTo(other) </font>--- 比较两个数,如果相等返回0,如果小于other返回负数,如多大于other返回正数
<font color='blue'>valueOf(other)</font> --- 将基数值转换为大数据类型
