---
title: 动态数组
comments: true
date: 2017-09-26 12:59:58
updated: 2017-11-16 12:59:58
tags: [Dynamic array]
categories: Algorithm
permalink:
---
最近在学数据结构，涉及到动态数组。我自己总结了一下，把与动态数组有关的基本操作用c和c++分别写了一遍。下面把总结的成果记录一下。

动态数组之所以称之为动态，关键就在于其可以在程序运行过程中自动增加分配的内存大小。要实现这一点的关键又在于下面两点：

- 实时记录数组的元素个数和分配的内存的大小
- realloc()函数重新分配内存

数组中的数据是连续存放的，可以随机访问。在数组尾部增加删除元素效率很高，但是在中间插入和删除元素则效率低下。
# C语言动态数组操作
要实现实时记录数组的元素个数和分配的内存的大小可以使用下面的一个结构体：
``` c++
typedef int ElementType;
typedef struct DynamicArray{
    ElementType* elems;
    int size;
    int capacity;
} *DynamicArrayPtr;
```
elems是数据所在地址块的首地址。size和capacity分别是所存储数据的个数和当前分配的容量，他们的值必须与当前数组的状况相对应。

## 初始化数组
``` c++
void init(DynamicArrayPtr DAP){
    DAP->elems=(ElementType*)malloc(sizeof(ElementType)*100);
    DAP->size=0;
    DAP->capacity=100;
}
```
初始化的过程中我们为数据存储区域分配了初始的可以容纳100个元素的内存空间，所以当前的容量为100。此时数组中还没有数据，所以大小为0.

## 打印动态数组
``` c++
void print(DynamicArrayPtr DAP) {
	for (int i = 0; i < DAP->size; i++) {
		printf("%d  ", DAP->elems[i]);
    }
    printf("\n");
}
```
## 内存重新分配
``` c++
void dealOverflow(DynamicArrayPtr DAP){
    if (DAP->size >= DAP->capacity) {
        ElementType* newbase = (ElementType*)realloc(DAP->elems, (50 + DAP->size)*sizeof(ElementType));
        if(newbase==NULL){
            printf("重新分配内存失败！");
            return ;
        }
        DAP->elems=newbase;
		DAP->capacity += DELTA_CAPACITY;
    }
}
```
内存重新分配的条件就是当前数组中的元素个数大于或等于当前分配的容量时.重新分配使用的函数是realloc(),在原来的容量的基础之上新分配了可以容纳20个元素的空间,分配完成之后需要将记录当前容量大小的变量更新一下。这个函数主要进行的是容量是否够用的检测，如果不够用则增加容量。在每增加一个元素之前就应该调用一次该函数。

## 尾部追加元素
``` c++
void append(DynamicArrayPtr DAP,ElementType e){
    dealOverflow(DAP);
    DAP->elems[DAP->size]=e;
    DAP->size++;
}
```
向尾部追加元素效率是比较高的。在追加之前首先要检测容量够不够用，追加之后要更新记录数组大小的变量的值。

## 插入元素
``` c++
void insert(DynamicArrayPtr DAP,int pos,ElementType e){
    //若容量不够则重新分配内存
    dealOverflow(DAP);
    //下标超界的处理
    if(pos>DAP->size||pos<0){
        exit(1);
        return;
    }
    
    int i;
    //移动元素
    for(i=DAP->size;i>pos;i--){
        DAP->elems[i]=DAP->elems[i-1];
    }
    //插入元素
    DAP->elems[i]=e;
    //更新sie
    DAP->size++;
}
```
插入元素效率较低，在所插入位置的元素之后的所有元素都必须向后移动一个位置。

## 删除元素
``` c++
void myremove(DynamicArrayPtr DAP,int pos){
    //下标超界的处理
    if(pos>=DAP->size||pos<0){
        exit(1);
        return;
    }
     //删除数据
     int i;
     for(i=pos;i<DAP->size;i++){
         DAP->elems[i]=DAP->elems[i+1];
     }
     更新size
     DAP->size--;
}
```
## 元素排序
``` c++
void sort(DynamicArrayPtr DAP,char rule='<'){
        if(rule=='<'){//从小到大排序
            for(int i=0;i<DAP->size;i++){
                for(int j=i+1;j<DAP->size;j++){
                    if(DAP->elems[i]>DAP->elems[j]){
                        int temp=DAP->elems[i];
                        DAP->elems[i]=DAP->elems[j];
                        DAP->elems[j]=temp;
                    }
                }
            }
        }
        else if(rule=='>'){//从大到小排序
            for(int i=0;i<DAP->size;i++){
                for(int j=i+1;j<DAP->size;j++){
                    if(DAP->elems[i]<DAP->elems[j]){
                        int temp=DAP->elems[i];
                        DAP->elems[i]=DAP->elems[j];
                        DAP->elems[j]=temp;
                    }
                }
            }
        }
        else{//rule参数输入错误
            printf("请输入正确的排序规则('<' or '>')!!!");
            exit(1);
        }
}
```
上面使用冒泡法则对元素进行排序，rule参数指定排序规则，默认按照从小到大排序。

## 合并两个数组并排序
``` c++
void mergeAndsort(DynamicArrayPtr DAP1,DynamicArrayPtr DAP2,char rule='<'){
    for(int i=0;i<DAP2->size;i++){
        append(DAP1,DAP2->elems[i]);
    }
    //排序
    sort(DAP1,rule);
 }
```
## 两个有序数组合并排序
``` c++
/**
 * 功能:对两个已经按照相同规则排好序的动态数组合并并按照指定规则排序排序，
 * 根据原始规则与指定的规则的不同有四种排列组合
 * ＠para:DA１，DA2是两个原始的已有序的动态数组
 * ＠para:DA3是最终生成的结果动态数组
 * ＠para:originrule是原始的已排序的动态数组的排序规则('<' or '>')
 * ＠para:rule是合并后的的动态数组的排序规则('<' or '>')
 */ 
void mergeAndsort_for_sorted(DynamicArrayPtr DA1,DynamicArrayPtr DA2,DynamicArrayPtr DA3,char originrule,char rule='<'){
    if(originrule=='<'&&rule=='<'){
            int i=0,j=0;
            while(i!=DA1->size&&j!=DA2->size){
                if(DA1->elems[i]<DA2->elems[j]){
                    append(DA3,DA1->elems[i]);
                    i++;
                }else if(DA1->elems[i]==DA2->elems[j]) {
                    //防止循环的最后一个相同的时候漏掉一个
                    append(DA3,DA1->elems[i]);
                    i++;
                    append(DA3,DA2->elems[j]);
                    j++;
                }else{
                    append(DA3,DA2->elems[j]);
                    j++;
                }
            }
            if(DA1->size==i){
                while(j<DA2->size){
                   append(DA3,DA2->elems[j]);
                    j++;
                }
                }
            else if(DA2->size==j){
                while(i<DA1->size){
                   append(DA3,DA1->elems[i]);
                    i++;
                }
                }
    }
    else if(originrule=='>'&&rule=='>'){
       int i=0,j=0;
       while(i!=DA1->size&&j!=DA2->size){
           if(DA1->elems[i]<DA2->elems[j]){
               append(DA3,DA2->elems[j]);
               j++;
           }else if(DA1->elems[i]==DA2->elems[j]) {
               //防止循环的最后一个相同的时候漏掉一个
               append(DA3,DA1->elems[i]);
               i++;
               append(DA3,DA2->elems[j]);
               j++;
           }else{
               append(DA3,DA1->elems[i]);
               i++;
           }
       }
       if(DA1->size==i){
           while(j<DA2->size){
              append(DA3,DA2->elems[j]);
               j++;
           }
           }
       else if(DA2->size==j){
           while(i<DA1->size){
              append(DA3,DA1->elems[i]);
               i++;
           }
           }
    }
    else if(originrule=='<'&&rule=='>'){
       int i=DA1->size-1,j=DA2->size-1;
       while(i!=-1&&j!=-1){
           if(DA1->elems[i]<DA2->elems[j]){
               append(DA3,DA2->elems[j]);
               j--;
           }else if(DA1->elems[i]==DA2->elems[j]) {
               //防止循环的最后一个相同的时候漏掉一个
               append(DA3,DA1->elems[i]);
               i--;
               append(DA3,DA2->elems[j]);
               j--;
           }else{
               append(DA3,DA1->elems[i]);
               i--;
           }
       }
       if(-1==i){
           while(j!=-1){
              append(DA3,DA2->elems[j]);
               j--;
           }
           }
       else if(-1==j){
           while(i!=-1){
              append(DA3,DA1->elems[i]);
               i--;
           }
           }
    }
    else if(originrule=='>'&&rule=='<'){
       int i=DA1->size-1,j=DA2->size-1;
       while(i!=-1&&j!=-1){
           if(DA1->elems[i]<DA2->elems[j]){
               append(DA3,DA1->elems[i]);
               i--;
           }else if(DA1->elems[i]==DA2->elems[j]) {
               //防止循环的最后一个相同的时候漏掉一个
               append(DA3,DA1->elems[i]);
               i--;
               append(DA3,DA2->elems[j]);
               j--;
           }else{
               append(DA3,DA2->elems[j]);
               j--;
           }
       }
       if(-1==i){
           while(j!=-1){
              append(DA3,DA2->elems[j]);
               j--;
           }
           }
       else if(-1==j){
           while(i!=-1){
              append(DA3,DA1->elems[i]);
               i--;
           }
           }
    }
    else{
        printf("原始规则或指定的规则不正确");
        exit(1);
    }
}
``` 
# C++版动态数组
我用c++把上面的c的内容封装了一下，下面的是全部的内容。

## DynamicArray.h的内容
``` c++
/**
 * 接口文件
 */ 
#ifndef DYNAMICARRAY_H
#define DYNAMICARRAY_H
#include <iostream>
#define INIT_CAPACITY 100
#define DELTA_CAPACITY 100
using namespace std;
typedef int ElementType;
namespace mydynamicarray{
    class DynamicArray;
    //定义这个类
    class DynamicArray{
    public:
        //构造函数
        DynamicArray();
        //获取长度
        int getSize();
        //获取容量
        int getCapacity();
        //最大值的索引
        int max();
        //最小值的索引
        int min();
        //打印输出
        void print();
        //在末尾追加数据
        void append(ElementType e);
        //在指定下标的前面插入指定的数据
        void insert(int pos,ElementType e);
        //移除指定下标的数据
        void remove(int pos);
        //销毁
        void todestroy();
        //按照指定规则排序，默认从小到大
        void sort(charneirong rule='<');
        //合并两个动态数组，并且按照规则排序
        void mergeAndsort(DynamicArray&,char rule);
        //对两个已经具有相同顺序的动态数组进行合并并且排序
        void mergeAndsort_for_sorted(DynamicArray&,DynamicArray&,char originrule,char rule='<');
    
    private:
        void dealOverflow();//重新分配内存
        ElementType* elems;
        int size;
        int capacity;
    };//注意类的定义的末尾需要有分号
}
#endif
```
## DynamicArray.cpp的内容
``` c++
/**
 * 实现文件
 */ 
#include <iostream>
#include <cstdlib>
#include "DynamicArray.h"
using namespace std;
namespace mydynamicarray{
    //构造函数
    DynamicArray::DynamicArray(){
        elems=new ElementType[INIT_CAPACITY];
        size=0;
        capacity=INIT_CAPACITY;
    }
   
    //获取长度
    int DynamicArray::getSize(){
        return size;
    }
    //获取容量
    int DynamicArray::getCapacity(){
        return capacity;
    }
    //最大值
    int DynamicArray::max(){
        int result=0;
        for(int i=0;i<size-1;i++){
            result=(elems[i]>elems[i+1])?i:i+1;
        }
        return result;
    }
    //最小值
    int DynamicArray::min(){
        int result=0;
        for(int i=0;i<size-1;i++){
            result=(elems[i]<elems[i+1])?i:i+1;
        }
        return result;
    }
    //打印
    void DynamicArray::print(){
        for(int i=0;i<size;i++){
            cout<<elems[i]<<" ";
        }
        cout<<endl;
    }
    //处理重新分配问题
    void DynamicArray::dealOverflow(){
        if(size>=capacity){
            cout<<">>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>"<<endl;
            ElementType* newbase=(ElementType*)realloc(elems,size+DELTA_CAPACITY);
            if(newbase==NULL){
                cout<<"重新分配内存失败！"<<endl;
                return;
            }
            elems=newbase;
            capacity+=DELTA_CAPACITY;
        }
    }
    //尾部追加
    void DynamicArray::append(ElementType e){
        dealOverflow();
        elems[size]=e;
        size++;
    }
    //在指定下表位置插入
    void DynamicArray::insert(int pos,ElementType e){
        dealOverflow();
        if(pos<0||pos>size){
            cout<<"指定的位置不合法！"<<endl;
            return;
        }
        int i;
        for(i=size;i>pos;i--){
            elems[i]=elems[i-1];
        }
        elems[i]=e;
        size++;
    }
    //删除指定下表的元素
    void DynamicArray::remove(int pos){
        if(pos<0||pos>size){
            cout<<"指定的位置不合法！"<<endl;
            return;
        }
        int i;
        for(i=pos;i<size;i++){
            elems[i]=elems[i+1];
        }
        size--;
    }
    //销毁
    void DynamicArray::todestroy(){
        delete [] elems;
        size=0;
        capacity=0;
    }
    void DynamicArray::sort(char rule){
        if(rule=='<'){//从小到大排序
            for(int i=0;i<size;i++){
                for(int j=i+1;j<size;j++){
                    if(elems[i]>elems[j]){
                        int temp=elems[i];
                        elems[i]=elems[j];
                        elems[j]=temp;
                    }
                }
            }
        }
        else if(rule=='>'){//从大到小排序
            for(int i=0;i<size;i++){
                for(int j=i+1;j<size;j++){
                    if(elems[i]<elems[j]){
                        int temp=elems[i];
                        elems[i]=elems[j];
                        elems[j]=temp;
                    }
                }
            }
        }
        else{//rule参数输入错误
            cout<<"请输入正确的排序规则('<' or '>')!!!"<<endl;
            return;
        }
    }
    void DynamicArray::mergeAndsort(DynamicArray& dyArray,char rule='<'){
        for(int i=0;i<dyArray.size;i++){
            append(dyArray.elems[i]);
        }
        //销毁DA2,注意不能够两次释放，如果这里释放了，那么在其他的地方
        //不能再次释放
        // dyArray.todestroy();
        //排序
        sort(rule);
    }
    //对两个已经有序的动态数组进行合并排序
    void DynamicArray::mergeAndsort_for_sorted(DynamicArray& DA2,DynamicArray& DA3,char originrule,char rule){
        if(originrule=='<'&&rule=='<'){
                int i=0,j=0;
                while(i!=size&&j!=DA2.size){
                    if(elems[i]<DA2.elems[j]){
                        DA3.append(elems[i]);
                        i++;
                    }else if(elems[i]==DA2.elems[j]) {
                        //防止循环的最后一个相同的时候漏掉一个
                        DA3.append(elems[i]);
                        i++;
                        DA3.append(DA2.elems[j]);
                        j++;
                    }else{
                        DA3.append(DA2.elems[j]);
                        j++;
                    }
                }
                if(size==i){
                    while(j<DA2.size){
                       DA3.append(DA2.elems[j]);
                        j++;
                    }
                    }
                else if(DA2.size==j){
                    while(i<size){
                       DA3.append(elems[i]);
                        i++;
                    }
                    }
        }
        else if(originrule=='>'&&rule=='>'){
           int i=0,j=0;
           while(i!=size&&j!=DA2.size){
               if(elems[i]<DA2.elems[j]){
                   DA3.append(DA2.elems[j]);
                   j++;
               }else if(elems[i]==DA2.elems[j]) {
                   //防止循环的最后一个相同的时候漏掉一个
                   DA3.append(elems[i]);
                   i++;
                   DA3.append(DA2.elems[j]);
                   j++;
               }else{
                   DA3.append(elems[i]);
                   i++;
               }
           }
           if(size==i){
               while(j<DA2.size){
                  DA3.append(DA2.elems[j]);
                   j++;
               }
               }
           else if(DA2.size==j){
               while(i<size){
                  DA3.append(elems[i]);
                   i++;
               }
               }
        }
        else if(originrule=='<'&&rule=='>'){
           int i=size-1,j=DA2.size-1;
           while(i!=-1&&j!=-1){
               if(elems[i]<DA2.elems[j]){
                   DA3.append(DA2.elems[j]);
                   j--;
               }else if(elems[i]==DA2.elems[j]) {
                   //防止循环的最后一个相同的时候漏掉一个
                   DA3.append(elems[i]);
                   i--;
                   DA3.append(DA2.elems[j]);
                   j--;
               }else{
                   DA3.append(elems[i]);
                   i--;
               }
           }
           if(-1==i){
               while(j!=-1){
                  DA3.append(DA2.elems[j]);
                   j--;
               }
               }
           else if(-1==j){
               while(i!=-1){
                  DA3.append(elems[i]);
                   i--;
               }
               }
        }
        else if(originrule=='>'&&rule=='<'){
           int i=size-1,j=DA2.size-1;
           while(i!=-1&&j!=-1){
               if(elems[i]<DA2.elems[j]){
                   DA3.append(elems[i]);
                   i--;
               }else if(elems[i]==DA2.elems[j]) {
                   //防止循环的最后一个相同的时候漏掉一个
                   DA3.append(elems[i]);
                   i--;
                   DA3.append(DA2.elems[j]);
                   j--;
               }else{
                   DA3.append(DA2.elems[j]);
                   j--;
               }
           }
           if(-1==i){
               while(j!=-1){
                  DA3.append(DA2.elems[j]);
                   j--;
               }
               }
           else if(-1==j){
               while(i!=-1){
                  DA3.append(elems[i]);
                   i--;
               }
               }
        }
        else{
            printf("原始规则或指定的规则不正确");
            exit(1);
        }
    }
}
```
## main.cpp(测试)的内容
``` c++
/**
 * 测试文件
 */ 
#include <iostream>
#include "DynamicArray.h"
using namespace std;
using namespace mydynamicarray;
int main() {
   DynamicArray array1=DynamicArray();
   array1.append(1);
   array1.append(3);
   array1.append(2);
   array1.print();
   array1.insert(0,100);
   array1.insert(2,200);
   array1.print();
   for(int i=0;i<10;i++){
       array1.append(i);
   }
   array1.print();
   array1.sort('>');
   array1.print();
   array1.sort();
   array1.print();
   DynamicArray array2=DynamicArray();
   array2.append(1000);
   array2.append(3000);
   array2.append(2000);
   array2.append(250);
   array2.append(9000);
   array2.sort();
   cout<<array2.getSize()<<"||||||||||||"<<array2.getCapacity()<<endl;
   array2.print();
   array1.mergeAndsort(array2,'<');
   array1.print();
   DynamicArray array3=DynamicArray();
   array1.mergeAndsort_for_sorted(array2,array3,'<');
   array3.print();
   cout<<array3.max()<<endl;
   cout<<array3.min()<<endl;
   array1.todestroy();
   array2.todestroy();
   array3.todestroy();
   return 0;
}
```
## makefile
``` c++
main.exe:DynamicArray.o main.o
    g++ -o main.exe DynamicArray.o main.o
DynamicArray.o:DynamicArray.cpp DynamicArray.h
    g++ -c DynamicArray.cpp DynamicArray.h
main.o:main.cpp
    g++ -c main.cpp
clean:
    rm main.exe main.o DynamicArray.o DynamicArray.h.gch
```
