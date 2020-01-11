---
title: C++总结2
comments: true
date: 2017-09-10 08:46:48
updated: 2017-11-15 08:46:48
tags:
categories: C/C++
permalink:
---
这一篇文章打算把c++里面非常重要而又不容易记住的东西好好总结一下，主要涉及的知识点有独立编译，命名空间，友元，继承的细节，多态等。
# 独立编译和命名空间

其实在上一篇文章中,我已经讨论过独立编译的问题，只不过没有讲的深入，在这里我打算详细的讲一讲独立编译。
## 抽象数据类型(ADT)

抽象数据类型(Abstract Data Type)是一个类，它将类的接口与类的实现区分开来。对于标准的开发来讲，所有的类都应该设计成ADT。为了定义一个ADT类，需要将类的使用规范与类的实现细节彻底分开。这一点的实现需要遵循一下三条规则：

- 使所有的成员变量都成为类的私有成员。
- 使类的每一项基本操作成为类的一个公共成员函数,一个友元函数，一个普通函数或者一个重载的操作符。将类的定义与函数/操作符的声明以及用来告诉类的使用者该如何使用这个类及相关函数的必要的注释统统放在一个文件中，这个文件就是这个ADT类的接口文件。
- 确保使用ADT的程序员无法访问到基本操作的具体实现。实现由函数定义以及重载的操作符的定义（另外还包括任何辅助函数，或者这些定义需要的其他项）构成。

我们在设计类的时候只要按照ADT的设计原则，那么我们的类就是标准的类了。但是这就引出来了其他的一些问题，比如：编译时的依赖关系，接口文件的重复定义，命令空间问题。
## 解决编译时的依赖关系
这个问题在上一篇文章中已经讨论过了,这里不在赘述。
## 防止接口文件的重复编译
我们设计ADT类的其中一个目的就是要提高代码的可重用性。所有的文件都可以通过include某个ADT类的接口文件，来使用这个类提供的功能。这样的话就有可能出现类似这样的情形：我们项目中的一个文件include另外的两个文件，而另外的两个文件又分别include了同一个ＡＤＴ类，这样的话我们的项目就会编译两次这个ADT类，这是不被允许的。我们必须要解决这个问题。解决的办法就是在类的接口文件中，加三条预编译指令。
``` c++
#ifndef　XXX
#define XXX
//这中间放类的接口文件
#endif
```

将类的接口放在这三条命令之间，就可以防止重复编译。第一次编译的时候，会定义XXX这个宏，当第二次编译的时候，XXX这个宏已经被定义，编译器就会跳过这个接口文件，从而避免了重复编译。
## 命名空间问题。

大型项目的编码工作是由许多的程序员的合作完成的。每个人写的代码最后汇合到一起，这就难以避免的会产生命名冲突。要解决这个问题就要靠命名空间了。大家在编写c++代码的时候，用的最多的就是std命令空间了。其实我们写的每一句代码都是在某一个命名空间中，加入不明确指定一个命名空间，代码就默认放在全局命名空间中。全局命令空间不需要using指令，因为它是默认的命名空间。可以同时使用多个命名空间，比如我们总是在使用全局命名空间，同时也会使用std命名空间。假如一个名称在两个命名空间中都进行了定义，而你又使用了这两个命名空间，那么你使用这个名称的时候就会产生冲突，这个名称是哪个命名空间的呢？所以我们在使用一个名称的时候一般都要指定这个名称属于那个命名空间。
为了防止我们自己定义的类中的名称与全局命名空间或用户引用的其他命名空间中的名称产生冲突，我们最好把自己的类放在一个自己的命名空间中。

### 创建和使用命名空间

要将代码放到一个命名空间中，需要采取一下形式来定义一个命名空间分组：
``` c++
namespace my_namespace_name{
	some_code
}
```

将上述命名空间分组放到自己的代码中，就相当与将some_code中定义的名称放到命名空间my_namespace_name中，为了使用这些名称，需要使用一下的using 指令：
``` c++
using namespace my_namespace_name;
```
下面给一点代码来演示命名空间的创建与使用：
``` c++
#include <iostream>
//程序中需要用到cout,所以需要std命名空间
using namespace std;
//在两个命名空间中声明两个函数
namespace ns1{
    void func1(void);
}
namespace ns2{
    void func2(void);
}
int main(void){
    //要指定使用那个命名空间，才能使用那个命名空间中定义的名称
    {
        using namespace ns1;
        func1();
    }
    {
        using namespace ns2;
        func2();
    }
    return 0;
}
//在同样的命名空间中定义这两个函数
namespace ns1{
    void func1(){
        cout<<"here is func1"<<endl;
    }
}
namespace ns2{
    void func2(){
        cout<<"here is func2"<<endl;
    }
}
```
### 限定名称

假如现在你遇到了这样一种情况，你需要使用ns1命名空间中的func1函数和ns2中的func2函数，但是在ns1和ns2中又分别定义了一个名为func的同名函数。这时如果你使用using namespace ns1和using namespace ns2就不合适了。因为这会使得ns1和ns2中的所有的名称都进入可用状态，这就产生了冲突。一种更为保险的方法就是下面这样：
``` c++
using ns1::func1;
using ns2::func2;
```
这样的话，就只会让ns1命名空间中的func1和ns2中的func2进入可用状态，而两个命名空间中的其他名称仍然不可使用。
# 类中的注意点
## this指针

在 C++ 中，每一个对象都能通过this指针访问自己的地址。this指针是所有成员函数的隐含参数。因此，在成员函数内部，它可以用来指向调用对象。友元函数没有 this 指针，因为友元不是类的成员。只有成员函数才有 this 指针。
## 友元

面向对象的设计思想提高了数据封装的程度和程序数据的安全性。外界对对象内部数据的访问得到了严格的控制。外部要想访问对象的私有属性，必须要通过对象提供的方法去访问。但是有时为了简化函数定义和提高效率，我们想让自己定义的外部函数直接访问到对象的私有属性。这在c++中是通过友元函数实现的。就是将某个外部函数定义为某个类的友元，这样这个函数就得到了类的信任，从而可以直接访问类的私有属性。具体实现，看代码：
``` c++
# include<iostream>
# include <string>
using namespace std;
class Student;
class Student{
    public:
        Student(string _name,int _age);
        Student();
        string getName();
        void setName();
        int getAge();
        void setAge();
        //下面这个函数就是Studnet类的一个友元函数
        friend bool isSameName(Student s1,Student s2);
    private:
        string name;
        int age;
};
Student::Studnet(strng _name,int _age):name(_name),age(_age){
    //有意留空
}
Student::Student():name("none"),age(18){
    //有意留空
}
string Studnet::getName(){
    return name;
}
void Student::setName(string _name){
    name=_name;
}
int Student::getAge(){
    return age;
}
void Student::setAge(int _age){
    age=_age;
}
bool isSameName(Studnet s1,Student s2){
    return s1.name==s2.name;
}
```
上面的代码包含了一个Student类的接口和实现，由于这里只是演示，所以就没有考虑上文中提到的那些东西。上面定义了一个友元函数，用来判断两个学生是否同名。必须明确的是，友元函数并不是一个类的成员函数，他是一个外部函数，但是把他声明为某个类的友元之后，他就有了访问类的私有属性的权限。声明的方法是在当事类中声明，并且以friend关键字打头，定义的时候还是按照普通函数那样定义。假如我们的类有良好的取值和赋值函数的话，我们也可以不必把这个函数定义成友元函数，直接就把他定义成一个普通的外部函数，这时，他的定义就变成这样了：
``` c++
bool isSameName(Student s1,Student s2){
	return s1.getName()==s2.getName();
}
```
可见这样的话在一定程度上增加了函数定义的复杂度，并且降低了效率。确实友元可以简化函数的定义和提高效率，但是反对友元的人则说友元函数破坏了类的封装性。依我看两者都有道理，用那一种就看编程者的心情了。
## 继承

继承就是通过一个类(基类)派生出一个新类(派生类)的过程，派生类自动具有基类的所有成员变量和函数，并且可以根据需要添加更多的成员函数或成员变量。
先举一个例子

下面我们先来看一个关于继承的小项目,项目结构如下：
Project:
human.h
human.cpp
student.h
student.cpp
teacher.h
teacher.cpp
main.cpp
makefile
上面的项目定义了一个Human类，然后由Human类派生出Student类和Teacher类，xxx.h为对应类的接口文件，xxx.cpp为对应类的实现文件，main.cpp为测试这三个类的文件，makefile为编译的文件。
human.h的内容：
``` c++
#ifndef HUMAN_H
#define HUMAN_H
# include<iostream>
# include <string>
using namespace std;
namespace human_namespace{
class Human;
class Human{
    public:
        Human(string _name,int _age);
        Human();
        string getName();
        void setName(string _name);
        int getAge();
        void setAge(int _age);
        //下面这个函数就是Studnet类的一个友元函数
        friend bool isSameName(Human s1,Human s2);
    private:
        string name;
        int age;
};
}
#endif

human.cpp的内容：

#include <iostream>
#include "human.h"
using namespace std;
namespace human_namespace{
    Human::Human(string _name,int _age):name(_name),age(_age){
        //有意留空
    }
    
    Human::Human():name("none"),age(18){
        //有意留空
    }
    
    string Human::getName(){
        return name;
    }
    
    void Human::setName(string _name){
        name=_name;
    }
    
    int Human::getAge(){
        return age;
    }
    
    void Human::setAge(int _age){
        age=_age;
    }
    
    bool isSameName(Human s1,Human s2){
        return s1.name==s2.name;
    }
}
```
student.h的内容：
``` c++
#ifndef STUDENT_H
#define STUDENT_H
#include <string>
#include "human.h"
using namespace human_namespace;
using namespace std;
namespace student_namespace{
    class Student:public Human{
    public:
        Student(string _name,int _age,string _schoolNmae);
        Student();
        void setSchoolName(string _schoolName);
        string getSchoolName(void);
        void studying(void);
    private:
        string schoolName;
    };
}
#endif

student.cpp的内容：

#include <iostream>
#include <string>
#include "student.h"
using namespace std;
using namespace human_namespace;
namespace student_namespace{
    Student::Student(string _name,int _age,string _schoolName):Human(_name,_age),schoolName(_schoolName){
        //主体有意留空
    }
    Student::Student():Human(),schoolName("none"){
        //主体有意留空
    }
    void Student::setSchoolName(string _schoolName){
        schoolName=_schoolName;
    }
    string Student::getSchoolName(){
        return schoolName;
    }
    void Student::studying(void){
        cout<<"我是学生"<<getName()<<"，我在学习！！！"<<endl;
    }
}
```
teacher.h的内容：
``` c++
#ifndef TEACHER_H
#define TEACHER_H
#include <string>
#include "human.h"
using namespace std;
using namespace human_namespace;
namespace teacher_namespace{
    
        class Teacher: public Human{
        public:
            Teacher(string _name,int _age,string _subject);
            Teacher();
            void setSubject(string _schoolName);
            string getSubject(void);
            void teaching(void);
        private:
            string subject;
        };
    }
#endif
```
teacher.cpp的内容：
``` c++
#include <iostream>
#include <string>
#include "teacher.h"
using namespace std;
using namespace human_namespace;
namespace teacher_namespace{
    Teacher::Teacher(string _name,int _age,string _subject):Human(_name,_age),subject(_subject){
        //主体有意留空
    }
    Teacher::Teacher():Human(),subject("none"){
        //主体有意留空
    }
    void Teacher::setSubject(string _subject){
        subject=_subject;
    }
    string Teacher::getSubject(){
        return subject;
    }
    void Teacher::teaching(void){
        cout<<"我是一个老师，我在教书！！！"<<endl;
    }
}
```
main.cpp的内容:
``` c++
#include <iostream>
#include "student.h"
#include "teacher.h"
using namespace std;
using namespace student_namespace;
using namespace teacher_namespace;
int main() {
    Student s1=Student("小明",25,"黄冈中学");
    Student s2;
    Teacher t1=Teacher("老王",40,"语文");
    Teacher t2;
    cout<<s1.getName()<<endl;
    cout<<s2.getName()<<endl;
    s1.studying();
    s1.setName("小小明");
    s1.studying();
    cout<<t1.getSubject()<<endl;
    cout<<t2.getAge()<<endl;
}
```
makefile的内容：
``` c++
main.exe:student.o teacher.o main.o human.o
	g++ -o main.exe student.o teacher.o main.o human.o
human.o:human.cpp human.h
student.o:student.cpp student.h  
	g++ -c student.cpp student.h 
teacher.o:teacher.cpp teacher.h 
	g++ -c teacher.cpp teacher.h 
main.o:main.cpp
	g++ -c main.cpp
cleanall:
	rm main.exe student.o teacher.o main.o human.o student.h.gch teacher.h.gch
cleansome:
	rm student.o teacher.o main.o human.o student.h.gch teacher.h.gch
```
继承的编码语法这里就不在赘述。派生类自动获得基类的所有成员变量和成员函数（特例：一些特殊的成员函数，比如构造函数将不会被自动继承，私有成员函数根本就不会被继承），继承的成员函数和成员变量不在派生类的定义中提到(特例：如果你需要更改一个继承的函数的定义，那么你需要在派生类的定义中列出它)，但他们会自动成为派生类的成员。
## 派生类中的构造函数

基类中的构造函数不被派生类继承，但可以在派生类的构造函数的定义中调用基类的构造函数。要用一种特殊的语法来调用基类的构造函数，即初始化区域：
``` c++
Teacher::Teacher(string _name,int _age,string _subject):Human(_name,_age),subject(_subject){
        //主体有意留空
    }
Teacher::Teacher():Human(),subject("none"){
    //主体有意留空
}
```
Teacher类继承自Human类，这个Teacher类的构造函数在定义是就是调用了Human类的构造函数。对派生类定义构造函数应该包括对某个基类构造函数的调用，并将这个调用放在构造函数定义的初始化区域。假如不包括对任何基础类构造函数的调用，那么在调用派生类构造函数时，会自动调用基类的默认构造函数(即没有参数的那个构造函数)，如果基类并没有定义这样的默认构造函数，那么在编译时就会报错。
派生类成员函数定义中可以使用来自基类的私有变量吗？

答案当然是不可以。就如上面的小项目，Student类继承自Human类，现在在Student类中增加了一个方法studying,这个方法需要用到name,上面我是这样处理的：
``` c++
void Student::studying(void){
        cout<<"我是学生"<<getName()<<"，我在学习！！！"<<endl;
    }
```
这里调用的是继承得来的getName()这一取值函数来获取当前对象的name,按理说name这一个基类的私有变量也被继承才对，为何不直接使用name呢？这里需要注意，name是基类Human的私有成员变量，所以只有在Human类的成员函数的定义中才能够直接访问，在其他的任何类中（包括派生类）的成员函数的定义中都不能够直接通过名称来访问.虽然Student类有一个名为name的成员变量(从Human继承得来)，但是在Student类定义的任何成员函数中对成员name的任何直接访问都是非法的。其它的继承来的私有属性或函数也一样。
## 私有成员函数不会被继承

上面讲到除非在基类的接口与实现中，否则不能直接访问基类的私有成员变量和私有成员函数，即使是在派生类的一个成员函数的定义中。私有成员变量与私有成员函数类似，但是对私有成员函数来讲这种限制似乎更加严格，在派生类中，基类私有成员变量好歹可以通过取值函数和赋值函数来操作，但是私有成员函数则根本就不可用了。事实上私有成员函数根本就不会被继承下来。
## protected限定符

在前面类的定义中只使用了两种类的成员:public和private,其实还存在第三种:protected.对除了基类的派生类以外的所有类以及外部函数来说，基类中用protected标记的成员和用private标记的成员没有任何区别。但是对与基类的派生类来说，则可以在自己的成员函数的定义中直接通过名称来访问基类的protected成员。如果在Human类中name这一属性没有被标记为private,而被标记为protected，那么Student类中studying函数应该像下面这样定义：
``` c++
void Student::studying(void){
        cout<<"我是学生"<<name<<"，我在学习！！！"<<endl;
    }
```
被protected标记的成员在继承时仍然成为派生类的protected成员，所以对派生类的派生类来讲，它仍然可以通过名称来直接访问在它的爷爷类中定义的protected成员。
同样的，对protected的使用也是说法不一，有人认为使用protected是一种不好的风格，因为它违背了“隐藏类实现的细节”这一原则。但也有人认为protected的使用简化了派生类成员函数定义，提高了效率。
## 重定义成员函数

继承时基类中的绝大多数成员函数会被原封不动的继承到派生类中，但有时这并不是我们想要的，我们可能还需要某些成员函数的定义做出一些变化，这时我们就需要对其进行重定义。重定义需要注意需要在派生类中明确列出需要修改定义的那些继承成员函数的声明，并且不可改变原函数的参数数量，顺序和类型。
## 重定义和重载的比较

不要混淆在派生类中对一个函数定义的重定义以及对一个函数名的重载。重定义函数时，派生类中给出的新函数定义和原函数具有相同的参数数量顺序和类型。与基类的中的函数定义相比，如果派生类中的函数使用了数量不同的参数，或者某个参数具有不同的类型，那么派生类实际上会同时存在两个函数，这成为重载，而非重定义。
## 访问重定义的基函数

假如我们重定义了一个函数，使其在派生类中的定义有别与基类中的定义，在这种情况下，并不是说基类中的定义就再也不能由派生类的对象使用了。要为派生类的对象调用函数的基类版本。
现在假定Human类中有一个saying函数，它不接受任何参数。Student类继承自Human类，并且在Student类中重定义了saying函数：
``` c++
//声明对象
Human A;
Student B;
//A,B分别调用自己版本的saying()
A.saying();
B.saying();
//B调用saying的基类版本
B.Human::saying();
```
## 多态

多态按字面的意思就是多种形态。当类之间存在层次结构，并且类之间是通过继承关联时，就会用到多态。C++ 多态意味着调用成员函数时，会根据调用函数的对象的类型来执行不同的函数。

下面的实例中，基类 Shape 被派生为两个类，如下所示：
``` c++
#include <iostream> 
using namespace std;
 
class Shape {
   protected:
      int width, height;
   public:
      Shape( int a=0, int b=0)
      {
         width = a;
         height = b;
      }
      int area()
      {
         cout << "Parent class area :" <<endl;
         return 0;
      }
};
class Rectangle: public Shape{
   public:
      Rectangle( int a=0, int b=0):Shape(a, b) { }
      int area ()
      { 
         cout << "Rectangle class area :" <<endl;
         return (width * height); 
      }
};
class Triangle: public Shape{
   public:
      Triangle( int a=0, int b=0):Shape(a, b) { }
      int area ()
      { 
         cout << "Triangle class area :" <<endl;
         return (width * height / 2); 
      }
};
// 程序的主函数
int main( )
{
   Shape *shape;
   Rectangle rec(10,7);
   Triangle  tri(10,5);
 
   // 存储矩形的地址
   shape = &rec;
   // 调用矩形的求面积函数 area
   shape->area();
 
   // 存储三角形的地址
   shape = &tri;
   // 调用三角形的求面积函数 area
   shape->area();
   
   return 0;
}
```
当上面的代码被编译和执行时，它会产生下列结果：
```
Parent class area
Parent class area
```
导致错误输出的原因是，调用函数 area() 被编译器设置为基类中的版本，这就是所谓的静态多态，或静态链接 - 函数调用在程序执行前就准备好了。有时候这也被称为早绑定，因为 area() 函数在程序编译期间就已经设置好了。
但现在，让我们对程序稍作修改，在 Shape 类中，area() 的声明前放置关键字 virtual，如下所示：
``` c++
class Shape {
   protected:
      int width, height;
   public:
      Shape( int a=0, int b=0)
      {
         width = a;
         height = b;
      }
      virtual int area()
      {
         cout << "Parent class area :" <<endl;
         return 0;
      }
};
```
修改后，当编译和执行前面的实例代码时，它会产生以下结果：
```
Rectangle class area
Triangle class area
```
此时，编译器看的是指针的内容，而不是它的类型。因此，由于 tri 和 rec 类的对象的地址存储在 *shape 中，所以会调用各自的 area() 函数。
正如您所看到的，每个子类都有一个函数 area() 的独立实现。这就是多态的一般使用方式。有了多态，您可以有多个不同的类，都带有同一个名称但具有不同实现的函数，函数的参数甚至可以是相同的。
## 虚函数

虚函数是在基类中使用关键字 virtual 声明的函数。在派生类中重新定义基类中定义的虚函数时，会告诉编译器不要静态链接到该函数。
我们想要的是在程序中任意点可以根据所调用的对象类型来选择调用的函数，这种操作被称为动态链接，或后期绑定。
## 纯虚函数

您可能想要在基类中定义虚函数，以便在派生类中重新定义该函数更好地适用于对象，但是您在基类中又不能对虚函数给出有意义的实现，这个时候就会用到纯虚函数。
我们可以把基类中的虚函数 area() 改写如下：
``` c++
class Shape {
   protected:
      int width, height;
   public:
      Shape( int a=0, int b=0)
      {
         width = a;
         height = b;
      }
      // pure virtual function
      virtual int area() = 0;
};
```
= 0 告诉编译器，函数没有主体，上面的虚函数是纯虚函数。

# 参考
C++面向对象程序设计(第七版)
