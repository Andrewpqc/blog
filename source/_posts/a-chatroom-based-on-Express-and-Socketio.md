---
title: 基于Socket.io和Express的聊天室
comments: true
date: 2017-08-31 08:06:27
updated: 2017-11-15 08:06:27
tags: [Socket.io,Express]
categories: NodeJS
permalink:
---
# WebSocket协议
HTTP协议是无状态的，每次处理完客户端的一个(http/1.0)或几个(http/1.1)请求就会立即断开并且。这就导致很难实现客户端与服务器数据的实时同步。以往大家实现实时通信都是通过ajax长轮询，和long poll长连接两种方式。ajax长轮询就是客户端每隔很短的时间就去访问一次服务器，看服务器那里有没有新的数据，如果有就将数据渲染在页面上，没有就一段时间之后又发起相同的请求。如果发起请求的间隔时间够短，就可以给人造成一种实时通信的错觉。ajax请求是一种非阻塞的请求，渲染消息的时候不用重新加载页面，所以用户体验比较好。但是这样频繁的发请求，并且大多数请求是无用的，这对服务器和客户端都造成了巨大的资源浪费。long poll长连接和ajax轮询其实是差不多的,只不过是采用阻塞的方式，如果服务器现在还没有消息，那么就不返回response给他，导致客户端一直在等待,直到等到了消息才返回，返回后又发起一个http请求，又到服务器哪儿去等着。上面的两种方式其实都是客户端不断的发起http请求，服务器被动的接受请求并处理。他们都不是很好的处理这个问题的方式。这时h5中提出了Websocket协议。Websocket解决了下面的几个问题：
１．在客户端与服务器首次连接之后，客户端与服务器的地位就平等了，两者之间可以进行真正的全双工通信。不仅客户端可以发送请求给服务器，服务器也可以主动通知客户端了。也就是说客户端再也不用一次一次的到服务器那里去询问有没有新数据，或者傻傻的到服务器那里去等着，直接在有新数据的时候让服务器通知一下客户端就行了。
２．首次连接之后，客户端与服务器之间的通信不需要发送头信息了，提高了信息交换的效率。

## Websocket的细节

首先是客户端发送一个http请求给服务器，这个请求头中包含转换协议的数据，服务器接受到之后就会在之后本次会话的其他心意转为tcp,之后两者就是以tcp协议来交流的，而不是http.由于websocket的这种设计与原来的通过http协议运作的浏览器与服务器的架构有很大的不同，所以websocket的应用需要浏览器和服务器以及中间可能有的代理服务器的共同的支持。好在现在的主流的高版本的浏览器，和主流的高版本的服务器软件都已经支持了websocket这一协议，并且越来越多的产品都在慢慢的支持websocket.

## socket.IO库

socket.IO是一个基于websocket的一个实时获取浏览器/客户端数据的库，他能够使开发者不必关心websocket底层的细节，而只需要关心自己的业务逻辑，顶层调用非常简单。并且，它还为不支持websocket协议的浏览器，提供了长轮询的透明模拟机制,它会自动根据浏览器从WebSocket、AJAX长轮询、Iframe流等等各种方式中选择最佳的方式来实现网络实时应用，非常方便和人性化，而且支持的浏览器最低达IE5.5。这使得在大部分情境下，你都能通过socket.io与浏览器保持类似长连接的功能。由于nodejs的单线程，异步I/O,事件驱动的特点，它非常适合编写websocket方面的应用。
# 用socket.io和express打造聊天室
## 下载socket.io
``` bash
$ npm install socket.io
```
## 服务器端代码
``` javascript
//导入express并实例化一个app
var express=require('express');
var app=express();
//socket.io需要原生node的http模块的支持
var http=require('http');
var server = http.createServer(app);
var io = require('socket.io')(server);
server.listen(3000);
//监听客户端的连接，并将连接传入socket
io.on('connection', function (socket) {
		//当前连接的socket监听客户端的名为'c2s'的数据发送，并将接收到的数据传入msg
        socket.on('c2s',function(msg){
        		//向所有的当前在线的客户端发送一个名为's2c'的广播
                io.sockets.emit('s2c', msg);
        })
});
```

## 客户端代码
``` html
<!DOCTYPE html>
<html>
  <head>
    <title>聊天| 聊天室</title>
  </head>
  <body>
	<h1>欢迎你，<span id='username'>{{username}}</span></h1>
		<input type="text" name="content" id="content">
		<button id="submit">发布</button>
		<ul id="ul">
		</ul>
	<script type="text/javascript" src='/socket.io/socket.io.js'></script>
	<script type="text/javascript" src='/public/javascripts/plugin/jquery-1.12.4.js'></script>
	<script type="text/javascript">
        $(function () {
        	//创建一个sokeet对象，io()是上面的socket.io.js暴露给我们的一个函数
			var socket=io.connect('http://localhost');
			//点击提交触发事件
            $("#submit").click(function (){
            	//判断输入框中是否有内容
            	if($('#content').val()){
            		//向服务器发送一个名为'c2s'的消息，后面的一个对象为消息的内容
            		socket.emit("c2s",{user:$("#username").text(),content:$('#content').val()});
            		//清空输入框
                	$("#content").val('');
            	}else{
            		//如果输入框为空，则什么也不做
            	} 
            });
            //按下enter键触发事件
            $("#content").keydown(function (e) {
                    if((e.keyCode===13)&&($('#content').val()){
                            socket.emit("c2s",{user:$("#username").text(),content:$('#content').val()});
                            $("#content").val(" ");
					}else{
						//啥也不做
					}
            });
            //监听服务器发出的's2c'消息，消息内容传入msg        	
            socket.on('s2c',function(msg){
            	//利用jQuery将消息渲染到页面上
                  $("#ul").prepend("<li><b>"+msg.user+"</b>说"+msg.content+"</li>")
            });
   		 });
	</script>
  </body>
</html>
```
当我们的服务器跑起来之后,我们可以直接在浏览器中访问这样一个地址：127.0.0.1:3000/socket.io/socket.io.js,这时我们会发现这一个url已经被劫持了，页面中出现了一个js脚本。我们再回头看一看我们的项目文件夹中，并没有出现socket.io.js这样一个脚本。这个js文件实际放在了服务器端的node_modules文件夹中，在请求这个文件时会重定向，因此不要诧异服务器端不存在这个文件但为什么还能正常工作。其实你可以把服务器端的socket.io.js这个文件拷贝到本地，使它成为客户端的js文件，这样就不用每次都向服务器请求这个js文件，以增强稳定性。无论如何我们必须在客户端引用这个脚本才能使用socket.io的功能。从上面的代码看，群聊聊天室的原理是:某一个客户端需要发言，他就把自己的发言内容先发送到聊天室的服务器，然后由服务器对所有连接到服务器的客户端进行广播，这样其他用户就可以看到你的消息，这也就实现了群聊。

## socket.IO的使用

从上面的聊天室的例子就可以发现，socket.io在使用上的核心其实就是两个函数emit()和on().
``` javascript
emit()
```
用来发射一个事件或者说触发一个事件，第一个参数为事件名，第二个参数为要发送的数据，第三个参数为回调函数（一般省略，如需对方接受到信息后立即得到确认时，则需要用到回调函数）。

``` javascript
on()
```
用来监听一个 emit 发射的事件，第一个参数为要监听的事件名，第二个参数为一个匿名函数用来接收对方发来的数据，该匿名函数的第一个参数为接收的数据，若有第二个参数，则为要返回的函数。

### 服务器端
#### 监听客户端的链接与断开
``` javascript
//监听客户端连接,回调函数会传递本次连接的socket
io.on(‘connection’,function(socket));
//监听客户端的断开连接,这里会传入"transport close"这个字符串到c这个形参中
socket.on('disconnect',function(c))
```
#### 发送消息
``` javascript
//给所有的客户端广播消息(包含发送消息过来触发这个函数的那个客户端)
io.sockets.emit(‘String’,data);
或　io.emit('String',data);
//给客户端广播消息(除了该socket所属于的那个客户端)
socket.boradcast.emit('String',data)
//给指定的客户端发送消息
io.sockets.socket(socketid).emit(‘String’, data);
//给该socket的客户端发送消息
socket.emit(‘String’, data);
```
#### 监听客户端的事件
``` javascript
//监听该socket所属于的客户端的String消息，这里的String是自定义的消息名称
socket.on('String',function(data))
```
#### 分组
``` javascript
io.on('connection',function(socket){
	socket.on('group1',function(data){
	socket.join('group1')
	});
	socket.on('group2',function(data){
	socket.join('group2')
	});	
});
```
如上面的代码，当一个客户端链接进来之后，并且发送了一个名为group1的事件消息，那么在服务器端，它与服务器的socket就通过socket.join(‘group1’)加入了一个名为group1的分组中，客户端的代码可以这样写：
``` javascript
<script type='text/javascript' src='/socket.io/socket.io.js'></script>
/**
*创建与服务器的连接，并且返回一个socket对象，这里由于是服务器和客户端到是在本地
*所以在connect()中可以不用传递值，如果是在生产环境这里就需要传值，如：
* socket.connect('localhost:3000')
*/
var socket=io.connect();
/**
*发送一个名为group1的消息，就可以在服务器的处理中被加入group1组
*/
socket.emit('group1',{});
```
一个用户可以存在与多个分组中
#### 踢出分组
``` javascript
socket.leave(data.room)
```
#### 向一个组广播消息
``` javascript
//向一个组广播消息(发送者无法收到消息)，并且这一方法允许当前socket不在这一分组中
socket.broadcast.to('your group name').emit('broadcast group message');
//向一个组广播消息(包括发送者都能收到消息),发送者必须在分组中
io.sockets.in('your group name').emit('broadcast romm message')
```
#### 获取连接的客户端socket
``` javascript
io.sockets.clients().forEach(function(socket){
	//可以对其进行操作
})
```
#### 获取分组信息
``` javascript
//获取所有的组别信息
io.sockets.manager.rooms
//获取此socketid进入的组别信息
io.sockets.manager.roomClients[socket.id]
//获取某一个房间中的客户端，返回所有在此房间的socket实例
io.sockets.clients('某组名')
```
### 客户端
#### 建立socket链接
``` javascript
var socket=io('http://<hostname>:<port>');
或者
var socket=io.connect('http://<hostname>:<port>')
```
如果是在本机做测试，客户端，服务器都在同一台主机上，则上面两个连接方式
的括号中可以不用写东西

#### 监听服务器的消息
``` javascript
socket.on('msg',function(data){
	console.log(data);
	//向服务器发送消息
	socket.emit('msg_c2s',{});
})；
```
#### 监听socket断开与重连
``` javascript
socket.on('disconnect',function(){
	console.log('与服务器断开链接')；
});
socket.on('reconnect',function(){
	console.log('重新链接到服务器')
});
```
上述的’disconnect’,’reconnect’是socket.io默认支持的可以被客户端监听的事件，像这样的事件有：
connect：连接成功
connecting：正在连接
disconnect：断开连接
connect_failed：连接失败
error：错误发生，并且无法被其他事件类型所处理
message：同服务器端message事件
anything：同服务器端anything事件
reconnect_failed：重连失败
reconnect：成功重连
reconnecting：正在重连
当第一次连接时，事件触发顺序为：connecting->connect；当失去连接时，事件触发顺序为：disconnect->reconnecting（可能进行多次）->connecting->reconnect->connect
# 我的demo

[demo](https://github.com/Andrewpqc/simpleChatroom)