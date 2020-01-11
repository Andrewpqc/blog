---
title: List in STL
comments: true
date: 2017-09-13 09:11:51
updated: 2017-11-15 09:11:51
tags: [List STL]
categories: C/C++
permalink:
---
与vector一样，list也是STL模板库中提供的序列式容器之一。不同的是，list是由双向链表来实现的，元素与元素之间的地址不一定是连续的，，元素之间的序列关系由指针维护。list支持前后两种移动方式。与vector不同,list不支持随机访问，不可以使用operator[] 或at()来获取元素。list的优势在于任何位置执行插入和删除操作都非常迅速，因为改变的仅仅是指针与节点的链接而已。list的使用需要头文件和std命名空间。
## list对象的定义与构造函数

list对象的定义与初始化一般有一下几种方法：
``` c++
list<TYPE> mylist;　　　　　　　　//初始化一个空lkongist
list<TYPE> mylist(size);          //初始化一个初始大小为size的空list
list<TYPE> mylist(size,vaule);    //初始化一个装有size个vaule的list
list<TYPE> mylist(elselist);      //复制构造函数
list<TYPE> mylist(first,last);　　//将由迭代器指定的其他list中的多个元素复制以构成新list
```
## 基础成员函数
### 元素的赋值

list模板类提供了两个成员函数`push_back()`,`push_front()`,分别用来将新的节点追加到list的尾部和插入到list的头部。与之对应的`pop_back()`和`pop_front()`函数则是在相应位置做删除操作。在vector中只提供了`push_back()`函数和`pop_back()`函数，主要是因为vector中在头部插入和删除元素效率很低，所以就没有直接提供相应操作，这也说明了list是双向链表。
### list容器的容量

和list容量相关的主要是下面的几个函数：
``` c++
mylist.size();         //返回容器中元素的个数
mylist.max_size();　　//返回容器中最多可以容纳节点的数目，一般是一个非常大的数字，我们可以不用管
mylist.resize();　　　//重新设置容器的大小
```
### 迭代器相关

list模板类包含的和迭代器相关的函数主要有：begin(),end(),rbegin(),rend(),back(),front()
下面我们通过一个程序演示他们的使用：
``` c++
#include<iostream>
#include<algorithm>
#include<list>
using namespace std;
void printList(int& i){
    cout<<i<<" ";
}
int main(void){
    list<int> myIntlist;
    myIntlist.push_back(1);
    myIntlist.push_back(2);
    myIntlist.push_back(3);
    myIntlist.push_back(4);
    //顺序打印
    for_each(myIntlist.begin(),myIntlist.end(),printList);
    cout<<endl;
    //反序打印
    for_each(myIntlist.rbegin(),myIntlist.rend(),printList);
    cout<<endl;
    list<int>::reverse_iterator riter;
    riter=myIntlist.rbegin();
    list<int>::iterator iter;
    iter=myIntlist.begin();
    cout<<*riter<<endl;
    cout<<*iter<<endl;
    
    int last=myIntlist.back();
    cout<<last<<endl;
    
    return 0;
}
```
输出：
```
1 2 3 4
4 3 2 1
4
1
4
```
### 判断链表是否为空

使用list对象的empty()方法判断链表是否为空，该函数返回bool值

### list元素的存取与访问

list对象不能使用operator[]和at()来实现随机访问，但是可以用迭代器来进行元素的访问。
### 元素的插入与删除

可以实现list元素插入与删除的函数有：
``` c++
//在链表尾端插入
push_back();
//删除尾端的元素
pop_back();
//链表头部插入
push_front();
//移除头部的元素
pop_front();
//可以实现任意位置的插入
insert()
//任意位置的删除
erase()
//清空链表
clear()
//下面两个成员函数也可以用来删除链表中的元素
remove()
remove_if()
```
### 运算符函数

operator==,operator<,operator>,operator<=,operator>=,operator!=这些运算符函数均可以用于两个list对象之间的比较，前提是参与比较的两个list对象的格式应该完全相同。
### 合并两个list

list还提供了成员函数merge()成员函数，用于将两个具有相同格式的list对象合并,同时提供了sort()成员函数用于将成员排序:
``` c++
#include <iostream>
#include <list>
#include <string>
#include <algorithm>
using namespace std;
void printList(string s){
    cout<<s<<" ";
}
void printList2(int s){
    cout<<s<<" ";
}
int main (void){
    list<string> L1,L2;
    L1.push_back("abc");
    L1.push_back("def");
    L1.push_back("ghi");
    for_each(L1.begin(),L1.end(),printList);
    cout<<endl;
    L2.push_back("jkl");
    L2.push_back("mno");
    L2.push_back("pqr");
    for_each(L2.begin(),L2.end(),printList);
    cout<<endl;
    L1.merge(L2,greater<string>());
    for_each(L1.begin(),L1.end(),printList);
    cout<<endl;
    list<int> L3,L4;
    L3.push_back(2);
    L3.push_back(3);
    L3.push_back(1);
    for_each(L3.begin(),L3.end(),printList2);
    cout<<endl;
    //从小到大排序
    L3.sort();
    for_each(L3.begin(),L3.end(),printList2);
    cout<<endl;
    //从大到小【排序
    L3.sort(greater<int>());
    for_each(L3.begin(),L3.end(),printList2);
    cout<<endl;
    return 0;
}
```
输出：
```
abc def ghi
jkl mno pqr
jkl mno pqr abc def ghi
2 3 1
1 2 3
3 2 1
```
### unique()重复的元素只保留一个
``` c++
#include <iostream>
#include <list>
#include <string>
#include <algorithm>
using namespace std;
void printList(string s){
    cout<<s<<" ";
}
void printList2(int s){
    cout<<s<<" ";
}
int main (void){
    list<int> L1;
    L1.push_back(1);
    L1.push_back(2);
    L1.push_back(1);
    for_each(L1.begin(),L1.end(),printList2);
    cout<<endl;
    //这里要先sort再unique(),不然的话没有效果
    L1.sort();
    L1.unique();
    for_each(L1.begin(),L1.end(),printList2);
    cout<<endl;
    return 0;
}
```
输出：
```
1 2 1
1 2
```
### reverse()反转顺序
``` c++
#include <iostream>
#include <list>
#include <string>
#include <algorithm>
using namespace std;
void printList(string s){
    cout<<s<<" ";
}
void printList2(int s){
    cout<<s<<" ";
}
int main (void){
    list<int> L1;
    L1.push_back(1);
    L1.push_back(2);
    L1.push_back(3);
    for_each(L1.begin(),L1.end(),printList2);
    cout<<endl;
    //这里要先sort再unique(),不然的话没有效果
    //L1.sort();
    L1.reverse();
    for_each(L1.begin(),L1.end(),printList2);
    cout<<endl;
    return 0;
}
```
输出：
```
1 2 3
3 2 1
```