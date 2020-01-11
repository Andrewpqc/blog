---
title: 限制资源访问
comments: true
date: 2018-03-09 13:01:25
updated: 2018-03-09 13:01:25
tags: [NGINX]
categories: Nginx
permalink:
---
昨天爬了一下小幸运，爬取了大约4000条左右的愿望信息。爬取的过程中我没有设置代理和设置延迟，任由代码迅速的发送请求。爬取的过程相当顺利。这说明小幸运的服务端没有做限制来访IP请求频率的处理。这很容易引发DDOS(分布式拒绝服务)攻击。基于此，我研究了一下对客户端请求限制的方法。收获记录如下!

NGINX有多个内建的模块可以帮助我们控制客户端对应用的使用。比如:限制连接数，限制请求和响应的速率，限制带宽等。
## 限制连接
我们可以通过某一个预定义的变量来限制某一客户端的连接数，通常这个预定义的变量是客户端的IP地址。我们可以构建一个共享内存的域来存储连接指标，然后使用**limit_conn**指令来限制打开的连接数，具体的配置如下：
``` nginx
http{
    limit_conn_zone $binary_remote_addr zone=limitbyaddr:10m;
    limit_conn_status 429;
    ...
    server{
        ...
        limit_conn limitbyaddr 40;
        ...
    }
}
```
上述配置文件创建了一个共享内存区域(zone)并命名为limitbyaddr。使用的预定义的变量是**$binary_remote_addr**,也就是客户端的IP地址的二进制形式。共享内存区域的大小被设置为了10MB.**limit_conn**指令需要两个参数：一个是**limit_conn_zone**的名字，第二个是允许的最大连接数。**limit_conn_status**指令设置连接数超过指定值之后的服务器响应,默认的响应是503(service unavailable)，在上述配置中如果同一个客户端同时打开的连接数超过40,那么就会收到429响应(too many requests)。**limit_conn_zone**只在http指令块中可用，**limit_conn**和**limit_conn_status**在http,server,location指令块中都可用。


## 限制请求速率
通过预定义的变量可以来限制请求的速率，通常这个预定义的变量也是客户端的IP地址。利用rate-limiting模块来限制请求的速率。配置如下：
``` nginx
http{
    limit_req_zone $binary_remote_addr zone=limitbyaddr:10m rate=1r/s;
    limit_req_status 429;
    ...
    server{
        ...
        limit_req zone=limitbyaddr burst=10 nodelay;
        ...
    }
}
```

第一个参数:设置使用哪个配置区域来做限制，与上面limit\_req_zone 里的name对应
第二个参数：burst=5，重点说明一下这个配置，burst爆发的意思，这个配置的意思是设置一个大小为5的缓冲区，当有大量请求（爆发）过来时，超过了访问频次限制的请求可以先放到这个缓冲区内
第三个参数：nodelay，如果设置，超过访问频次而且缓冲区也满了的时候就会直接返回**limit_req_status**的值，如果没有设置，则所有请求会等待排队

上述配置创建了一个共享内存的区域(zone)并将其命名为limitbyaddr.预定义的键是客户端的IP地址的二进制形式。共享内存区域的大小被设置成了10MB,这个共享内存区域还设置了rate的关键字参数，用来指定允许客户端请求的频率上限。**limit_req**指令需要两个可选的关键字参数：zone,burst。zone指明了使用的共享内存区域。当某一个共享内存区域的请求速率达到了之后，请求就会被延迟处理，直到达到burst值。burst值默认为0.**limit\_req**的第三个可选参数是:**nodelay**,此参数使客户端在受限制之前无延迟地使用其突发。

## 限制带宽
如果你需要限制每个客户端的下载带宽，你可以使用NGINX的**limit_rate**和**limit_rate_after**指令来实现:
``` nginx
location /download/ {
    ...
    limit_rate_after 10m;
    limit_rate 1m;
    ...
}
```
上面的配置指明了对于/download/开头的URL的请求，在成功返回10MB的数据之后，开始以每秒1MB的速率传输数据至客户端。