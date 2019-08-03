---
title: Java辨析题二
date: 2019-07-14 19:30:49
tags:
- Java
categories:
- Java
- Basis
top:
mathjax:
---



{% cq %}

面向对象和面向过程的区别➕Java和c++的区别➕构造器constructor是否可以被override➕hashcode与equals

{% endcq %}

<!-- more -->

# **面向对象和面向过程的区别**

------

## **面向过程**

- 优点:性能比面向对象高,因为类调用时需要实例化,开销比较大,比较消耗资源.比如单片机,嵌入式开发,linux/unix等一般采用面向过程开发,性能是最重要的因素
- 缺点:没有面向对象易维护,易复用,易拓展

## **面向对象**

- 优点:易维护,易复用,易拓展,由于面向对象有封装,继承,,多态的特性,可以设计出低耦合的系统,使系统更加灵活,更加易于维护
- 缺点:性能比面向对象低

<br>

# **Java和c++的区别**

------

- 都是面向对象的语言,都支持封装,继承,多态
- Java不提供指针访问内存,程序内存更加安全
- Java类只支持单继承,c++支持多继承,但是Java可以实现多个接口
- Java有自动释放内存管理机制,不需要手动释放内存

<br>

# **构造器constructor是否可以被override**

------

父类的私有属性和构造方法不能被继承,所以说constructor不能被override,但是可以overload,所以一个类中有多个构造方法

<br>

# **hashcode与equals**

------

你重写过hashcode和equals吗.为什么重写equals是必须重写hashcode?  

## **hashcode()介绍**

- hashcode()的作用是获取哈希码,他实际上是方茴一个int整数.这个哈希码的作用是确定该对象在哈希表的索引位置.hashcode定义在jdk的object.java中,这意味着在java的任何类中都有hashcode()函数.  
- 哈希表存储的是键值对,他的特点是能根据"键"快速的检索出对应的值,这其中就利用了哈希码  

## **为什么要有hashcode**

> 我们以"hashcode如何检查重复"为例,说明为什么要用hashcode
>
> 当你把对象加入hashset时,hashset会计算对象的hashcode值来判断对象加入的位置,同时也会与其他已经加入的对象的hashcode值做对比,如果没有相符的hashcode,hashset会假设对象没有重复出现,如果发现有相同的hashcode值的对象,这时会调用equals()方法来检查hashcode对象是否真的相同.如果两者相同,hashset就不会让其加入操作成功.如果不同的话,就会重新散列到其他位置,这样的话会减少equals的次数,也就相应的提高了执行速度.

## **hashcode()和equals的相关规定**

1. 如果两个对象相等,则hashcode也一定相同
2. 两个对象相等,对两个对象调用equals的方法都返回true
3. 两个对象有相同的hashcode值,它们也不一定相等
4. equals方法被覆盖,则hashcode方法也必须被覆盖
5. hashcode的默认行为是对堆上的对象产生独特值,如果没有重写hashcode,则该class文件中的两个对象无论如何都不会相等(即使两个对象指向同一个数据)
