---
title: gcc/g++常用编译选项和gdb常用调试命令
date: 2018-11-25 17:17:58
updated: 2018-11-26 10:06:58
tags: [gcc/g++, gdb]
categories: C/C++
permalink:
---
[![](https://badge.juejin.im/entry/5bfac5b6e51d45127e7767b3/likes.svg?style=flat-square)](https://juejin.im/entry/5bfac5b6e51d45127e7767b3/detail)

gcc/g++编译器是我们写编译C/C++程序时离不开的编译工具，而gdb又是调试C/C++程序的利器，这一篇文章我们记录一下它们的惯常用法。

## gcc/g++常用编译选项
|   选项  | 作用    |
| :-----: | :------: |
| -c  | 生成可目标文件，但不进行链接 |
| -o  | 指定生成文件的文件名 |
| -g  | 在目标文件中添加调试信息，便于gdb调试或objdump反汇编 |
| -Wall  | 显示所有的警告信息(建议使用) |
| -Werror | 视警告为错误，出现警告即放弃编译 |
| -w  |   不显示任何警告信息(不建议使用) |
| -v  | 显示编译步骤 |
| -On |  (n=0,1,2,3) 设置编译器优化等级，O0为不优化，O3为最高等级优化，O1为默认优化等级 |
| -L  | 指定库文件的搜索目录 |
| -l  | (小写的L)链接某一库 |
| -I  | (大写的i)指定头文件路径 |
| -D  | 定义宏，例如-DAAA=1,-DBBBB |
| -U  | 取消宏定义，例如-UAAA |



## gdb常用调试命命令
我们的可执行文件要能够被gdb调试，必须在编译时加上调试信息，也即是加上`-g`选项.
``` bash
$ gcc -g example.c -o example
```
如上我们生成了一个带有调试信息的可执行文件example,要调试example可以执行下列命令:
``` bash
$ gdb example
```
这样我们就进入了gdb的调试命令行，如下所示:
```
GNU gdb (Ubuntu 8.1-0ubuntu3) 8.1.0.20180409-git
Copyright (C) 2018 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "x86_64-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
<http://www.gnu.org/software/gdb/documentation/>.
For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from example...done.
(gdb) 
```

如上可以看到命令行提示符为`(gdb) `,接着我们就可以在这个gdb的命令行提示符上面输入各种gdb的调试命令了(补充:这里也可以在shell中输入`gdb`,然后回车，这样直接进入到gdb的调试命令行，之后可以通过`file example`命令来载入待调试的可执行程序)。

下面我们就以如下的程序为例，详细的看下，在gdb的调试命令行中我们都有那些命令可用:
``` c
#include <stdio.h>

long func(int a){
    long sum = 0;
    for(int j=1;j<=a;j++){
    	sum += j;
    }
    return sum;
}

int main(void){
    int a =100;
    long sum = func(a);
    printf("%ld",sum);
    return 0;
}
```

### 查看源码
载入待调试的可执行文件之后，在gdb的命令行中输入`list`或者其简写`l`可以查看到程序的源码以及行号，如下:
``` gdb
(gdb) l
1	#include <stdio.h>
2	
3	long func(int a){
4	    long sum = 0;
5	    for(int j=1;j<=a;j++){
6	    	sum += j;
7	    }
8	    return sum;
9	}
10	
(gdb) 
11	int main(void){
12	    int a =100;
13	    long sum = func(a);
14	    printf("%ld",sum);
15	    return 0;
16	}
(gdb) 
Line number 17 out of range; example.c has 16 lines.
(gdb) 
```
如上输入`l`之后，默认会显示10行源代码，按回车之后会显示接下来的10行,直到文件的末尾。

### 添加断点
在gdb下添加断点使用命令`break`或简写 `b`，有下面几个常见用法（这里统一用 b:
```
b 函数名
b 行号
b 文件名:行号
b 行号 if条件
```
比如我们在 main函数和func函数上各添加一个断点：
```
(gdb) b main
Breakpoint 1 at 0x685: file example.c, line 12.
(gdb) break func
Breakpoint 2 at 0x651: file example.c, line 4.
```
如上我们成功加上了两个断点，在正确加上断点之后，会对应有一行输出，告诉我们断点的内存地址，断点对应的源文件名和行号。

### 查看断点
在加上断点之后，我们可以通过`info break`命令查看断点的信息:
```
(gdb) info break
Num     Type           Disp Enb Address            What
1       breakpoint     keep y   0x0000000000000685 in main at example.c:12
2       breakpoint     keep y   0x0000000000000651 in func at example.c:4
```
### 禁用和解禁断点
通过`disable <break number>`来禁用指定Num的断点，如下我们禁用1号断点:
```
(gdb) disable 1
(gdb) info break
Num     Type           Disp Enb Address            What
1       breakpoint     keep n   0x0000000000000685 in main at example.c:12
2       breakpoint     keep y   0x0000000000000651 in func at example.c:4
```
如上，`disable 1`之后，断点1的`Enb`列由之前的`y`变成了`n`，说明断点1已被禁用。

通过`eable <break number>`可以来解禁断点，如下我们对刚才禁用的断点1解禁:
```
(gdb) enable 1
(gdb) info break
Num     Type           Disp Enb Address            What
1       breakpoint     keep y   0x0000000000000685 in main at example.c:12
2       breakpoint     keep y   0x0000000000000651 in func at example.c:4
```
如上，断点1的Enb列又变成y了，它被成功解禁。

### 删除断点
我们可以用`delete <break number>`命令来删除掉一个断点，如下我们删除断点1:
```
(gdb) delete 1
(gdb) info break
Num     Type           Disp Enb Address            What
2       breakpoint     keep y   0x0000000000000651 in func at example.c:4
```
如上，断点1被成功删除。

### 启动程序
我们可以使用`run`命令或者简写`r`来启动程序的执行:
```
(gdb) r
Starting program: /home/andrew/example 

Breakpoint 2, func (a=100) at example.c:4
4	    long sum = 0;
```
如上，程序执行到断点2的时候就停止执行了。

### 查看变量的值
`p <variable name>`/`print <variable name>`可以查看某一个变量的当前值:
```
(gdb) p sum
$1 = 0
```
如上，当前sum的值为0

### 单步执行
`next`命令或者`n`可以单步执行，如下:
```
(gdb) n
5	    for(int j=1;j<=a;j++){
(gdb) p sum
$2 = 0
(gdb) n
6	    	sum += j;
(gdb) p sum
$3 = 0
(gdb) n
5	    for(int j=1;j<=a;j++){
(gdb) p sum
$4 = 1
(gdb) n
6	    	sum += j;
(gdb) p sum
$5 = 1
(gdb) n
5	    for(int j=1;j<=a;j++){
(gdb) p sum
$6 = 3
```

### 跳入跳出函数
```
(gdb) break 13
Breakpoint 1 at 0x693: file example.c, line 13.
(gdb) r
Starting program: /home/andrew/example 

Breakpoint 1, main () at example.c:13
13	    long sum = func(a);
(gdb) s
func (a=100) at example.c:4
4	    long sum = 0;
(gdb) n
5	    for(int j=1;j<=a;j++){
(gdb) n
6	    	sum += j;
(gdb) n
5	    for(int j=1;j<=a;j++){
(gdb) n
6	    	sum += j;
(gdb) finish
Run till exit from #0  func (a=100) at example.c:6
0x000055555555469d in main () at example.c:13
13	    long sum = func(a);
Value returned is $1 = 5050
(gdb) n
14	    printf("%ld",sum);
(gdb) 
```
如上，我在func()函数调用行加上了断点，然后`r`开始执行程序，之后程序在断点处停住，此时我执行`step`命令或其简写`s`来跳入func()函数内部调试，在内部依然像执行外部调试一样，如果要从函数跳出则执行`finished`，这时会导致函数执行完毕，并且打印出一些函数的返回信息，并且程序停在函数后的第一条语句处。

### 监控变量
使用`watch <varible name>`命令可以实现监控变量，使用`info watch`命令可以查看监控的变量。同时break所拥有的`enable`,`disable`,`delete`等动词对于watch依然使用，且用法大同小异。这里就不再赘述。


### 显示变量的值
使用`display <varible name>`命令可以在每一步执行之后打印变量的当前值,如下:
```
(gdb) start 
Temporary breakpoint 1 at 0x685: file example.c, line 12.
Starting program: /home/andrew/example 

Temporary breakpoint 1, main () at example.c:12
12	    int a =100, b =50;
(gdb) display sum
1: sum = 140737488346112
(gdb) n
13	    long sum = func(a);
1: sum = 140737488346112
(gdb) 
14	    printf("%ld",sum);
1: sum = 5050
(gdb) 
16	    long sum2 = func(b);
1: sum = 5050
(gdb) 
17	    printf("%ld",sum2);
1: sum = 5050
(gdb) 
18	    return 0;
1: sum = 5050
(gdb)
```

### 进入shell
`shell`命令可以让我们从gdb命令行环境进入到shell的命令行环境，当我们在shell命令行环境中输入exit退出后，我们就又回到了之前的gdb命令行环境了。


### 可视化调试
在gdb命令行环境中输入`wi`命令，可以让我们进入可视化调试环境，这个环境可以看到源代码，所使用的调试命令与上面讲到的一致。

