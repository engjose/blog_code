---
title: Thinking in Java(二)初始化与清理
thumbnail: 'http://oayt7zau6.bkt.clouddn.com/thinking_in_java_basic.jpg'
date: 2017-05-30 21:51:53
updated: 2017-05-30 21:51:53
layout:
comments:
categories: THINKING IN JAVA
tags: 读书笔记
permalink:
---
### 构造方法保证初始化
1. java会在用户操作对象之前自动调用相应的构造器,从而保证了初始化的进行
2. 构造器是一种特殊的方法,因为它没有返回值,这与返回值void有明显的不通,构造器不会返回任何的东西

### 区分方法的重载
#### <font color='blue'>重载的背景:</font><br/>
在任何的编程语言中,相同的词语可以表达多种不同的含义---它们被重载了
#### <font color='blue'>区分重载方法:</font><br/>
1. 每个重载的方法都必须有一个独一无二的参数列表
2. 其实参数顺序也可以区分,但是一般不这么做,因为这样使代码难以维护
3. 方法的重载与返回值类型无关

### this关键字
#### <font color='blue'>代表当前对象的引用</font>
假设你希望在方法的内部获取对当前对象的引用.由于这个引用是由编译器偷偷传递进去的,所以没有什么标识符可用;因此为了表示当前对象的引用,java引入了this关键字,它代表对当前对象的引用<br/>
如果方法要返回当前对象的引用:
```java
public class Leaf {
  int i = 0;
  Leaf increament() {
    i++;
    return this;
  }
}
```
#### <font color='blue'>在构造器中调用构造器</font>
如果在构造器中需要调用其他的构造器,我们就可以使用this(param1, param2, ...)的形式进行调用
### 垃圾回收器如何工作
在C和C++语言中在堆内存上分配对象是非常昂贵的;然而,垃圾回收器提高了对象分配的效率,存储空间的释放会影响存储空间的分配.这也意味着java从堆内存分配空间的速度可以和其他语言从堆栈上分配的速度相媲美.
#### <font color='blue'>垃圾回收---引用计数器算法</font>
引用计数器是一种简单但是速度很慢的垃圾回收技术:每个对象都含有一个引用计数器,当有引用链接到对象时计数器+1.当有引用离开对象或者置为null时,引用计数器-1.垃圾回收器会在全部对象的列表上遍历,当发现某个对象的引用计数器是0的时候,就释放其所占用的空间.引用计数器尝尝用来说明垃圾回收器的工作方式,但是从未应用于任何一种java虚拟机中.
#### <font color='blue'>垃圾回收---停止-复制</font>
在一些更快的模式中,垃圾回收并非采用引用计数的模式.他们的依据思想是:
对于任何活的对象,一定能最终追溯到其存活的堆栈或者静态区域中的引用.对于如何找到存活的对象,不同的虚拟机实现不同,有一种做法是停止-复制:显然这种做法是先停止程序的运行,然后将存活的对象复制到另外的一块空间中.
##### <font color='red'>停止-复制算法的弊端</font>
1. 把对象从一处搬运到另外一处,所指向它的引用都必须进行修正.
2. 需要两个分离的堆内存之间进行倒腾
3. 程序稳定之后可能产生少量的垃圾甚至没有垃圾,但是还是会停止-复制

#### <font color='blue'>垃圾回收---标记-清扫</font>
为了避免上面停止-复制产生的弊端,又提出来了另外的一种模式:标记-清扫.早起sun公司的虚拟机就采用了这个.标记-清扫所依据的思想仍然是从堆栈和存储区域触发,遍历所有的引用,进而找到所有存活的对象,每当它找到一个就会打一个标记,这个过程不会回收任何的对象,只有全部标记完成之后,清理动作才会开始,在清理的过程中没有打标记的对象将会被释放,不会发生任何的复制动作,同停止-复制一样标记-清扫不是在后台运行的,也必须得停止程序.
##### <font color='red'>标记-清扫算法的弊端</font>
1. 垃圾清除之后剩下的堆内存不是连续的
2. 一般标记-清扫速度非常慢,之后当产生垃圾非常少或者不产生垃圾的时候,它的速度就很快了

### 综合考虑---自适应
其实上面的暂停-复制和标记-清扫两种算法在虚拟机中都会用到.java虚拟机会进行自动监视:如果内存对象变动比较大,出现很多内存碎片,有大块的内存需要回收就采用停止-复制算法;但是当虚拟机发现所有的对象都很稳定,垃圾回收的速率比较低,就会切换到标记-清扫方式.这种自我调节的方式就是自适应技术.
### 静态初始化
无论创建多少个对象,静态数据只会占用一份存储空间.static关键字不能用于局部变量
#### <font color='blue'>静态数据初始化时机</font>
当首次生成这个类的一个对象时或者首次访问那个类的静态数据成员时,进行静态区域的加载