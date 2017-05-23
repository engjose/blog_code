---
title: join链接查询
thumbnail: 'http://oayt7zau6.bkt.clouddn.com/mysql_basic.jpg'
date: 2017-05-21 19:27:58
updated: 2017-05-21 19:27:58
layout:
comments:
categories: mysql
tags: 链接查询
permalink:
---
#### 数据表结构
表结构一:
![enter image description here](http://oayt7zau6.bkt.clouddn.com/mysql_join_table1.jpg)
表结构二:
![enter image description here](http://oayt7zau6.bkt.clouddn.com/mysql_join_table2.jpg)
#### Join从句的分类
![enter image description here](http://oayt7zau6.bkt.clouddn.com/mysql_join%E4%BB%8E%E5%8F%A51.jpg)
#### Inner join
inner join表示将两张表进行关联起来,查询里面的交集部分
```sql
SELECT * FROM user1 INNER JOIN user2 WHERE user1.username = user2.name;
```
查询结果:
当查询*的时候是将两张表进行拼接起来
![enter image description here](http://oayt7zau6.bkt.clouddn.com/inner_join%E7%BB%93%E6%9E%9C1.jpg)
#### 左外链接用途
![enter image description here](http://oayt7zau6.bkt.clouddn.com/%E5%B7%A6%E5%A4%96%E9%93%BE%E6%8E%A51.jpg)
左外链接一般查询的是左表的所有和右表所匹配的数据,匹配不上的全是null
```sql
SELECT * FROM user1 LEFT JOIN user2 on user1.username = user2.name
```
查询结果:
![enter image description here](http://oayt7zau6.bkt.clouddn.com/%E5%B7%A6%E5%A4%96%E9%93%BE%E6%8E%A5%E6%9F%A5%E8%AF%A2%E7%BB%93%E6%9E%9C.jpg)
查询只存在于左表中的数据
```sql
SELECT * FROM user1 LEFT JOIN user2 on user1.username = user2.name WHERE user2.name IS NOT NULL
```
查询结果:
![enter image description here](http://oayt7zau6.bkt.clouddn.com/inner_join%E7%BB%93%E6%9E%9C1.jpg)
#### 右外链接
右外链接和左外链接是相对的,只不过是以右表为基准
![enter image description here](http://oayt7zau6.bkt.clouddn.com/%E5%8F%B3%E5%A4%96%E9%93%BE%E6%8E%A5.jpg)
#### 全连接
全连接表示可以查询出两个表所有的数据或两个表交集之外的数据
![enter image description here](http://oayt7zau6.bkt.clouddn.com/%E5%85%A8%E8%BF%9E%E6%8E%A5.jpg)
代码演示:
```sql
SELECT * FROM user1  LEFT JOIN user2 ON user1.username = user2.name UNION ALL SELECT *  FROM user1 RIGHT JOIN user2 ON user1.username = user2.name
```
查询结果:
![enter image description here](http://oayt7zau6.bkt.clouddn.com/%E5%85%A8%E8%BF%9E%E6%8E%A5%E7%BB%93%E6%9E%9C.jpg)
