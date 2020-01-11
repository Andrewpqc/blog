---
title: Vector in STL
comments: true
date: 2017-09-26 09:03:39
updated: 2017-11-15 09:03:39
tags: [STL Vector]
categories: C/C++
permalink:
---
在c++的标准模板库中有许多模板容器，他们封装了底层的数据结构，并且可以用来装几乎所有的已有数据类型以及用户自定义的数据类型。vector(向量)就是其中一个非常好用的序列式的容器。vector是最简单的序列容器，支持元素的随机访问，当然有时候显得效率低一些。
# vector类基础
## vector对象定义

vector对象定义之前需要使用std命名空间和vector这一个头文件。
``` c++
#include <string>
#include <vector>
using namespace std;
int main (){
	vector<int> myIntVec;
	vector<string> myStrVec;
	return 0;
}
```
上面就是定义了两个vector对象，一个是用来装int类型的数据，一个是用来装string类型的数据。
## vector类对象初始化

vector对象的常用的初始化方法有以下几种：
``` c++
vector<T> v1          //v1是一个空vector，它潜在的元素是T类型的，执行默认初始化
vector<T> v2(v1)       //v2中包含有v1所有元素的副本
vector<T> v2 = v1       //等价于v2(v1)，v2中包含有v1所有元素的副本
vector<T> v3(n, val)      //v3包含了n个重复的元素，每个元素的值都是val
vector<T> v4(n)          //v4包含了n个重复地执行了值初始化的对象
vector<T> v5{a,b,c...}　//v5包含了初始值个数的元素，每个元素被赋予相应的初始值
vector<T> v5={a,b,c...}　//等价于v5{a,b,c...}
```
vector对象在没有初始化之前不能够用赋值符号对其进行赋值，但是可以用push_back()成员函数对其赋值例如：
``` c++
vector<int> myIntVec;
myIntVec[0]=0;  //错误
myIntVec.push_back(1); //正确
```
## 容器的大小和容量

vector类定义了size()和capacity()两个函数来实现对当前容器元素的个数统计和对容器当前分配的内存数量的统计，同时也提供了reserve()和resize()两个成员函数来预先设置容器的大小和修改容器的大小。
## vector类的成员函数
``` c++
vector<int> myvec;
myvec.push_back(1);　//向向量中推进去一个数
myvec.pop_back();　　//弹出（删除）向量末尾的元素
myvec.empty() //判断向量是否为空
myvec.front()  //获得向量中的第一个元素
myvec.back()  //获得向量中的最后一个元素
myvec.clear() //将容器清空
myvec.at()   //访问指定的下标的元素
```
## 遍历vector容器

遍历vector容器可以使用迭代器方式，也可以通过使用at()成员函数和循环语句实现：
``` c++
//迭代器加for循环来遍历vector对象
void for_iter(vector<TYPE>& myvec){
	vector<TYPE>::iterator iter;
	for(iter=myvec.begin();iter!=myvec.end();iter++){
	temp=*iter;
	//do somnthing with temp here...
}
}
//for循环加size()和at()成员函数来遍历
void for_at_size(vector<TYPE>& myvec){
	for(int i=0;i<myvec.size();i++){
	temp=myvec.at(i);
	//do something with temp here...
	}
}
```
## 使用`count_if`算法和`for_each`算法操作vector对象

`for_each`和`count_if`算法在algorithm头文件中
``` c++
#include <iostream>
#include <algorithm>
using namespace std;
void print(int& x){
	cout<<x<<endl;
}
bool if_bigger_than_3(int& x){
	return x>3
}
int main(void){
	//定义并初始化含有是个元素的vector对象
	vector<int> myvec={1,2,3,4,5,6,7,8,9,10};
	//for_each算法将指定区间的元素的引用逐一传递到print函数中
	//被print函数中的形参x接收并执行
	for_each(myvec.begin(),myvec.end(),print);
	
	//count_if算法将指定区间的元素的引用逐一传递到if_bigger_than_3函数中
	//返回值为if_bigger_than_3中返回true的次数
	auto count=count_if(myvec.begin(),myvec.end(),if_bigger_than_3);
	
	cout<<count<<endl;
	return 0;
}
```
# vector高级主题
## 元素访问方法

可以直接访问vector对象的操作方法主要包括以下４中：
``` c++
vector<MyType> myvector;
myvector[index];
myvector.at(index);
myvector.front();
myvector.back();
```
myvector[index]和myvector.at(index)两种访问方式返回的都是对象的引用类型，所以既可以取其中的元素，也可以对其赋值。前提是下标值必须有效。front()和back()分别返回第一个元素和最后一个元素。
## 迭代器相关函数
``` c++
vector<int> myvect={1,2,3,4,5};
//返回值类型均为迭代器类型，分别指向第一个元素和最后一个元素的下一个元素
myvect.begin();
myvect.end();
//返回值类型均为逆向迭代器类型，分别指向逆向迭代的第一个元素和逆向迭代的最后一个元素的下一个元素
myvect.rbegin();
myvect.rend();
```
## 元素查找和搜索

STL通用算法find()和find_if()

## 容器中元素的排序

sort算法：
``` c++
#include <iostream>
#include <vector>
#include <string>
#include <algorithm>
using namespace std;
typedef struct{
	int score;
	string name;
}Student;
bool sort_by_score(Student& s1,Student& s2){
	return s1.score>s2.score;
}
void print(Student & s){
	cout<<s.name<<":"<<s.score<<endl;
}
int main(void){
	vector<Student> myvec={{25,"mary"},{35,"jack"},{10,"Lily"}};
	cout<<"排序前："<<endl;
	for_each(myvec.begin(),myvec.end(),print);
	sort(myvec.begin(),myvec.end(),sort_by_score);
	cout<<"排序后："<<endl;
	for_each(myvec.begin(),myvec.end(),print);
	return 0;
}
```
上面程序的输出结果为：
``` 
排序前：
mary:25
jack:35
Lily:10
排序后：
jack:35
mary:25
Lily:10
```
## 插入元素

push_back()可以在vector对象的末尾插入元素，insert()函数可以在vector的任意位置插入元素。下面看演示：
``` c++
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;
void print(int& x){
    cout<<x<<" ";
}
int main(void){
    vector<int>myIntvec={1,2,3,4,5,6,7};
    for_each(myIntvec.begin(),myIntvec.end(),print);
    cout<<endl;
    //在开始位置前面插入０
    myIntvec.insert(myIntvec.begin(),0);
    for_each(myIntvec.begin(),myIntvec.end(),print);
    cout<<endl;
     //在末尾位置前面插入０
     myIntvec.insert(myIntvec.end(),0);
     for_each(myIntvec.begin(),myIntvec.end(),print);
     cout<<endl;
     //创建一个新向量
     vector<int>myIntvec2;
     //用push_back()依次向末尾添加数据
     myIntvec2.push_back(6);
     myIntvec2.push_back(5);
     myIntvec2.push_back(2);
     for_each(myIntvec2.begin(),myIntvec2.end(),print);
     cout<<endl;
     //用pop_back()依次从末尾移除数据
     myIntvec2.pop_back();
     for_each(myIntvec2.begin(),myIntvec2.end(),print);
     cout<<endl;
     //将myIntvec2中的全部元素插入到myIntvec的开始元素的前面
     myIntvec.insert(myIntvec.begin(),myIntvec2.begin(),myIntvec2.end());
     for_each(myIntvec.begin(),myIntvec.end(),print);
     cout<<endl;
     //插入多个相同的值,这里是在myIntvec的开始元素的前面插入两个５
     myIntvec.insert(myIntvec.begin(),2,5);
     for_each(myIntvec.begin(),myIntvec.end(),print);
     cout<<endl;
    return 0;
}
```
输出结果：
```
1 2 3 4 5 6 7
0 1 2 3 4 5 6 7
0 1 2 3 4 5 6 7 0
6 5 2
6 5
6 5 0 1 2 3 4 5 6 7 0
5 5 6 5 0 1 2 3 4 5 6 7 0
```
## 删除元素

在vector删除元素可以使用３个成员函数：pop_back(),erase(),和clear().还可以使用算法库的算法remove()来实现。
``` c++
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;
void print(int& x){
    cout<<x<<" ";
}
int main(void){
    vector<int>myIntvec={1,2,3,4,5,6,7};
    for_each(myIntvec.begin(),myIntvec.end(),print);
    cout<<endl;
    myIntVec.erase(myIntVec.begin());
    for_each(myIntvec.begin(),myIntvec.end(),print);
    cout<<endl;
    myIntVec.clear();
    for_each(myIntvec.begin(),myIntvec.end(),print);
    cout<<endl;
    return 0;
}
```
输出：
``` 
1 2 3 4 5 6 7
2 3 4 5 6 7
```
