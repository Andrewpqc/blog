---
title: Deque in STL
comments: true
date: 2017-09-14 09:19:05
updated: 2017-11-15 09:19:05
tags: [STL Deque]
categories: C/C++
permalink:
---


dequ(双端队列)和vector非常的相似，deque也是采取动态数组来管理容器中的数据，数据存储的地址都是连续的，可以对deque中的数据进行随机的访问。deque最重要的特征就是在容器的两端插入删除数据都非常的快速，这也是它和vector的主要的区别。(vector只是在尾端插入删除快速，在头部插入删除数据效率很低)。当程序中涉及到需要在容器的头部和尾部频繁的插入和删除数据时，deque就是最好的选择。同样的，作为动态数组的形式，deque必然也存在其局限性：在deque的中间插入和删除数据时效率很低，这一点和vector的情况是一致的。
## 用法

由于deque与vector及其的相似，这里就只总结一下它和vector在具体用法上的区别，其他的用法可以参考vector。
### 使用前提

使用前需要包含deque头文件，使用std命名空间。
### push_front()和pop_front()

deque在vector的基础之上增加了push_front()和pop_front()两个成员函数来实现对头部元素的快速操作。
