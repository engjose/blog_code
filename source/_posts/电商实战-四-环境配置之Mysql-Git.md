---
title: 电商实战(四)---环境配置之Mysql|Git
thumbnail: 'http://oayt7zau6.bkt.clouddn.com/%E7%94%B5%E5%95%861.jpg'
date: 2017-06-06 21:29:11
updated: 2017-06-06 21:29:11
layout:
comments:
categories: 项目实战
tags: 电商后台
permalink:
---
### Mysql的安装
#### <font color='blue'>下载安装Mysql</font>
1.打开终端输入--- 默认版本5.1.73
```sql
[root@centos-linux developer]# sudo yum -y install mysql-server
```
2.配置mysql的默认编码
```sql
-- 编辑/etc/my.cnf文件
[root@centos-linux etc]# sudo vim /etc/my.cnf

-- 在symbolic-links=0下面添加如下的配置
symbolic-links=0
default-character-set = utf8 -- 配置项
```
3.设置mysql随系统自动启动并查看
```sql
[root@centos-linux etc]# sudo chkconfig mysqld on
-- 2-5都是on状态表示会随机器启动
[root@centos-linux etc]# sudo chkconfig --list mysqld
mysqld          0:off   1:off   2:on    3:on    4:on    5:on    6:off
```
4.启动mysql服务
```sql
[root@centos-linux etc]# sudo service mysqld start
```
5.登录查看
```sql
-- 第一次使用非密码登录
[root@centos-linux etc]# mysql -u root

-- 查看授权用户
mysql> select user,host from mysql.user;
+------+---------------------+
| user | host                |
+------+---------------------+
| root | 127.0.0.1           |
|      | centos-linux.shared |
| root | centos-linux.shared |
|      | localhost           |
| root | localhost           |
+------+---------------------+

-- 删除匿名用户
mysql> delete from mysql.user where user='';
Query OK, 2 rows affected (0.00 sec)

mysql> select user,host from mysql.user;
+------+---------------------+
| user | host                |
+------+---------------------+
| root | 127.0.0.1           |
| root | centos-linux.shared |
| root | localhost           |
+------+---------------------+
3 rows in set (0.00 sec)

-- 退出mysql
mysql> exit
```
6.配置防火墙
这里同时把Nginx和Tomcat的端口也进行了配置,配置项如下:
```sql
[root@centos-linux etc]# sudo vim /etc/sysconfig/iptables

#nginx tomcat
-A INPUT -p tcp -m state --state NEW -m tcp --dport 80 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 8080 -j ACCEPT

#mysql
-A INPUT -p tcp -m state --state NEW -m tcp --dport 3306 -j ACCEPT

-- 重启防火墙
[root@centos-linux etc]# sudo service iptables restart
iptables: Applying firewall rules:                         [  OK  ]
```
7.设置账号
```sql
-- 登录mysql
[root@centos-linux etc]# mysql -u root
mysql> insert into mysql.user(Host,User,Password) values("localhost","panyuanyuan",password("123"));
Query OK, 1 row affected, 3 warnings (0.01 sec)

-- 查看用户表
mysql> select user,host from mysql.user;
+-------------+---------------------+
| user        | host                |
+-------------+---------------------+
| root        | 127.0.0.1           |
| root        | centos-linux.shared |
| panyuanyuan | localhost           |
| root        | localhost           |
+-------------+---------------------+
4 rows in set (0.00 sec)
```
8.创建一个数据库并设置权限
```sql
mysql> create database `mmall` default character set utf8 collate utf8_general_ci;
Query OK, 1 row affected (0.00 sec)

-- 给数据库设置权限
mysql> grant select,delete,create on mmal.* to panyuanyuan@'%' identified by '123' with grant option;
Query OK, 0 rows affected (0.00 sec)

-- 设置密码
mysql> set password for root@localhost=password('123');
Query OK, 0 rows affected (0.00 sec)
```
9.链接测试,用本机的Navicat链接CentOS的数据库链接
![enter image description here](http://oayt7zau6.bkt.clouddn.com/Navicat.jpg)
#### <font color='blue'>安装总结</font>
至此我们安装配置mysql完毕,接下来安装git
### Git的安装的安装与配置
1.下载GIT,本次采用的下载地址是http://learning.happymmall.com/git/git-v2.8.0.tar.gz
```sql
[root@centos-linux developer]# wget http://learning.happymmall.com/git/git-v2.8.0.tar.gz
```
2.安装git的依赖包
```sql
yum install curl
yum install curl-devel
yum install zlib-devel
yum install openssl-devel
yum install perl
yum install cpio
yum install expat-devel
yum install gettext-devel
```
3.解压git安装包
```sql
[root@centos-linux developer]# tar -zxvf git-v2.8.0.tar.gz 
```
4.安装git
```sql
-- 进入到git的解压目录
[root@centos-linux git-2.8.0]# cd git-2.8.0/

-- 编译git
[root@centos-linux git-2.8.0]# sudo make prefix=/usr/local/ all

-- 安装git
[root@centos-linux git-2.8.0]# sudo make prefix=/usr/local/ install

-- 验证git的版本
[root@centos-linux git-2.8.0]# git --version
git version 2.8.0
```
5.配置GIT
```sql
-- 输入以下命令一路回车生成ssh key
[root@centos-linux git_down]# ssh-keygen -t rsa -C "panyuanyuan1024@163.com"
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
43:a7:76:72:ce:58:8a:ca:cc:cb:0c:19:ae:3b:3c:71 panyuanyuan1024@163.com
The key's randomart image is:
-- 将私钥告诉本地系统
[root@centos-linux git_down]# ssh-add ~/.ssh/id_rsa
Could not open a connection to your authentication agent.

-- 解决以上出现的问题
[root@centos-linux git_down]# eval `ssh-agent`
Agent pid 15193

-- 再次输入以下命令,问题解决
[root@centos-linux git_down]# ssh-add ~/.ssh/id_rsa
Identity added: /root/.ssh/id_rsa (/root/.ssh/id_rsa)

-- 输入以下命令复制公钥
[root@centos-linux git_down]# cat ~/.ssh/id_rsa.pub 
```
6.将公钥配置到马云上面
![enter image description here](http://oayt7zau6.bkt.clouddn.com/%E9%A9%AC%E4%BA%91.jpg)
7.测试git配置
此时我们就可以验证git的配置是否成功,如果克隆项目成功表示我们的配置没有问题
```sql
[root@centos-linux learning]# git clone git@git.oschina.net:jose_uncle/learning.git
```



