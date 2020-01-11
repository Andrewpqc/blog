---
title: go语言并发编程——goroutine和通道
comments: true
date: 2018-02-23 16:05:26
updated: 2018-03-02 20:57:20
tags: [goroutine,channel]
categories: GO
permalink:
---
![go](/images/go.jpeg)
Go语言有两种并发编程的风格：一种是goroutine和通道，它们支持通信顺序进程(Communicating Sequential Process,CSP),CSP是一种并发的模式，可以在不同的执行体(goroutine)之间传递值，但是变量本身局限于单一的执行体。另一种是共享内存多线程的传统模型，它们和在其他的主流语言中使用的线程类似。在这里主要介绍goroutine和通道。

# goroutine
## 概念及本质
在Go语言中每一个并发执行的活动称为goroutine.这一概念初看上去和线程有些相似，但实际上**goroutine并不是线程，它只是对线程的多路复用**。一个goroutine在**启动时只需要一个非常小的栈**，并且这个栈可以按需扩展和缩小(在GO1.4中，goroutine启动时的栈大小仅为2KB)。正是因为**goroutine的轻量级**，所以**goroutine的数量可以比线程的数量多得多**。当一个goroutine被阻塞时，它也会阻塞所复用的操作系统线程，而运行时环境(runtime)则会把位于被阻塞线程上的其他goroutine移动到其他未阻塞的线程上继续运行。

## 使用goroutine
goroutine的用法非常简单:只要把go关键字添加到任意一个具名函数或者匿名函数的前面，该函数就会成为一个goroutine。特殊的，当一个程序启动时，有一个goroutine来调用main函数，称它为主goroutine。其他的新的goroutine都是通过go语句进行创建。go语句本身的执行立即完成，并不会阻塞程序继续向下执行。
``` go
f()   //调用f();等待它返回
go f()//新建一个调用f()的goroutine,不用等待
```

下面是一个使用goroutine的例子:主goroutine中新建了一个goroutine用于指示程序仍然在运行，而主goroutine本身仍然在进行求45个斐波那契数的计算。
``` go
package main

import (
    "fmt"
    "time"
)

func main() {
    go spinner(100 * time.Millisecond)
    fmt.Println(fib(45))
}

func spinner(delay time.Duration) {
    for {
        for _, d := range `-\|/` {
            fmt.Printf("\r%", d)
            time.Sleep(delay)
        }
    }
}

func fib(a int) int {
    if a < 2 {
        return a
    } else {
        return fib(a-1) + fib(a-2)
    }
}
```
通过运行该程序我们可以发现，提示符在运行的那一刻起就开始了，与此同时程序在进行着计算，几秒钟之后程序打印出第45个斐波那契数，然后主goroutine退出，我们发现主goroutine创建的goroutine也被暴力的直接终结了。

## 等待goroutine
我们来看下面这个并发程序：
``` go
package main

import (
    "fmt"
)

func printNumbers(){
    for i:=0;i<10;i++{
        fmt.Printf("%d ",i)
    }
    fmt.Println()
}

func printLetters(){
    for i:='A';i<10+'A';i++{
        fmt.Printf("% ",i)
    }
    fmt.Println()
}

func main(){
    go printNumbers()
    go printLetters()
}
```
我们运行程序，发现程序并没有产生任何输出！这是因为在该用例中，主goroutine在它的两个goroutine能够产生输出之前就已经结束了，结束之前还暴力的终结了刚刚创建的两个goroutine。显然，我们需要一种机制使程序可以在确保所有goroutine都已经执行完毕的情况下，再执行下一项工作。

为此，GO语言在**sync**包中提供了一种名为**等待组**(WaitGroup)的机制，它的运作方式非常简单直接:
- 声明一个等待组；
- 使用**Add**方法为等待组的计时器设置值;
- 当一个goroutine完成它的工作时，使用**Done**方法对等待组的计时器执行减一操作；
- 调用**Wait**方法，该方法将一直阻塞，直达等待组计数器的值变为0.

下面是添加等待的版本:
``` go
package main

import (
    "fmt"
    "sync"
)

func printNumbers(wg *sync.WaitGroup){
    for i:=0;i<10;i++{
        fmt.Printf("%d ",i)
    }
    fmt.Println()
    wg.Done()
}

func printLetters(wg *sync.WaitGroup){
    for i:='A';i<10+'A';i++{
        fmt.Printf("% ",i)
    }
    fmt.Println()
    g.Done()
}

func main(){
    var wg sync.WaitGroup
    wg.Add(2)
    go printNumbers(&wg)
    go printLetters(&wg)
    wg.Wait()
}
```
输出(您的输出可能有所不同):
```
A B C D E F G H I J 
0 1 2 3 4 5 6 7 8 9
```
如上，程序产生了我们期望的输出:>)
# 通道
在上面我们已经知道怎样通过go关键字将普通函数转换成goroutine以便让其独立运行，还知道可以设置等待组来同步独立运行的多个goroutine。下面我们将要学习如何使用通道在多个不同的goroutine之间通信。

## 基本语法及性质
通道是一种带有类型的值，它可以让不同的goroutine互相通信。通道用make函数创建，该函数在被调用之后将返回一个指向底层数据结构的引用作为结果值。
``` go
unbufferedChannel:=make(chan int)
bufferChannel:=make(chan int, 10)
```
如上，我们创建了两个可以容纳int类型值的通道，一个是**无缓冲通道**，一个是**有缓冲通道**。如果用户在创建通道的时候，向make函数提供了可选的第三个参数，那么make函数将创建出一个带有给定大小缓冲区的缓冲通道;未给定可选参数或者可选参数给定的为0,创建的都是无缓冲通道，也叫做**同步通道**。

像map一样，通道是一个使用make创建的数据结构的引用，当复制或者作为参数传递到一个函数时，复制的是引用，这样调用者和被调用者都引用同一份数据结构。

通道有两个主要操作，发送和接收，发送语句从一个goroutine传输一个值到另一个在执行接收表达式的goroutine。
``` go
ch <- x //把x发送到通道ch
a:= <-ch //从通道ch取值赋给变量a
<-ch    //从通道ch取一个值，并丢弃。
```
通道还支持第三个操作：关闭。关闭操作设置一个标志位来指示值当前已经发送完毕，这个通道后面没有值了。针对关闭后的通道的发送操作将导致宕机。在一个已经关闭的通道上进行接收操作，将获取所有已经发送的值，直到通道为空，这时任何接收操作会立即完成，同时获取到一个通道元素对应的零值。关闭通道使用内置的close函数完成：
``` go
close(ch)
```
## 无缓冲通道
无缓冲通道上的发送操作将会阻塞，直到另一个goroutine在对应的通道上执行接收操作，这时值传送完成，两个goroutine都可以继续执行。相反，如果接收操作先执行，接收方goroutine将阻塞，直到另一个goroutine在通道上发送一个值。

使用无缓冲通道进行通信导致发送和接收goroutine同步化，因此，无缓冲通道才被称为同步通道。

利用无缓冲通道的同步化特点，我们也可以也可以实现上面等待组实现的功能:
``` go
package main

import (
    "fmt"
)

func printNumbers(ch chan int){
    for i:=0;i<10;i++{
        fmt.Printf("%d ",i)
    }
    fmt.Println()
    ch<-1 
}

func printLetters(ch chan int){
    for i:='A';i<10+'A';i++{
        fmt.Printf("% ",i)
    }
    fmt.Println()
    ch<-1
}

func main(){
    ch1,ch2:=make(chan int),make(chan int)
    go printNumbers(ch1)
    go printLetters(ch2)
    <-ch1
    <-ch2
}

```

## 缓冲通道
缓冲通道有一个元素队列，队列的最大长度在创建的时候通过make的容量参数来设置，如下可以创建一个可以容纳三个字符串的缓冲通道:
``` go
ch:=make(chan string, 3)
```
缓冲通道上的发送操作在队列的尾部插入一个元素，接收操作从队列的头部移除一个元素。如果通道满了，发送操作将会阻塞所在goroutine直到另一个goroutine对它进行接收操作来留出可用的空间。反过来，如果通道是空的，执行接收操作的goroutine阻塞，直到另一个goroutine在通道上发送数据。

通道既不满也不空时，接收操作和发送操作都不会阻塞，通过这种方式，通道的缓冲区将发送和接收操作进行了解耦。

有时，程序需要知道缓冲通道的缓冲区的容量或者当前存储的元素个数，可以通过内置的**cap**,**len**函数求得：
``` go
n:=cap(ch)//求缓冲通道的容量
m:=len(ch)//求缓冲通道当前元素个数
```
## 管道
通道可以用来连接goroutine,这样一个输出是另一个的输入。这种情形叫**管道**(pipeline)。下面的程序由三个goroutine组成，它们被两个通道连接起来，从而形成了一个三级管道。
``` go
func main(){
    naturals := make(chan int)
    squares := make(chan int)

    //counter
    go func(){
        for x:=0;;x++{
            naturals<-x
        }
    }()

    //squarer
    go func(){
        for{
            x:=<-naturals
            squares<-x*x
        }
    }()

    //printer(在主goroutine中)
    for{
        fmt.Printf("%d ",<-squares)
    }
}
```
counter产生连续的自然数，通过管道传输到squarer进行平方，平方后的结果又通过管道传输到printer打印出来。正如我们所预料的那样，程序输出了无限的平方序列0,1,4,9，…

如果我们想生成有限的数字怎么办呢？
如果发送方知道没有更多的数据需要发送了，告诉接受者所在goroutine可以停止等待是很有用的，这可以通过调用内置的close函数来关闭通道：
``` go
close(ch)
```
在通道关闭后，任何后续的发送操作将会导致应用崩溃。当被关闭的通道被读完(就是最后一个发送值被接收)后，所有后续的接收操作顺畅进行，只是获取的是对应通道元素类型的零值。

没有一个直接的方式来判断是否通道已经关闭，但是这里有接收操作的一个变种，它产生两个结果：一个是接收到的通道值，另一个是一个布尔值(通常称之为ok),它为true时代表接收成功，为false代表当前的接收操作在一个关闭并且读完的通道上。利用这一特性我们可以修改squarer:
``` go
//sqarer
go func(){
    for{
        x,ok:=<-naturals
        if !ok{//naturals通道已经关闭，并且值接收完成
            break
        }
        squares<-x*x
    }
    close(squares)
}
```
上面的语法比较笨拙，而模式比较通用，所以go语言提供了range循环语法以在通道上迭代。这个语法更方便接收在通道上所有发送值，接受完最后一个值后关闭循环。下面是利用这一特性改写之后的程序：
``` go
func main(){
    naturals := make(chan int)
    squares := make(chan int)

    //counter
    go func(){
        for x:=0;x<10;x++{
            naturals<-x
        }
        close(naturals)
    }()

    //squarer
    go func(){
        for x:=range naturals{
            squares<-x*x
        }
        close(squares)
    }()

    //printer(在主goroutine中)
    for m:=range squares{
        fmt.Printf("%d ",m)
    }
}
```
结束时关闭每一个通道并不是必需的，只有在通知接收方goroutine所有的数据都发送完毕的时候才需要关闭通道。

## 缓冲与无缓冲的选择
缓冲通道和无缓冲通道的选择、缓冲通道容量的选择都会对程序的正确性产生影响。无缓冲通道提供强同步保障，因为每一次发送都需要和对应的接收同步；对于缓冲通道，这些操作则是解耦的。在通常情况下，如果我们知道要发送的值数量的上限，我们会创建一个容量为此上限的缓冲通道，在接收第一个值之前就完成所有的发送。

# goroutine和通道应用技巧
## 通用的并行循环模式
我们来看看一些通用的并行模式，来并行执行所有的循环迭代。考虑一个生成全尺寸图片缩略图的程序，下面的程序在每一个图像文件名字列表上进行循环，然后给每一个图像产生一副缩略图:
``` go
func makeThumbnails(filenames []string){
    for _,f:=range filenames{
        if _,err:=thumbnail.ImageFile(f);err!=nil{
            log.Println(err)
        }
    }
}
```
很显然，处理文件的顺序没有关系，因为每一个缩放操作和其他的操作独立。像这样由一些完全独立的子问题组成的问题称为**高度并行**。高度并行的问题是最容易实现并行的，有许多并行机制来实现线性扩展。如下，我们给每一个任务一个go关键字:
``` go
func makeThumbnails2(filenames []string){
    for _,f:=range filenames{
        go thumbnail.ImageFile(f)
    }
}
```
让这些任务并行去执行。如果我们运行这个函数，会发现它运行的实在是太快了，快的让人不敢相信，并且它也没有按照我们的预期生成缩略图。原因是函数在没有完成想要完成的事情之前就已经返回了。它启动了所有的goroutine,每一个文件一个，但是没有等他们执行完毕。

要想解决上面的问题，方法就是使用上文中已经提到的**同步通道**或**等待组**。用这两种方法的示例代码如下:
**使用同步通道**:
``` go
func makeThumbnails3(filenames []string){
    ch:=make(chan struct{})
    for_,f:=range filenames{
        go func(f string){
            thumbnail.ImageFile(f)
            ch<-struct{}{}
        }(f)
    }

    for range filenames{
        <-ch
    }
}
```

**使用等待组**:
``` go
func makeThumbnails3(filenames []string){
    var wg sync.WaitGroup
    for_,f:=range filenames{
        wg.Add(1)
        go func(f string){
            defer wg.Done()
            thumbnail.ImageFile(f)
        }(f)
    }
    wg.Wait()
}
```
## 限制并发数量
我们来看一个简单的网络爬虫,它以广度优先的顺序来探索网页的链接：
``` go
func crawl(url string)[]string{
    fmt.Println(url)
    list,err:=links.Extract(url)
    if err!=nil{
        log.Println(err)
    }
    return list
}


func main(){
    worklist:=make(chan []string)
    go func(){worklist<-os.Args[1:]}()

    seen:=make(map[string]bool)
    for list:=range worklist{
        for _,url:=range list{
            if !seen[url]{
                seen[url]=true
                go func(u string){
                    worklist<-crawl(url)
                }(url)
            }
        }
    }
}
```
程序首先从命令行收集起始的url，然后并发的爬取这些页面并且从中提取出页面的新url,如果是之前已经爬取的页面则不再爬取，并且将这些新的url发送至通道中，供程序继续爬取。可见我们是通过对crawl的独立调用从而充分的利用了Web上的I/O并行机制。

这个爬虫高度并发，但是它有两个问题。一个是程序的并行度太高，一下子会创建太多的网络连接，超过程序能打开文件的限制。第二是这个程序永远不会结束。

无限制的并行通常不是一个好主意，因为系统中总有限制因素。例如对于计算型应用CPU核数，对于磁盘I/O操作磁头和磁盘的个数，下载流所使用的网络带宽，或者Web服务本身的容量等等。解决高并行的方法就是根据资源的可用情况限制并发的个数，以匹配合适的并行度。对于上面的程序我们有一个简单的方法就是确保对于_links.Extract()_的同时调用不超过n个。我们可以使用容量为n的缓冲通道来建立一个并发原语，称为***计数信号量***。概念上，对于缓冲通道中的n个空闲槽，每一个代表一个令牌，持有者可以执行，通过发送一个值到通道中领取令牌，从通道中接受一个值来释放令牌，创建一个新的空闲槽。这保证了在没有接收操作的时候，最多同时有n个发送。因为通道元素的类型在这里并不重要，所以我们使用struct{},它所占用的空间的大小为0.

同时，为了让程序终止，当任务列表为空且爬取goroutine都结束以后，需要从主循环退出。基于以上两点我们可以做出如下的改进:
``` go
var tokens = make(chan struct{},20)

func crawl(url string)[]string{
    fmt.Println(url)
    tokens<-struct{}{}//获取令牌
    list,err:=links.Extract(url)
    <-tokens　　　　　　//释放令牌
    if err!=nil{
        log.Println(err)
    }
    return list
}

func main(){
    worklist:=make(chan []string)
    var n int
    n ++  //n用来记录任务列表中的任务个数
    go func (){worklist<-os.Args[1:]}()
    seen:=make(map[string]bool)
    for ;n>0;n--{
        list := <-worklist
        for _,link:=range list{
            if !seen[link]{
                seen[link] = true
                n++
                go func (link string){
                    worklist<-crawl(link)
                }(link)
            }
        }
    }
}
```

## 节拍器
在go语言的**time包**中内置了节拍器,time.Tick函数返回一个通道，它定期发送事件，像一个节拍器一样。每个事件的值是一个时间戳，一般我们不关心通道里面的内容，而是把它当做执行定时任务的工具:
``` go
func main(){
    fmt.Println("Commencing countdown.")
    tick:=time.Tick(1*time.Second)//时间间隔为１s
    for countdown:=10;countdown>0;countdown--{
        fmt.Println(countdown)
        <-tick
    }
}
```
如上，我们利用节拍器实现了一个倒计时的功能，每一次for循环会被阻塞一秒，这样程序就是一秒输出一个数字。

Tick函数很方便使用，但是它仅仅在应用的整个生命周期中都需要时才合适，否则我们就需要使用这个模式:
``` go
ticker:=time.NewTicker(1*time.Second)
<-ticker.C    //从ticker的通道里面接收
ticker.Stop()　//当不需要该节拍器之后，可以显式的停止它
```

## 使用select多路复用
现在我要给上面的倒计时程序加一个需求，在倒计时的过程中，如果用户按下键盘，则倒计时终止。下面的程序实现了这个需求:
``` go
func main(){
    abort:=make(chan struct{})
    go func (){
        os.Stdin.Read(make([]byte,1))
        abort<-struct{}
    }
    fmt.Println("Commencing countdown.")
    tick:=time.Tick(1*time.Second)//时间间隔为１s
    for countdown:=10;countdown>0;countdown--{
        fmt.Println(countdown)
        select{
        case <-tick:
            //什么都不做
        case <-abort:
            fmt.Println("abort...!")
            return//返回，终止  
        }
    }
}
```
每一次倒计时迭代需要等待事件到达两个通道中的一个：计时器通道(前提是一切顺利，即用户没有按键)，中止事件通道(前提是有异常，即用户按键)。
如上，我们使用了一个select语句实现了该功能,下面是select语句的通用形式:
``` go
select{
case <-ch1:
    //code block 1
case z:<-ch2:
    //code block 2
case ch3<-data:
    //code block 3
default:
    //code block
}
```
像switch语句一样，它有一系列的情况和一个可选的默认分支，每一个情况指定一次通信(在一些通信上进行发送或者接收操作)，和相关联的一段代码块。接收表达式操作可能出现在它本身，像第一种情况，或者在一个短变量声明中，像第二种情况，第二种形式可以让你引用你所接收的值。

select一直等待，直到一次通信来告知有一些情况可以执行。然后，它进行它进行这次通信，执行此情况所对应的语句；其他的通信将不会发生。对于没有对应情况的select语句，它将会永远等待。当有多个情况同时满足的时候，select随机选择一个，这样保证每一个通道有相同的机会被选中。