---
title: 伪共享
comments: true
date: 2020-01-11 21:34:41
updated: 2020-01-11 21:34:41
tags:
categories: OS
permalink:
---
# CPU缓存与缓存行
多核时代的到来使得并发编程愈来愈受到关注，这一节我们来了解一下号称多核并发编程中的性能刺客的伪共享(False Sharing)。要了解伪共享还得从CPU缓存讲起。
![](/images/cpu_cache.png)

CPU缓存是位于CPU和主存之间的临时存储器，它的容量比主存小得多但是交换速度比主存快得多。CPU缓存的基本单位是缓存行(Cache Line),缓存行的大小通常是64字节。一个缓存行与一块连续的64字节大小的主存对应。

# 伪共享
![](/images/cache-line-false-sharing.png)
``` go
type struct Test {
     X uint64
     Y uint64
}
```
如上图所示,T变量的X,Y两个成员被缓存在了同一个Cache Line中，并且Core1和Core2都缓存了一份。一旦一个Core修改了成员X或者Y,就会触发缓存一致性协议，导致对方Core的缓存失效，从而对方Core需要从上一级缓存中重新加载Cache Line, 而这个开销是及其昂贵的。这里描述的不同core上的执行流并发写同一个变量，相互失效对方core的缓存行就是所谓的伪共享(False Sharing).

# 规避
```
sysctl -a | grep cache | grep size
```

```
fs.generic.nfs.server.reqcache_size: 64
hw.l3cachesize: 3145728
hw.l2cachesize: 262144
hw.l1dcachesize: 32768
hw.l1icachesize: 32768
hw.cachelinesize: 64
hw.cachesize: 8589934592 32768 262144 3145728 0 0 0 0 0 0
machdep.cpu.cache.size: 256
machdep.cpu.cache.linesize: 64
```

规避方法很简单，找到系统的cache line大小，在成员X,Y中间插入padding成员，将X,Y分到不同的缓冲行中即可。
``` go
type struct Test {
     X uint64    // 8
     p [7]uint64 // 56 padding   
     Y uint64    // 8
}
```


