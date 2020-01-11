---
title: Python内置的序列类型
comments: true
date: 2017-10-29 15:38:40
updated: 2017-11-16 15:38:40
tags: [python sequence]
categories: Python
permalink:
---
# 内置序列类型概览
按照可否存储不同数据类型来分，可以分为容序列和扁平序列。

- 容器序列
list, tuple, collections.deque　这些序列类型可以容纳不同类型的数据。
- 扁平序列
str, bytes, bytearray, memoryview, array.array　这类序列只能容纳一种类型的数据

按照是否可变可以分为可变序列和不可变序列。
- 可变序列
list, bytesarray, array.array, collections.deque, memoryview
- 不可变序列
tuple, str, bytes

# 列表推导式和生成器表达式
列表推导是构建列表的快捷方式。而生成器表达式则可以用来创建其他的任何序列。
## 善于使用列表推导式
``` python
from pprint import pprint
a=[1,2,3]
b=["a","b","c"]
c=[(d,e) for d,e in zip(a,b)]
print(c)
g=[(i,j) for i in a for j in b]
pprint(g)
"""注意比较c,g两个列表推导式的区别"""
h=[i+i for i in range(10) if i % 2==0]
print(h)
#生成器表达式
#把列表推导式的中括号改为圆括号就得到了生成器表达式
a=(i for i in range(10))
print(a)
print(a.__next__())
print(a.__next__())
print(a.__next__())
print(a.__next__())
print(a.__next__())
#当生成器表达式作为函数调用过程中的唯一
#参数的时候不需要再用圆括号括起来
```
输出：
``` python
[(1, 'a'), (2, 'b'), (3, 'c')]
[(1, 'a'),
 (1, 'b'),
 (1, 'c'),
 (2, 'a'),
 (2, 'b'),
 (2, 'c'),
 (3, 'a'),
 (3, 'b'),
 (3, 'c')]
 [0, 4, 8, 12, 16]
<generator object <genexpr> at 0x7f28a9f3fc50>
0
1
2
3
4
```
## 元组不仅仅是不可变列表
如果仅仅是把元组理解为不可变的列表，那其他的信息——他所含有的元素的总数和他们的位置似乎就变得可有可无了。但是这样并没有发挥元组本来所有的力量。元组的特点就是它所记录数据的数量和位置。

## 元组拆包
``` python
#拆包
a=(1,2,3)
n1,n2,n3=a
print(n1)
print(n2)
print(n3)
#用*处理剩下的元素
b=("aaa","bbb","ccc","ddd")
b1,b2,*rest=b
print(b1)
print(b2)
print(rest)
#变量的值
m=1
n=2
print(m,n)
m,n=n,m
print(m,n)
#用*运算符把一个可迭代对象拆开作为函数的参数
c=(5,6)
def myadd(a,b):
    return a+b
print(myadd(*c))
```
输出：
```
1
2
3
aaa
bbb
['ccc', 'ddd']
1 2
2 1
11
```
## 具名元组

collctions.namedtuple()是一个工厂函数，它可以用来构建一个带字段名的元组和一个有名字的类。

### 基本使用
``` python
from collections import namedtuple
City=namedtuple("City",["name","country","population"])
Beijing=City("Beijing","China",100)
Tokyo=City("Tokyo","Japan",96)
print(Beijing.country)
print(Tokyo.population)
print(Tokyo[1])
```
输出：
``` python
China
96
Japan
```
创建一个具名元组需要两个参数，一个是类名，另一个是类的各个字段的名字。后者可以是由数个字符串组成的可迭代对象，或者是由空格分隔开的字段名组成的字符串。存放在对应字段里的数据要以一串参数的形式传入到构造函数中。可以通过字段名或者位置来获取一个字段的信息。

### 具名元组的属性和方法
``` python
print(City._fields)#获得所有的字段名
cityMsg=("Shanghai","China",99)
shanghai=City._make(cityMsg)#创建对象的一种方式，和City(*cityMsg)一样
print(shanghai._asdict())
for key,value in shanghai._asdict().items():
    print(key,":",value)
```
输出：
``` python
('name', 'country', 'population')
OrderedDict([('name', 'Shanghai'), ('country', 'China'), ('population', 99)])
name : Shanghai
country : China
population : 99
```
## 序列排序
在这里我们用list这一个序列来作为例子讲解python中序列的排序，其他的可排序序列和list是一致。这里主要讨论list.sort()和sorted()。前者是对列表就地排序，它会返回None,将list改变为有序的list.而后者则不会改变传入的list，只是复制了一个新的list,将新的list调整顺序之后返回。无论是list.sort()还是sorted()都有两个可选的关键字参数：key,reverse。他们分别决定排序的标准和升降序。

## 用bisect来管理已排序的序列
bisect模块包含两个主要的函数，bisect和insort两个函数都是利用二分查找算法来在有序序列中查找或插入元素。
``` python
import bisect
a=[1,2,38,85]
b=bisect.bisect(a,39)
#返回39插入已升序排序序列a中后仍然满足升序规则的下标值
a.insert(b,39)
print(a)
#上面两步操作可以直接用下面的替代
c=[1,2,38,85]
d=bisect.insort(c,39)#直接插入并且保持升序
print(c)
```
输出：
```
[1, 2, 38, 39, 85]
[1, 2, 38, 39, 85]
```
# 当列表不是首选时
有时候因为列表实在太方便了，所以python程序员会过度使用它。但是，如果你只需要处理数字列表的话，数组可能是个更好的选择，下面我们就来探讨一下可以替换列表的数据结构。

## 数组

数组(array)可以紧凑盛放基本的变量，如：字符，整数，浮点数。他是一个序列类型，在表现上和list非常想，但是它存储的数据类型受到了限制，它只能存储相同的基本类型的数据。所存储的数据的类型由array对象初始化时传入的typecode指定。以下的为所支持的typecode:

| Typecode |	C Type	| Python Type |	Minimum size in bytes |
| :-----------: | :----------: | :-----------: | :--------------: |
| b |	signed char | int	|  1 |
| B	 | unsigned char |	int |	1 |
| u	 | Py_UNICODE |	Unicode character |	2 |
| h	 | signed short |	int |	2 |
| H  |	unsigned char |	int |	2 |
| i	| signed int |	int |	2 |
| I	 | unsigned int |	int |	2 |
| l |	signed long |	int |	4 |
| L	| unsigned long	 | int	| 4 |
| q |	signed long  long |	int |	8 |
|  Q |	unsigned long long |	int |	8 |
| f	 | float |	float	 |  4 |
| d |   double |	float |	8 |


``` python
from array import array
from random import random
a1=array('q',(i for i in range(100)))
#其基本上和list具备相同的行为
print(a1.typecode)
print(a1[2])
print(a1.count(55))
print(a1.pop())
a3=array('d',(random() for i in range(10**3)))
with open("float.bin","wb") as f:
    a3.tofile(f)
a2=array("d")
with open("float.bin","rb") as f2:
    a2.fromfile(f2,10**3)
print(a2[3])
```
## 内存视图

memoryview是一个内置类，他能让用户在不复制内容的情况下操作同一个数组的不同切片，在数据结构之间共享内存。

``` python
from array import array
a=array('h',[1,2,3])
print(a)
m1=memoryview(a)
print(len((m1)))
m1[0]=15
print(m1)
print(a)
```
输出：
``` 
array('h', [1, 2, 3])
3
<memory at 0x7f7d2df4dc48>
array('h', [15, 2, 3])
```
## 双向队列和其他形式的队列
``` python
from collections import deque
dq=deque(range(10),maxlen=10)
print(dq)
dq.rotate(3)
print(dq)
dq.rotate(-3)
print(dq)
dq.append(100)
print(dq)
dq.appendleft(-100)
print(dq)
dq.extend([111,222,333])
print(dq)
dq.extendleft([-111,-222,-333])
print(dq)
print(dq.pop())
print(dq)
print(dq.popleft())
print(dq)
```
输出：
``` 
deque([0, 1, 2, 3, 4, 5, 6, 7, 8, 9], maxlen=10)
deque([7, 8, 9, 0, 1, 2, 3, 4, 5, 6], maxlen=10)
deque([0, 1, 2, 3, 4, 5, 6, 7, 8, 9], maxlen=10)
deque([1, 2, 3, 4, 5, 6, 7, 8, 9, 100], maxlen=10)
deque([-100, 1, 2, 3, 4, 5, 6, 7, 8, 9], maxlen=10)
deque([3, 4, 5, 6, 7, 8, 9, 111, 222, 333], maxlen=10)
deque([-333, -222, -111, 3, 4, 5, 6, 7, 8, 9], maxlen=10)
9
deque([-333, -222, -111, 3, 4, 5, 6, 7, 8], maxlen=10)
-333
deque([-222, -111, 3, 4, 5, 6, 7, 8], maxlen=10)
```
