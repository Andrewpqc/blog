---
title:  单链表
comments: true
date: 2017-09-28 13:09:24
updated: 2017-11-16 13:09:24
tags: [Linked List]
categories: Algorithm
permalink:
---
这一篇博客总结一下单链表常见方法的实现。相比于动态数组，对于单链表来说，它存储的元素并不要求在内存上连续，元素与元素之间通过指针串联起来。这样有许多好处，比如在单链表中间位置插入和删除元素的效率非常高，同时，它也有不足之处，它无法像动态数组那样可以通过下标直接访问。

链表实现的关键在于指针。一个节点除了保存着数据本身，还保存着指向下一个节点的指针。这样我们就可以通过一个节点找到下一个节点。从而将所有数据连接起来。
![单链表示意图](/images/单链表.png)
如上图所示，首先有一个头结点，该节点的数据域并不存放有效数据，指针域指向下一个节点。而最后的一个节点的指针域指向NULL。下面我来总结一下单链表的常见操作。

# 面向过程实现单链表操作
## 定义结构体
``` c++
typedef struct Node{
    int data;
    struct Node* next;    
}Node,*NodePtr;
```
结构体中存储着int类型的数据，和指向自身类型的指针。

## 初始化链表
``` c++
NodePtr createList(void){
    NodePtr headPtr=new Node;
    if(headPtr==NULL){
        cout<<"分配内存失败"<<endl;
        return NULL;
    }
    headPtr->next=NULL;
    return headPtr;
}
```
这里分配头节点的内存空间，并且将头结点中的指针赋值为NULL,防止其指向了不明的变量。由于头结点不存放实际的数据，所以并没有对data赋值。最后返回指向头结点的指针(即头指针)。

## 计算长度
``` c++
int listSize(NodePtr phead){
    int i=0;
    //跳过头结点
    phead=phead->next;
    while(phead!=NULL){
        i+=1;
        phead=phead->next;
    }
    return i;
}
```
这里计算的是链表中有效节点的个数，由于头结点并不包含有效数据，所以直接跳过了头节点(phead=phead->next),while循环的条件为phead!=NULL,这是因为我们在设计链表的时候总是会将最后一个节点中的指针指向NULL,所以当phead==NULL时，链表也就循环完了。

## 打印链表
``` c++
void printList(NodePtr phead){
    phead=phead->next;
    cout<<"打印所有节点：";
    while(phead!=NULL){
        cout<<phead->data<<" ";
        phead=phead->next;
    }
    cout<<endl;
}
```
这里与上方求链表的长度是一致的。

## 增加数据
``` c++
void addNode(NodePtr phead){
    int a=0;
    cout<<"输入你要增加的节点数："<<endl;
    cin>>a;
    for (int i=0;i<a;i++){
        NodePtr ptemp=new Node;
        if(ptemp==NULL) {
        cout<<"分配内存失败"<<endl;
        return;
        }
        ptemp->next=NULL;
        cout<<"输入第"<<i+1<<"个节点的数据值"<<endl;
        cin>>ptemp->data;
        phead->next=ptemp;
        phead=phead->next;
    } 
}
```
这里以循环的方式增加链表的节点个数。通过收集控制台的输入来决定增加的数据的个数和具体的数据。每一个节点的增加都要经历如下的步骤：

- 分配内存空间
- 赋值，数据域赋对应值，指针域赋值为NULL
- 链接，将新节点链接到链表的尾端

下面的是一个节点加入到链表尾端的示意图:
![尾端添加节点示意图](/images/链表尾部插入.png)

## 在头部插入节点

``` c++
void insertAtHead(NodePtr phead){
    NodePtr ptemp=new Node;
    cout<<"输入在头部插入的数据值"<<endl;
    cin>>ptemp->data;
    ptemp->next=phead->next;
    phead->next=ptemp;
}
```
![头部插入节点示意图](/images/头部插入.png)

## 尾部插入
``` c++
void insertAtTail(NodePtr phead){
    //先创建需要插入的节点
    NodePtr pinsert=new Node;
    cout<<"输入在尾部插入的数据"<<endl;
    cin>>pinsert->data;
    pinsert->next=NULL;
    NodePtr ptemp=phead;
    phead=phead->next;
    while(phead!=NULL){
        phead=phead->next;
        ptemp=ptemp->next;
    }
    ptemp->next=pinsert;
}
```
## 匹配并删除第一个匹配节点
``` c++
void deleteOneNode(NodePtr phead,int _data){
    NodePtr ptemp;
    ptemp=phead;
    phead=phead->next;
    //空链表的情况
    if(phead==NULL){
        cout<<"链表为空！"<<endl;
    }
    while(phead!=NULL){
        //找到了
        if(phead->data==_data){
            ptemp->next=phead->next;
            delete phead;
            break;
        }
        //没找到
        phead=phead->next;
        ptemp=ptemp->next;
    }
}
```
这里我们需要一前一后的指针来分别指向一前一后两个节点。前面的指针负责寻找目标节点，当找到了目标节点之后就将目标节点后一个节点的指针域指向目标节点的下一个节点，这样目标节点也就不再属于该链表了。同时释放目标节点的内存。跳出循环，这样也就删除了匹配的第一个节点。

## 匹配并删除所有匹配节点
``` c++
void deleteAllNode(NodePtr phead,int _data){
    NodePtr ptemp;
    ptemp=phead;
    phead=phead->next;
    //空链表的情况
    if(phead==NULL){
        cout<<"链表为空！"<<endl;
        return;
    }
    while(phead!=NULL){
        if(phead->data==_data){
            ptemp->next=phead->next;
            delete phead;
            phead=ptemp->next;
        }
        else{
        phead=phead->next;
        ptemp=ptemp->next;
        }
``` 
这里与上面的删除第一个匹配的节点的情况差不多，只不过在删除了一个节点之后还需要保持phead和ptemp的这种一前一后的关系，让他继续去删除所有匹配的节点。这里大家可能会有点奇怪，明明phead已经被释放了，为什么还可以将ptemp->next赋给phead呢？其实这里需要注意的是delete phead只是释放了phead所指向的那一块内存，并没有释放phead变量本身。

## 销毁整个链表
``` c++
void destoryList(NodePtr phead){
    if(phead->next!=NULL){
        destoryList(phead->next);
    }
    delete phead; 
}
```
这里使用递归的方式释放了整个链表。

## 测试
``` c++
int main(void){
    //创建头结点
    NodePtr phead=createList();
    //测试增加节点
    addNode(phead);
    printList(phead);
    cout<<"节点数量：";
    cout<<listSize(phead)<<endl;
    // //测试从尾部插入一个节点
    insertAtTail(phead);
    cout<<"从尾部插入节点后："<<endl;
    printList(phead);
    cout<<"节点数量：";
    cout<<listSize(phead)<<endl;
    insertAtHead(phead);
    cout<<"从头插入节点后："<<endl;
    printList(phead);
    cout<<"节点数量：";
    cout<<listSize(phead)<<endl;
    //测试删除第一个匹配的节点
    int a;
    cout<<"请输入你要删除的元素的值(只删除第一个匹配的值)："<<endl;
    cin>>a;
    deleteOneNode(phead,a);
    cout<<"删除第一个"<<a<<"后："<<endl;
    printList(phead);
    cout<<"节点数量：";
    cout<<listSize(phead)<<endl;
    //测试删除所有匹配的节点
    int b;
    cout<<"请输入你要删除的元素的值(将删除所有匹配的值)："<<endl;
    cin>>b;
    cin>>b;
    deleteAllNode(phead,b);
    cout<<"删除所有的"<<b<<"后："<<endl;
    printList(phead);
    cout<<"节点数量：";
    cout<<listSize(phead)<<endl;
    destoryList(head);
    return 0;
}
```
输出结果
``` c++
输入你要增加的节点数：
10
输入第1个节点的数据值
1
输入第2个节点的数据值
2
输入第3个节点的数据值
5
输入第4个节点的数据值
3
输入第5个节点的数据值
2
输入第6个节点的数据值
1
输入第7个节点的数据值
4
输入第8个节点的数据值
5
输入第9个节点的数据值
6
输入第10个节点的数据值
8
打印所有节点：1 2 5 3 2 1 4 5 6 8 
节点数量：10
输入在尾部插入的数据
100
从尾部插入节点后：
打印所有节点：1 2 5 3 2 1 4 5 6 8 100 
节点数量：11
输入在头部插入的数据值
-1
从头插入节点后：
打印所有节点：-1 1 2 5 3 2 1 4 5 6 8 100 
节点数量：12
请输入你要删除的元素的值(只删除第一个匹配的值)：
2
删除第一个2后：
打印所有节点：-1 1 5 3 2 1 4 5 6 8 100 
节点数量：11
请输入你要删除的元素的值(将删除所有匹配的值)：
1
删除所有的1后：
打印所有节点：-1 5 3 2 4 5 6 8 100 
节点数量：9
```
# 面向对象实现单链表操作
我同样的用面向对象的思想实现了一遍这样的操作.

## LinkedList.h
``` c++
/**
 * 接口文件
 */
#ifndef LINKEDLIST_H
#define LINKEDLIST_H
#include <iostream>
using namespace std;
typedef int ElementType;
namespace mylinkedlist
{
typedef struct Node
{
    struct Node *next;
    ElementType data;
} Node, *NodePtr;
class LinkedList;
//定义这个类
class LinkedList
{
  public:
    //构造函数
    LinkedList();
    //获取链表长度
    int size();
    //打印链表
    void print();
    //尾部增加节点
    void append(ElementType e);
    //头部插入节点
    void push_head(ElementType e);
    //释放整个链表
    void destroy();
    //匹配并删除第一个匹配的节点
    void deleteFirst(ElementType e);
    //匹配并删除所有的匹配的节点
    void deleteAll(ElementType e);
    //反序
    void reverse();
    //排序,默认从小到大排序
    void sort(char rule = '<');
    //在指定位置插入指定节点
    //在第pos个节点之后插入数据域为e的节点
    void insert(int pos, ElementType e);
    //将两个已经按相同规则排序的链表合并并且按指定规则排序
    void merge(LinkedList &, char originalrule, char rule = '<');
  private:
    NodePtr head;
    NodePtr tail;
    int count;
};
}
#endif
```
## LinkedList.cpp
``` c++
/**
 * 实现文件
 */
#include <iostream>
#include "LinkedList.h"
using namespace std;
namespace mylinkedlist
{
//LinkedList构造函数
LinkedList::LinkedList()
{
    head = new Node;
    head->next = NULL; //保护指针，防止指向不明变量
    tail = head;
    count = 0;
}
//获取链表长度
int LinkedList::size()
{
    return count;
}
//尾部插入节点
void LinkedList::append(ElementType e)
{
    //开辟内存
    NodePtr ptemp = new Node;
    if (ptemp == NULL)
    {
        cout << "分配内存失败！" << endl;
        exit(1);
    }
    //填充节点
    ptemp->data = e;
    ptemp->next = NULL;
    //连接
    tail->next = ptemp;
    tail = ptemp;
    //更新count
    count++;
}
//头部插入
void LinkedList::push_head(ElementType e)
{
    NodePtr ptemp = new Node;
    if (ptemp == NULL)
    {
        cout << "分配内存失败！" << endl;
        exit(1);
    }
    ptemp->data = e;
    ptemp->next = head->next;
    head->next = ptemp;
    count++;
}
//打印链表
void LinkedList::print()
{
    NodePtr phead = head;
    phead = phead->next;
    while (phead != NULL)
    {
        cout << phead->data << " ";
        phead = phead->next;
    }
    cout << endl;
}
//匹配并删除第一个匹配的节点
void LinkedList::deleteFirst(ElementType e)
{
    NodePtr phead, ptemp;
    phead = head;
    ptemp = phead;
    phead = phead->next;
    while (phead != NULL)
    {
        if (phead->data == e)
        {
            ptemp->next = phead->next;
            delete phead;
            count--;
            break;
        }
        phead = phead->next;
        ptemp = ptemp->next;
    }
}
//匹配并删除所有匹配的节点
void LinkedList::deleteAll(ElementType e)
{
    NodePtr phead, ptemp;
    phead = head;
    ptemp = phead;
    phead = phead->next;
    while (phead != NULL)
    {
        if (phead->data == e)
        {
            ptemp->next = phead->next;
            delete phead;
            count--;
            phead = ptemp->next;
        }
        else
        {
            phead = phead->next;
            ptemp = ptemp->next;
        }
    }
}
//反序
void LinkedList::reverse()
{
    NodePtr p, q,r;
    NodePtr h=new Node;
    p = NULL;
    q = head->next;
    delete head;
    // head = NULL; //旧的头指针是新的尾指针，next需要指向NULL
    while (q)
    {
        r = q->next; //先保留下一个step要处理的指针
        q->next = p; //然后p q交替工作进行反向
        p = q;
        q = r;
    }
    h->next=p;
    
    // tail = head;
    head = h; // 最后q必然指向NULL，所以返回了p作为新的头指针
}
//排序
void LinkedList::sort(char rule)
{
    NodePtr phead, ptemp;
    if (rule == '<')
    {
        for (phead = head->next; phead != NULL; phead = phead->next)
        {
            for (ptemp = phead->next; ptemp != NULL; ptemp = ptemp->next)
            {
                if (phead->data > ptemp->data)
                {
                    ElementType temp = phead->data;
                    phead->data = ptemp->data;
                    ptemp->data = temp;
                }
            }
        }
    }
    else if (rule == '>')
    {
        for (phead = head->next; phead != NULL; phead = phead->next)
        {
            for (ptemp = phead->next; ptemp != NULL; ptemp = ptemp->next)
            {
                if (phead->data < ptemp->data)
                {
                    ElementType temp = phead->data;
                    phead->data = ptemp->data;
                    ptemp->data = temp;
                }
            }
        }
    }
    else
    {
        cout << "请正确指明排序规则,'<'或'>'!";
        exit(1);
    }
}
//在第pos个节点之后插入数据域为e的节点
void LinkedList::insert(int pos, ElementType e)
{
    //首先要找到第pos个节点
    NodePtr ptemp = head;
    ptemp = ptemp->next;
    for (int i = 0; i < pos - 1; i++)
    {
        ptemp = ptemp->next;
    }
    NodePtr ptemp1 = new Node;
    ptemp1->data = e;
    ptemp1->next = ptemp->next;
    ptemp->next = ptemp1;
}
}
```
## main.cpp
``` c++
#include <iostream>
#include "LinkedList.h"
using namespace mylinkedlist;
int main(void){
    LinkedList l1=LinkedList();
    l1.append(1);
    l1.append(1);
    l1.append(3);
    l1.append(2);
    l1.append(2);
    l1.append(5);
    l1.append(12);
    l1.append(3);
    // l1.insert(2,100);
    l1.print();
    l1.reverse();
    l1.print();
    l1.reverse();
    l1.print();
    l1.sort('<');
    cout<<l1.size()<<endl;
    l1.print();
    l1.reverse();
    l1.print();
    l1.deleteFirst(1);
    cout<<l1.size()<<endl;
    l1.print();
    l1.deleteAll(2);
   
    cout<<l1.size()<<endl;
    l1.print();
    l1.sort('<');
    l1.print();
    return 0;
}
```
## makefile
``` makefile
main.exe:LinkedList.o main.o
	g++ -o main.exe LinkedList.o main.o
LinkedList.o:LinkedList.cpp LinkedList.h
	g++ -c LinkedList.cpp LinkedList.h
main.o:main.cpp
	g++ -c main.cpp
clean:
	rm main.exe main.o LinkedList.o LinkedList.h.gch
```