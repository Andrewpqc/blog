---
title: 递归和非递归解决八皇后问题
comments: true
date: 2017-10-10 13:22:55
updated: 2017-11-16 13:22:55
tags: [Eight Queens]
categories: Algorithm
permalink:
---
# 问题背景
八皇后问题是一个以国际象棋为背景的问题：如何能够在8×8的国际象棋棋盘上放置八个皇后，使得任何一个皇后都无法直接吃掉其他的皇后？为了达到此目的，任两个皇后都不能处于同一条横行、纵行或斜线上。八皇后问题可以推广为更一般的n皇后摆放问题：这时棋盘的大小变为n×n，而皇后个数也变成n。当且仅当n = 1或n ≥ 4时问题有解.
![八皇后示意图](/images/八皇后.png)

八皇后问题最早是由国际象棋棋手马克斯·贝瑟尔（Max Bezzel）于1848年提出。第一个解在1850年由弗朗兹·诺克（Franz Nauck）给出。并且将其推广为更一般的n皇后摆放问题。诺克也是首先将问题推广到更一般的n皇后摆放问题的人之一。

# 递归实现求解八皇后(N皇后)
``` c++
#include <iostream>
using namespace std;
#define MAX 12
int a[MAX]={0};//下标表示行，对应的值表示列
int sum=0;//解的总数
//打印摆放好的棋盘
void show(int *a){
    for(int i=0;i<MAX-1;i++){
        for(int j=0;j<MAX-1;j++){
            if(a[i]==j){
                cout<<"Q ";
            }
            else{
                cout<<"* ";
            }
        }
        cout<<endl;
    }
}
//现在假设第n行以前的都放好了，现在检查第n+1是否可以放置皇后
bool check(int n){
    for (int i=0;i<n;i++){
        if(a[i]==a[n]||abs(a[i]-a[n])==(n-i)){
            return false;
        }
    }
    return true;
}
void put(int n){
    for (int i=0;i<MAX;i++){
        a[n]=i;
        if(check(n)){
            if(n==MAX-1){
                //说明此时全部摆放完成，下面打印排放的结果
                show(a);
                cout<<endl;
                sum+=1;
                cout<<sum<<endl;
            }
            else{
                put(n+1);
            }
        }
}
}
int main(void){
    put(0);
    cout<<"sum:"<<sum<<endl;
    return 0;
}
```
# 非递归实现求解八皇后(N皇后)
``` c++
#include <iostream>
#include <cmath>
using namespace std;
#define MAX 8
#define INIT -1
int q[MAX]={INIT};
int sum=0;
bool check(int row,int col){
    for(int i=0;i<MAX;i++){
        if(q[i]==col||abs(q[i]-col)==abs(row-i)){
            return false;
        }
    }
    return true;
}
void show(int * a){
    for(int i=0;i<MAX-1;i++){
        for(int j=0;j<MAX-1;j++){
            if(a[i]==j){
                cout<<"Q ";
            }
            else{
                cout<<"* ";
            }
        }
        cout<<endl;
    }
}
int main (void){
    int i=0,j=0;
    while(i<MAX){
        while(j<MAX){
            cout<<check(i,j)<<endl;
            if(check(i,j)){
                cout<<"ok"<<endl;
                q[i]=j;
                j=0;
                break;
            }
            else{
                ++j;
            }
        }
        //这个第i行都没有地方放，这时就需要回溯，就说明上面
        //有的行放错了位置
        if(q[i]==INIT){
            if(i==0)
                break;
            else{
                i--;
                j=q[i]+1;
                q[i]=INIT;
                continue;
            }
        }
        //说明此时已经排好一组了
        if(i==MAX-1){
            cout<<sum+1<<endl;
            show(q);
            cout<<endl;
            sum+=1;
            j=q[i]+1;
            q[i]=INIT;
            continue;
        }
        i++;
    }
    cout<<"sum:"<<sum<<endl;
    return 0;
}
```