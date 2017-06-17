---
title: 电商实战(三)---环境配置之NGINX
thumbnail: 'http://oayt7zau6.bkt.clouddn.com/%E7%94%B5%E5%95%861.jpg'
date: 2017-06-06 21:29:11
updated: 2017-06-06 21:29:11
layout:
comments:
categories: 项目实战
tags: 电商后台
permalink:
---
### NGINX简介与安装
#### <font color='blue'>NGINX是什么</font>
Ngix是一个轻量级的WEB服务器,也是一个反向代理服务器,是俄罗斯人开发的.
#### <font color='blue'>NGINX安装</font>
1.首先安装Nginx环境依赖gcc.打开终端查看机器是否已经安装gcc
```sql
[root@centos-linux ~]# gcc -v
gcc version 4.4.7 20120313 (Red Hat 4.4.7-17) (GCC)
```
如果没有安装的话,执行下面命令进行安装:
```sql
yum install gcc
```
2.安装Nginx的依赖pcre
```sql
[root@centos-linux ~]# sudo yum install pcre pcre-devel
```
3.安装Nginx的依赖zlib
```sql
[root@centos-linux ~]# sudo yum install zlib zlib-devel
```
4.安装Nginx的依赖openssl
```sql
[root@centos-linux ~]# sudo yum install openssl openssl-devel
```
5.下载Nginx,这里采用的地址是:http://learning.happymmall.com/nginx/linux-nginx-1.10.2.tar.gz
```sql
[root@centos-linux developer]# wget http://learning.happymmall.com/nginx/linux-nginx-1.10.2.tar.gz

-- 下载完成之后解压(这里的下载路径是/developer)
[root@centos-linux developer]# tar -zxvf linux-nginx-1.10.2.tar.gz
```
6.配置文件的检查和编译安装
```sql
-- 进入到解压后的Nginx文件目录进行文件检查
[root@centos-linux nginx-1.10.2]# sudo ./configure
-- 检查完成之后输入sudo make命令进行编译
[root@centos-linux nginx-1.10.2]# sudo make
-- 编译完成之后进行安装
[root@centos-linux nginx-1.10.2]# sudo make install

-- 安装完成之后查看安装的目录
[root@centos-linux nginx-1.10.2]# whereis nginx
nginx: /usr/local/nginx
```
7.启动Nginx
```sql
-- 进入到Nginx的安装目录下面的/sbin目录
[root@centos-linux nginx-1.10.2]# cd /usr/local/nginx/
[root@centos-linux nginx]# ll
total 16
drwxr-xr-x. 2 root root 4096 Jun 10 15:12 conf
drwxr-xr-x. 2 root root 4096 Jun 10 15:12 html
drwxr-xr-x. 2 root root 4096 Jun 10 15:12 logs
drwxr-xr-x. 2 root root 4096 Jun 10 15:12 sbin
[root@centos-linux nginx]# cd sbin/

-- 启动Nginx
[root@centos-linux sbin]# ./nginx

-- 查看Nginx所运行的进程
[root@centos-linux sbin]# ps aux| grep nginx 
root     17481  0.0  0.0  23968   824 ?        Ss   15:16   0:00 nginx: master process ./nginx
nobody   17482  0.0  0.1  24388  1404 ?        S    15:16   0:00 nginx: worker process
root     17605  0.0  0.0 103312   880 pts/2    S+   15:16   0:00 grep nginx
```
8.安装验证,因为Nginx默认是80端口,所以这里关闭防火墙直接访问ip即可
![enter image description here](http://oayt7zau6.bkt.clouddn.com/Nginx%E5%AE%89%E8%A3%85.jpg)


### NGINX配置
#### <font color='blue'>NGINX配置服务转发</font>
1.进入到Nginx的安装目录的conf目录下,并创建vhost文件夹(方便管理)
```sql
[root@centos-linux ~]# cd /usr/local/nginx/conf/
```
2.编辑nginx.conf文件添加如下配置
```sql
[root@centos-linux conf]# vim nginx.conf

-- 添加如下配置,在HHTPS的上面配置
include vhost/*.conf;
# HTTPS server
```
3.如果没有外网只在本机器局域网访问的话配置本机的host文件(不是linux)
```sql
shizhengyangdeMacBook-Pro% sudo vim hosts
-- 添加如下的配置,将本机的ip和域名进行绑定
10.211.55.3 www.learning.com
```
4.在nignx安装目录下的conf目录下创建vhost目录,并创建域名.conf进行配置
```sql
[root@centos-linux sbin]# cd /usr/local/nginx/conf/vhost/
[root@centos-linux vhost]# vim www.learning.com.conf
server {
    listen 80;
    autoindex on;
    server_name www.learning.com;
    access_log /usr/local/nginx/logs/access.log combined;
    index index.html index.htm index.jsp index.php;
    if ( $query_string ~* ".*[\;'\<\>].*" ){
        return 404;
    }
    location / {
       proxy_pass http://127.0.0.1:8080;
       add_header Access-Control-Allow-Origin *;
    }
}
```
5.配置完成之后重启Nginx服务器
```sql
-- 进入到Nginx的安装目录下的sbin下执行命令
[root@centos-linux vhost]#/usr/local/nginx/sbin
[root@centos-linux sbin]# ./nginx -s reload -- 重启Nginx
```
6.启动测试
打开本机浏览器(修改hosts文件的机器)访问www.learning.com,发下已经被转发
![enter image description here](http://oayt7zau6.bkt.clouddn.com/nginx%E6%B5%8B%E8%AF%95.jpg)
#### <font color='blue'>NGINX配置目录转发</font>
1.因为在本机局域网环境测试,所以这里要配置本机的hosts文件(不是linux)
```sql
10.211.55.3 www.learning.com
10.211.55.3 www.learning.image.com -- 新添加的图片的地址
```
2.配置linux的Nginx反向代理配置
```sql
-- 找到nginx安装目录下的conf下的vhost文件夹
[root@centos-linux sbin]# cd /usr/local/nginx/conf/vhost/
-- 创建文件
[root@centos-linux sbin]#vim www.learning.image.com.conf
-- 编辑www.learning.image.com.conf文件
server {
    listen 80;
    autoindex off;
    server_name www.learning.image.com;
    access_log /usr/local/nginx/logs/access.log combined;
    index index.html index.htm index.jsp index.php;
    if ( $query_string ~* ".*[\;'\<\>].*" ){
        return 404;
    }
    location / {
       root /ftpfile/image/; -- 路径指向FTP文件下的image文件夹
       add_header Access-Control-Allow-Origin *;
    }
}
```
3.在FTP的文件夹下创建image文件夹,并上传图片1.jpg
![enter image description here](http://oayt7zau6.bkt.clouddn.com/ftp%E6%96%87%E4%BB%B6%E5%A4%B9.jpg)
4.重启Nginx服务器
```sql
[root@centos-linux sbin]# cd /usr/local/nginx/sbin
[root@centos-linux sbin]# ./nginx -s reload
```
5.访问测试,在本机(修改hosts文件的机器)访问www.learning.image.com/1.jpg
![enter image description here](http://oayt7zau6.bkt.clouddn.com/nginx_test.jpg)
到这里Nginx的安装和配置已经完成,并且与之前的vsftpd服务器已经形成了一个文件服务器,接下来下一章节我们安装Mysql和git
