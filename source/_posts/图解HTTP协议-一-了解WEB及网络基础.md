---
title: 图解HTTP协议(一)---了解WEB及网络基础
thumbnail: 'https://example.com/image.jpg'
date: 2017-06-04 12:42:25
updated: 2017-06-04 12:42:25
layout:
comments:
categories:
tags:
permalink:
---
### TCP/IP
协议中存在着各式各样的内存,我们把互联网相关联的协议集合称之为TCP/IP家族.还有一种认为就是TCP/IP是TCP协议家族和IP协议家族的总称.
### TCP/IP协议的分层管理
TCP/IP重要的一点就是分层管理,按照层次可分为四层:应用层,传输层,网络层,数据链路层.
#### <font color='blue'>分层的优势</font>
值得一提的是层次化之后,设计也变得简单.处于应用层上的应用可以只考虑分给自己的任务,无需关注其他层的任务,比如数据怎么传输,传输给谁...
#### <font color='blue'>应用层的作用</font>
应用层决定了向用户提供服务时的通信服务.TCP/IP家族预存了各种通用的应用服务.比如ftp(文件传输协议),DNS域名解析协议.HTTP协议也属于其中.
