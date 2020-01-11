---
title: Python数据模型
comments: true
date: 2017-10-28 15:27:13
updated: 2017-11-16 15:27:13
tags: [Fluent Python]
categories: Python
permalink:
---
最近在读一本python的书Fluent Python,收获很多。如果你是一个有一定python编程经验,并且想让自己的代码变的更加pythonic,那么这本书就非常适合你。就我个人而言，看了这本书前面的几章，感觉自己以前写的东西都不是python,之前自己对python的理解太过于肤浅了。这本书打开了我重新认识python的大门。

![fluent python](/images/fp.jpg)
书中在深入介绍许多重要的主题时还穿插着介绍了许多我自己编程时很少用到的但是又非常有用的tips，全书行文逻辑比较复杂，感觉自己无法把每一部分的内容都理清，所以干脆就按照原书中的章节和知识点记录并介绍。下面就开始第一章的内容。

# 一摞python风格的纸牌
第一章里作者就以一摞python风格的纸牌这个例子向我们展示了python语言的强大和优美。其中涉及到python的命名元组，列表推导，`__len__`,`__getitem__`两个特殊方法等知识点。

下面话不多说，直接拿出代码：
``` python
import collections
#创建了一个命名元祖Card
Card=collections.namedtuple("Card",["value","style"])
#创建了一个纸牌类
class FrenchDeck(object):
    #准备纸牌的牌面值和花色，使用了列表推导和字符串分割
    values=[str(i) for i in range(2,11)]+list("JQKA")
    styles="spades diamonds clubs hearts".split()
    def __init__(self):
        self._cards=[Card(value,style) for value in self.values
                     for style in self.styles]
    #实现__len__魔术方法，使得我们的FrenchDeck的对象支持len()求元素个数
    def __len__(self):
        return len(self._cards)
    #实现了__getitem__魔术方法，使得我们的FrenchDeck的对象支持切片
    def __getitem__(self, position):
        return self._cards[position]
```
经过上面的简单定义，我们就得到了一个功能非常强大的类，下面我们就可以使用这个类了：
``` python
T=FrenchDeck()
#源于__len__方法
length=len(T)
print(length)
#源于__getitem__方法
print(T[2])
print(T[1:5])
print(T[1:5:2])
from random import choice
a=choice(T)
print(a)
```
输出结果：
``` python
Card(value='2', style='clubs')
[Card(value='2', style='diamonds'), Card(value='2', style='clubs'), Card(value='2',style='hearts'), Card(value='3', style='spades')]
[Card(value='2', style='diamonds'), Card(value='2', style='hearts')]
Card(value='10', style='clubs')
```
## 命名元组

我们可以使用collections.namedtuple(命名元组)来构建一个简单的类，这个简单的类可以用来构建只有少数属性而没有方法的对象。比如上面的Card=collections.namedtuple("Card",["value","style"])语句就是创建了一个只有vaule,style两个属性的类Card.
``` python
import collections
#比如下面就是构建了HH类，他有a,b两个属性
HH=collections.namedtuple("HH",['a','b'])
#实例化H1这个对象
H1=HH(1,2)
print(H1.a+H1.b)#输出３
```
现在就可以简单的将命名元组理解为只有属性没有方法的类，虽然在绝大多数的时候我们也是这样做的。关于命名元组的更多介绍将在介绍元组时讲到。

## __len__和__getitem__

在这个例子中第二个重要的知识点就是`__len__`和`__getitem__`两个魔术方法的使用，使用了`__len__`就让我们的对象直接支持内置的len()方法求元素的个数。而`__getitem__`直接就让我们的对象支持了和列表等可迭代对象同等的切片操作，并且让我们的对象直接成为可迭代对象。关于特殊方法的介绍是本书的重点，后面会介绍许多这样的方法。

# 如何使用特殊方法
首先要明确的是，特殊方法的实现是为了被python解释器调用，你自己并不需要调用他们，也就是说没有`object.__len__()`这种写法，而应该写成`len(object)`。如果object是你自己定义的类的对象，那么`len(object)`执行时，python解释器就会自己去调用由你实现的`__len__()`方法。对于python的内置类型来说，比如list,str等，Cpython会抄个近路，`__len__()`会直接读取底层结构体中存储用来表示当前元素个数的变量的值并且返回，这就比逐个计数快许多。

## 模拟数值类型

下面我们通过一个特殊方法实现一个二维向量类：
``` python
from math import hypot
#hypot(x,y)返回x**2+y**2的和开平方
class Vector(object):
    def __init__(self,x=0,y=0):
        self.x=x
        self.y=y
    def __add__(self, other):
        """支持加法"""
        return Vector(self.x+other.x,self.y+other.y)
    def __mul__(self, other):
        """支持乘法"""
        return Vector(self.x*other,self.y*other)
    def __abs__(self):
        """支持求绝对值(求模)"""
        return hypot(self.x,self.y)
    def __repr__(self):
        return "Vector Object Vector(%s,%s)"%(self.x,self.y)
    def __bool__(self):
        """长度是否为０"""
        return bool(abs(self))
```
下面我们就可以使用这个Vector类了。
``` python
v1=Vector(1,2)
v2=Vector(2,3)
v3=v1+v2
print(v3)
v4=v1*3
print(v4)
print(abs(v1))
v5=Vector()
print(bool(v1))
print(bool(v5))
```
输出：
``` python
Vector Object Vector(3,5)
Vector Object Vector(3,6)
2.23606797749979
True
False
```
## 字符串的表示形式

python中有两个内置函数repr()和str()两者都可以把一个对象用字符串的形式表达出来以便于辨认。当然，str()还有着类型转换的作用。对于自定义类的对象来说，他们两个分别依赖于__repr__(),__str__()两个特殊方法。交互式控制台和调试程序用repr函数来获得对象的字符串表示形式。而在程序中使用str(object)函数或者print(object)的时候会调用__str__().如果你只想实现两个特殊方法中的一个，__repr__()是个更好的选择，因为一个对象没有__str__()函数的时候，而python又需要调用它的时候，解释器会用__repr__()作为替代。

## 算术运算符

在Vector的例子中，我们通过__add__()和__mul__()为Vector类带来了+和*这两个运算符。需要注意的是，这两个方法的返回值都是新创建的向量对象，被操作的对象本身并没有发生变化，在程序中只是读取了他们的值而已。

## 自定义的布尔值

尽管python里有bool类型，但是实际上任何的对象都可以用于需要布尔值的上下文中。为了判断一个值x是真是假，python会调用bool(x),这个函数只能返回True或者False.

默认情况下，我们自己定义的类的实例总被认为是真，除非这个类对__bool__()或者__len__()函数有自己的实现。bool(x)的背后是调用x.__bool__()的结果，如果不存在这个方法，那么bool(x)会尝试调用x.__len__()如果返回0,则bool(x)会返回False,否则返回True.

# python之禅(The Zen of Python)
在python交互式控制台中输入import this即可看到。
``` 
The Zen of Python, by Tim Peters
Beautiful is better than ugly.
Explicit is better than implicit.
Simple is better than complex.
Complex is better than complicated.
Flat is better than nested.
Sparse is better than dense.
Readability counts.
Special cases aren't special enough to break the rules.
Although practicality beats purity.
Errors should never pass silently.
Unless explicitly silenced.
In the face of ambiguity, refuse the temptation to guess.
There should be one-- and preferably only one --obvious way to do it.
Although that way may not be obvious at first unless you're Dutch.
Now is better than never.
Although never is often better than *right* now.
If the implementation is hard to explain, it's a bad idea.
If the implementation is easy to explain, it may be a good idea.
Namespaces are one honking great idea -- let's do more of those!
```