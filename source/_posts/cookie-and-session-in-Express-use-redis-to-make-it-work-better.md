---
title: 'Express框架中的Cookie，Session，以及Redis使用'
comments: true
date: 2017-09-17 20:33:05
updated: 2017-11-14 20:33:05
tags: [express,session,cookie,redis]
categories: NodeJS
permalink:
---
本文主要包括express框架中cookie和session的使用和需要注意的点，以及redis的使用，如何在express项目中集成redis来缓存session等内容。
# Cookie和Session基本原理
## What's Cookie?
众所周知，HTTP协议本身是无状态的，服务器不负责维护会话的状态。每次客户端访问完,服务器就会立即断开链接。客户端访问了服务器，服务器不会关心客户端是谁，他只是在人的事先规划之下根据客户端请求报文中包含的信息做出相应的响应，完成了此次响应，就会立即与客户端断开，然后去处理下一个请求(注意：在异步非阻塞的设计之下，服务器并不是完完全全处理完一个请求才处理下一个请求的，node就是这样的设计)。即使下一个请求仍然由同一个客户端发起，服务器也不会有所察觉，他还是会按照一个全新的请求去处理。这样的设计就导致了访问者的身份和状态无法确定，这样的话我们要想在网上做一些具有连贯性的任务就不可实现。比如用户登录，网上购物，游戏的积分，线上学习的进度情况等等。于是cookie出现了。cookie是存储在客户端主机的磁盘上的一串有特定意义的字符串。cookie的实现是客户端和服务器相互配合的结果。目前只要是还在使用的浏览器都支持了cookie的实现。当某一个请求到达服务器，服务器如果需要追踪会话状态的话，就会在响应报文中设置一些特定的数据，这些数据告诉浏览器这个网站需要设置cookie并且cookie中需要包含那些信息。浏览器解析到了相应的信息，就会按照大家事先沟通好的方式，将这些cookie数据写入自己所在主机的磁盘的某一个位置。下一次当再次访问这个网站时，浏览器就会携带之前记录的cookie。这些cookie数据送到服务器，服务器就可以根据它来确定当前来访用户的身份，进而找到数据库中关于此用户的信息，从而确定其会话状态。

## What’s session?

Session同Cookie一样，都是用来解决用户会话的跟踪问题。但是Cookie存在着安全的问题，Cookie中存储的数据必定含有一个可以确定用户身份的关键信息，这一个信息有可能是敏感信息，这样的信息存储在用户磁盘上是极其不安全的。Cookie可以被窃取，被篡改等。虽然也有加密cookie，但是不管怎么说，存储在客户端总是让人不放心的。并且要让客户端每次请求都要带着额外的一堆数据，这对于网络传输来说也是不利的。就在这时Session出现了，session是基于cookie的。但是和cookie不同的是，原来cookie中的数据转移到了服务器端，而客户端则只保留一个无意义的随机字符串，这个无意义的字符串与存储在服务器端的该客户端的相关信息有着一一对应的关系。现在客户端访问这个服务器就只需要带着这一个随机字符串，然后服务器就可以根据这一个字符串确定来访客户的会话状态了。Session与Cookie相比，安全性有了极大的提高，并且减少了客户端访问服务器时所携带的数据量。

# 在Express中使用Cookie或Session

## Express项目中使用Cookie

在express 4.x的版本的api中上行的req和下行的res中都有cookie,分别是读取cookie和设置cookie。在express中使用cookie需要用到cookie-parser这一个中间件，此中间件的使用方法是：
首先下载cookie-parser(如果你使用express-genarater生成项目的话，就已经下载好了cookie-parser，并且用到了项目中)

###下载cookie-parser

``` bash
$ npm install cookie-parser --save
```
### 使用
``` javascript
var express=require('express');
var cookieParser=require('cookie-parser');　//引包
app.use(cookieParser())　　　//这里的cookieParser()括号中没有字符串就是不带签名的cookie
//下面就可以使用cookie了
//设置cookie
res.cookie("username","xiaoming" ,{domain: '.example.com', path: '/admin', secure: true })
//这里的vaule是一个简单字符串也可以是一个JSON格式的字符串
//读取cookie
req.cookies.username
                 ->"xiaoming"
```
### 设置cookie时的参数

domain　：cookie在什么域名下有效，类型为String,。默认为网站域名
expires : cookie过期日期，类型为Date。如果没有设置或者设置为0，那么该cookie只在这个这个session有效，即关闭浏览器后，这个cookie会被浏览器删除。
httpOnly: 只能被web server访问，类型Boolean。不能通过js代码来读取cookie
maxAge : 实现expires的功能，设置cookie过期的时间，指明从现在开始，多少毫秒以后，cookie到期。
path : cookie在什么路径下有效，默认为’/‘，类型为String
secure ：只能被HTTPS使用，类型Boolean，默认为false
signed :使用签名，类型Boolean，默认为false。express会使用req.secret来完成签名，需要cookie-parser配合使用,将其设为true时，在取ｃｏｏｋｉｅ时要用 req.signedCookie（）方法
sameSite : Boolean or String Value of the “SameSite” Set-Cookie attribute.

### 签名cookie

要使用带签名的cookie(一种更安全的cookie的处理方式)，需要cookie-parser的配合：
``` javascript
//使用cookie-parser中间件的时候要传入secret
app.use(cookieParser("This is a secret that signedcookie needed"));
//设置cookie时要将signed设为true
res.cookie("username","andrewpqc",{signed:true})
//读取签名cookie时要使用req.signedCookies
req.signedCookies.username
					->andrewpqc
```
### 清除cookie

``` javascript
res.clearCookie('name', { path: '/admin' });
//这里设置的path要保持和设置该cookie时的一致
```
## Express项目中使用Session

Express中session的实现依赖于express-session

### 安装

``` bash
npm install --save express-session
```
### 使用
``` javascript
var express=require('express');
var session=require('express-session');
var app=express();
app.use(session({
	secret:"keyboard cat",
	resave:false,
	saveUninitialized:true,
	//cookie:{secret:true}//这里设为true时，是https协议使用的
	//注意在实验时这里的secret要设为ｆａｌｓｅ，因为本地用的是http协议
}));
app.get('/',function(req,res,next){
	if(req.session.login){
		res.send('欢迎您'＋','+req.session.username)
	}else{
		res.send('你还没有登录，请登录！')
	}
});
app.get('/login',function(req,res,next){
	req.session.login=true;        //设置session
	req.session.username="xiaoming";//设置session
	req.session.cookie.expires=new Date(Date.now()+3600000);//设置超时时间,这里１小时后失效
	res.send('你已经成功登陆')
});
```
注意，这里在使用时和cookie不同，cookie的req,res都对应有cookie接口，分别读取和设置cookie。而session则不同，无论是读取还是设置，都是通过req.session这一个接口完成的。session的数据并不是存储在cookie中，cookie中存储的仅仅是session ID,而其他的有用的信息存储在服务器端。

### 可选参数以及详细说明

``` javascript
app.use(session(OPTIONS))
```
这里有一点需要说明的是从express-session 1.5.0开始，这一个模块已经不再需要cookie-parser这一个中间件了。并且如果你在cookie-parser中使用的secret与express-session中的secret不一致的话就会起冲突。所以最好的处理办法是，如果你已经决定在项目中使用session的，你大可把cookie-parser从你的代码中去掉.

# redis的基本使用

## 为什么要使用redis?

session中的数据要存储在服务器端，怎么存就成了一个问题？可以存储在服务器所在主机的文件系统中，也可以存储在服务器端的数据库中，还可以直接存储在服务器的内存中。(express-session默认就将session数据直接缓存在服务器所在主机的内存中)但是无论是存储在文件中还是普通数据库中，最终程序还是需要在磁盘上面进行读写。大家知道这种数据从磁盘上的读写是非常耗时的。而sesssion数据则是需要频繁的使用的，这就意味着频繁的I/O操作，这对程序的性能是一个巨大的损害。无疑，把session数据存储在内存中是一个最好的选择。但是像express-session默认的将其直接存储在服务器主机的内存中也是不行的。随着网站用户数的增加，session数据会越来越多，这将极大的占用服务器的内存，从而使程序变慢。网站的维护也将变得困难，因为一旦服务器重启，内存中的session数据就将全部丢失。这种方式也不利于session数据的共享。目前最常用的方式就是专门提供一个数据库，它把数据存储在内存中，必要时也可以将数据写入磁盘中，并且还支持多个程序的连接，实现数据的共享，同时提供操作数据的接口，方便程序操作数据。Redis就是这样的一个数据库。我们把Redis单独放在一个稳定的内存大的主机之上，做成一个数据库服务器，这样的话，上述的所有问题就统统解决了。
其实除了可以存储session数据之外，redis还可以缓存很多东西。比如我们可以把html缓存在redis中，客户端发请求需要这个html页面，那么我们的程序就可以先在缓存中找这个页面，如果没有找到在去数据库中找。类似这样的操作可以极大的提高响应的速度，增强用户体验。

## redis使用(基于ubuntu系统)

### 安装redis

``` bash
$ sudo apt-get update
$ sudo apt-get install redis
```
上面的两条命令就可以安装redis了，但是这样安装的redis往往不是redis的最新版本，(但是也够用了，本人就是这样安装的，安装的是3.2.1版，当前最新的稳定版本是４．０．１)其实区别不大。
如果想要安装最新版可以使用下面的几条命令：
``` bash
$ wget http://download.redis.io/releases/redis-4.0.1.tar.gz
$ tar xzf redis-4.0.1.tar.gz
$ cd redis-4.0.1
$ make
```
这样的安装时间可能会长一些。

### 基本工具介绍

安装完成之后我们就有了两个基本工具了，服务启动工具：redis-server,命令行的客户端工具：redis-cli.

#### redis-server

``` bash
$ redis-server
```
上面这条命令就可以启动redis的服务了，redis默认监听的是６３７９端口。但是这种方式启动redis,是以前台的方式启动的，这样的话，我们就必须保持这个会话，关闭回话窗口或按下ctrl+c,服务就停止了。我们还可以以守护式进程的方式，让redis在后台运行。这就需要配置了,在/etc/redis.conf或者/etc/redis/redis.conf中，我们把daemonize后面的no改为yes就行了。这时我们再次启动redis

``` bash
$ redis-server /etc/redis/redis.conf
```
这次启动要在命令后面带上刚才修改了的配置文件的路径。这样的话，我们的redis就会在后台运行了。
注意：有的版本的redis默认daemonize的选项就是yes,也就是说默认就是在后台运行的，这时我们在运行$ redis-server就会告诉我们6379端口被占用。

#### redis-cli

redis-cli是一个redis的命令行客户端工具，用于连接和操作redis服务器。

``` bash
$ redis-cli
```
上面的这条命令，默认是连接到本地的6379端口，我们也可以指定主机名和端口号来进行远程链接

``` bash
$ redis-cli -h 127.0.0.1 -p 6379
```
这条命令同样链接到本地的6379
链接到了redis服务器之后，我们就进入了与redis服务器的会话交互中，我们可以通过一些指令来进行操作：

``` bash
$ keys *             //查看存储的所有的键
$ set hello world    //设置键hello对应的值为world
$ get hello          //查看键hello对应的值
```


#### GUI工具

推荐Redis Desktop Manager

### nodejs中操作redis

在nodejs中操作redis,需要一个redis的驱动。目前比较受欢迎的有两个node_redis,ioredis.下面我们就使用node-redis来讲解。

#### 安装node_redis

``` bash
$ npm install --save redis
```
#### 使用

``` javascript
//node_redis插件
const redis = require('redis');
//链接redis服务器
var client = redis.createClient(6379, "localhost");
//保存数据
client.set("this is a key", "this is a vaule");
//获取数据
client.get('this is a key', function(err, value) {
	if (err) {
		console.log(err.message);
		return;
	} else {
		console.log(value)
	}
});
//我们可以改写toString方法，让redis存储一个json字符串，这是我们可以把他
//从redis中取出来之后就可以转化成对象，接着使用
Object.prototype.toString = function() {
	return JSON.stringify(this)
};
//当然，最好不要改变原型链中的方法
//我们保存一个对象
client.set('key', {"a": 1,"b": 2});
client.get('key', function(err, value) {
	if (err) {
		console.log(err.message);
		return
	} else {
		console.log(vaule, typeof vaule)
	}
})
//从输出结果中可以看到，上面的打印ｖａｕｌｅ的类型，是一个字符串
//那是因为当存储的是一个对象的时候，redis会使用toString方法，把他转化成字符串，见上
//操作列表list
//从右边依次插入a,b,c,d,e
client.rpush('testList','a');
client.rpush('testList','b');
client.rpush('testList','c');
client.rpush('testList','d');
client.rpush('testList','e');
//同理，还有一个lpush
//从左边加数据
//删除数据（从右边删rpop,从左边删lpop）
client.rpop('testList',function(err,v){
	if(err){
		console.log(err.message);
		return;
	}else{
		console.log(v);//这里的v就是被删除的数据
	}
});
//读取列表
//取数据需要指定从哪里取到哪里，参数中如果是非负数就是从左边开始取，负数就是从右边开始取
//下面的这条语句就是取出全部的数据
client.lrange("testList",0,-1,function(err,list){
	if(err){
		console.log(err.message);
		return;
	}else{
		consol.log(list);
	}
})
//操作集合
//往集合里面添加东西
client.sadd("testSet",'a');
client.sadd("testSet",'b');
client.sadd("testSet",'c');
client.sadd("testSet",'d');
//集合里面的数据有互异性，在向里面添加数据时只会添加集合中没有的数据，集合中已经有的数据
//将无法插入
//从集合中取东西
client.smembers('testSet',function(err,values){
	console.log(values)
});
//消息中介
//通过redis的消息订阅与发布可以实现两个进程之间的通信
//消息的发布
client.publish('testPublish','message from hhh');
//消息的订阅
client.subscribe('testPublish');
client.on('message',function(channel,msg){
	console.log(channel,msg)
});
```

#### 在Express项目中用redis来缓存Session

在express项目中使用redis需要connect-redis这一个插件。首先要安装connect-redis

``` bash
npm install -save connect-redis
```
#### 使用

``` javascript
var express=require('express');
var redis=require('redis);
var session=require('express-session');
var RedisStore=require('connect-redis')(session);
var client=redis.createClient(6379,"127.0.0.1");
var app=express();
app.use(session({
	secret: 'recommand 128 bytes random string',
	resave:true,
	saveUninitialized:true,
	cookie:{},
	//在express-session中间件选项中加入store就可以了
	store:new RedisStore({client:client})
}));
```
配置完上面的内容之后，我们还是照常在我们的代码中去操作session，但是这是我们的session的数据就已经存在了redis的缓存系统中了。
