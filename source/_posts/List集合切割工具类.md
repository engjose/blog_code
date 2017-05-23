---
title: List集合切割工具类
thumbnail: 'http://oayt7zau6.bkt.clouddn.com/%E5%B7%A5%E5%85%B7%E7%B1%BB.jpg'
date: 2017-05-15T22:55:00.000Z
updated: 2017-05-15T22:55:00.000Z
layout: null
comments: null
categories: 工具类
tags: 集合体系
permalink: null
---

#### 问题背景
我们在处理List集合的时候难免会遇到集合的size比较大,这时候我们就需要将一个比较大的list集合拆分成几个size比较小的集合进型处理,这样操作主要是考虑到了性能的问题,所以以下的工具类是针对于一个大的List集合进行拆分的过程
#### 代码实现
```java
/**
 * 切割List集合,将其分成等分的
 *
 * @param source
 * @param pageSize
 * @return
 */
public static <T> List<List<T>> splitList(List<T> source, Integer pageSize) {
  if (null == source || source.size() == 0) {
    return null;
  }
  int size = source.size();
  int num = size / pageSize + 1;

  // 切割list集合
  List<List<T>> target = new ArrayList<List<T>>();
  List<T> subList = null;
  for (int i = 0; i < num; i++) {
    int from = i * pageSize;
    //如果不是最后一个集合
    if (i != num - 1) {
      subList = source.subList(from, from + pageSize);
    } else {
        subList = source.subList(from, from + size % pageSize);
    }
    target.add(subList);
  }
    return target;
}
```
#### 代码理解 ####
1. num:表示大集合将要被切割的个数
2. pageSize表示每个小集合的size
3. subList = source.subList(from, from + pageSize):如果不是最后一个集合,我们需要切割from 到from + pageSize个
4. subList = source.subList(from, from + size % pageSize): 如果切割刀最后一个集合的时候,我们需要从from切割到from + size的余数(size % pageSize)

#### 功能测试 ####
```java
public static void main(String[] args) {
    List<Integer> list = new ArrayList<Integer>();
    list.add(1);
    list.add(2);
    list.add(3);
    list.add(4);
    list.add(5);
    list.add(5);
    list.add(6);

    List<List<Integer>> splitList = splitList(list, 2);
    System.out.println(splitList);
}
```
#### 测试运行结果
[[1, 2], [3, 4], [5, 5], [6]]
