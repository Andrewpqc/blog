---
title: C++总结1
comments: true
date: 2017-10-24 08:35:14
updated: 2017-11-15 08:35:14
tags: [makefile]
categories: C/C++
permalink:
---
上学期的时候自学过c++,之后就一直在搞其他的高级语言.这学期选了一个面向对象程序设计的课，讲的是c++。但是我发现自己把c++的语法快忘干净了。所以这几天又在看c++的书了。感觉第二次看比第一次看要简单很多。许多东西扫一扫基本上也就知道是怎么回事了。但是温故而知新，我又发现了很多新的知识点和一些以前没遇到过的坑。这些坑绝大部分是来源于开发环境的变化（以前是在windows上的ide中开发的，而现在却是在linux上用一个简单的文本编辑器编辑的)。以下的点全部基于g++编译器和Ｕｂｕｎｔｕ操作系统，其他的编译器或操作系统可能情况不同。

# 定义类
## 类的相互引用

有时，我们在定义多个类的时候可能会遇到两个类的相互引用的情况，比如下面：
``` c++
class A{
	public:
	B func(...);
	...
	private:
	B obj;
	...
};
 class B{
 	public:
	A func(...);
	...
 	private:
 	A obj;
	...
 }；
```
这就出现了这样的尴尬局面：当把A放在B前面的时候，A中引用的B还没有定义，反之亦然。这时我们可以在两个类的定义之前先声明两个类就可以解决这个问题了，向下面这样：
``` c++
//先申明两个类
class A;
class B;
class A{
	public:
	B func(...);
	...
	private:
	B obj;
	...
};
 class B{
 	public:
	A func(...);
	...
 	private:
 	A obj;
	...
 }；
```
## 定义类时引用自身

还有时我们可能会在一个类的定义的时候使用了这个类，比如下面这样：
``` c++
class A{
	public:
	void func(A& );
	private:
	A obj;
};
```
这在g++这个编译器中会报错，显示A没有定义，这个时候的解决办法其实和上面的是一样的，我们可以在前面加上类的声明：
``` c++
//声明类A
calss A;
class A{
	public:
	void func(A& );
	private:
	A obj;
};
```
## 类定义后面要加分号

这个问题也不是什么大问题，但是以前这个问题一直是ide帮我解决的，现在没有ide了，就写一下提醒自己吧！

# 多文件编译与链接

以前有ide，要想检验自己写的对不对，只需要点个按钮就行了，完全不用自己去管多文件之间的依赖关系，编译链接这些东西的，因为ide已经帮你做好了。但是在linux上则不同。现在这些工作都需要自己完成。
## 关于gcc和g++
在正式了解多文件编译与链接之前，先了解一下gcc和g++.

>gcc 最开始的时候是 GNU C Compiler, 如你所知，就是一个c编译器。但是后来因为这个项目里边集成了更多其他不同语言的编译器，GCC就代表 the GNU Compiler Collection，所以表示一堆编译器的合集。 g++则是GCC的c++编译器。现在你在编译代码时调用的gcc，已经不是当初那个c语言编译器了，更确切的说他是一个驱动程序，根据代码的后缀名来判断调用c编译器还是c++编译器 (g++)。比如你的代码后缀是*.c，他会调用c编译器还有linker去链接c的library。如果你的代码后缀是cpp, 他会调用g++编译器，当然librarycall也是>c++版本的。当然我说了这么多你可能感到有些混乱，没关系，你就把gcc当成c语言编译器，g++当成c++语言编译器用就是了。

出处：https://www.zhihu.com/question/20940822/answer/16667772

## 几个g++命令

假设我们有一个test.cpp源文件，我们要把他编译链接成test.exe文件，这会经历下面这些步骤：
１.预处理[预处理器]
``` bash
g++ -E test.cpp > test.i
```
功能：输出预处理后的文件，linux下以.i为后缀名。只激活预处理,这个不生成文件,你需要把它重定向到一个输出文件里 。这一步主要做了这些事情：宏的替换，还有注释的消除，还有找到相关的库文件。用编辑器打开test.i会发现有很多很多代码，你只需要看最后部分就会发现，预处理做了宏的替换，还有注释的消除，可以理解为无关代码的清除。

2.将预处理后的文件不转换成汇编语言,生成以.s结尾的文件[编译器]
``` bash
g++ -S test.cpp
```
功能:会生成test.s文件，.s文件表示是汇编文件，用编辑器打开就都是汇编指令。

3.由汇编指令变为目标代码(机器代码)生成.o的文件[汇编器as]
``` bash
g++ -c test.cpp
```
功能：.o是GCC生成的目标文件，除非你是做编译器和连接器调试开发的，否则打开这种.o没有任何意义。二进制机器码一般人也读不了。

4.连接目标代码,生成可执行程序[链接器ld]
``` bash
g++ test.o -L F:\vs2008\VC\include\iostream
```
功能：将.o文件与所需的库文件链接整合形成.exe文件，这就是可执行文件。-L 表示链接，这里我后面写的是绝对路径，相对各人电脑不同

在上面各个步骤中你可以用-o命令输出你自己想要的各种名字。比如最后一个命令，用下面的输出test.exe
你可以
``` bash
g++ Test.o -o Test.exe -L F:\vs2008\VC\include\iostream
```
上面的四个步骤其实可以合并成一条命令：
``` bash
g++ test.cpp
```
这一条命令包含了上面的四个步骤，在ubuntu16.4上会默认生成名为a.out的可执行文件，这样我们就可以通过./a.out来执行这个可执行文件了。
我们也可以指定生成的可执行文件的文件名，通过-o选项
``` bash
g++ -o test.exe test.cpp
```
## 为什么用多文件

在实际的开发中，为了便于团队的协作和保持代码的可维护性，加强代码的可重用性，以及通过设计抽象数据类型来加强封装，使的将不同功能的模块放在不同的文件中是一种最好的选择。程序的实现往往是通过一个一个组件来实现的，而程序的组件往往是通过一个或几个封装好的类来实现的，为了符合抽象数据类型(ADT)的设计原则,组件的设计者通常都是将类的接口文件(类的声明)，类的实现文件(类的具体功能的实现)分开来写，而使用这个组件的应用程序又是放在另外的一个文件中。这样来看在一个文件中就有了３中类型的文件：接口文件，实现文件，应用程序文件。他们彼此之间通过相互include来建立联系。这样一来，在一个项目中，文件与文件之间就有了非常复杂的依赖关系。所以在编译时先编译谁后编译谁就成了一个问题。
## 用makefile解决问题

make+makefile可以非常方便的控制这种程序的编译链接问题。
先来看下面的一个小项目：
projectName:
student.h
student.cpp
main.cpp
这是一个小小的项目，student.h(接口文件)中定义了一个学生类，代码如下：
``` c++
//防止重复定义
#ifndef STUDENT_H
#define STUDENT_H
#include <iostream>
#include <string>
using namespace std;
//这里先声明一下，因为在这个类的定义中使用了这个类本身
class Student;
//定义这个类
class Student{
public:
    Student(string name,int age,int score);
    Student();
    string getName();
    void setName(string name);
    int getAge();
    void setAge(int age);
    int getScore();
    void setScore(int score);
    friend bool is_score_equal(Student t1,Student t2);
private:
    string name;
    int age;
    int score;
};//注意类的定义的末尾需要有分号
#endif

student.cpp(实现文件)中实现了这个类的功能，代码如下：

#include <iostream>
#include <string>
#include "student.h"
using namespace std;
Student::Student(string name,int age,int score):name(name),age(age),score(score){
    cout<<"初始化成功"<<endl;
}
Student::Student():name("none"),age(18),score(60){
    cout<<"初始化成功"<<endl;
}
string Student::getName(){
    return name;
}
void Student::setName(string _name){
/**
*这里传入的变量名称最好不要与私有变量名相同,如传入name,然后
*赋值写为name=name,这在g++中会报错，下面其他的也一样
*/
    name=_name;
}
int Student::getAge(){
    return age;
}
void Student::setAge(int _age){
    age=_age;
}
int Student::getScore(){
    return score;
}
void Student::setScore(int _score){
    
    score=_score;
}
bool is_score_equal(Student t1,Student t2){
    return t1.score==t2.score;
}
```
main.cpp(应用程序文件)使用了这个类：
``` c++
#include <iostream>
#include "student.h"
using namespace std;
int main (void){
    Student s1("小明",20,96);
    Student s2;
    s2.setScore(96);
    s2.setAge(55);
    bool b=is_score_equal(s1,s2);
    cout<<s2.getAge()<<endl;
    cout<<b<<" "<<s1.getScore()<<" "<<s2.getScore()<<endl;
}
```
现在我要把这个项目编译出来，那么我需要一个名为makefile的文件来指定文件的依赖关系和编译链接逻辑，下面先拿出makefile:
``` makefile
main.exe:main.o student.o 
	g++ -o main.exe student.o main.o
student.o:student.cpp student.h
	g++ -c student.cpp
main.o:main.cpp
	g++ -c main.cpp
clean:
	rm main.exe main.o student.o
```
把这个makefile文件放在项目根目录，这时只有我们在根目录执行make命令：
``` bash
$ make
```
执行完后就可以在项目目录中看到main.o,student.o两个目标文件，和main.exe这个可执行文件，这个main.exe就是我们最终需要的那个文件了。这时我们再执行./main.exe就可以执行我们的程序了。
``` bash
$ ./main.exe
```
同时，如果我们想要清除中间生成的目标文件和最终的可执行文件，从而进行重新编译链接，我们可以使用下面的命令：
```
$ make clean
```
## makefile的语法

更多关于makefile的内容请参考[这里](http://wiki.ubuntu.org.cn/%E8%B7%9F%E6%88%91%E4%B8%80%E8%B5%B7%E5%86%99Makefile:MakeFile%E4%BB%8B%E7%BB%8D#makefile.E7.9A.84.E8.A7.84.E5.88.99)