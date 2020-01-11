---
title: Python的Socket练习——伪SSH
comments: true
date: 2017-09-01 08:21:36
updated: 2017-11-15 08:21:36
tags: [python socket]
categories: Python
permalink:
---
这一篇博客介绍一下我最近用python中的socket模块写的一个类似于ssh的小工具，虽说是一个工具，但是比那个真的ssh还是弱了不少，基本上也没什么使用价值。但是学习价值还是有的。主要的学习点是网络传输中的数据包粘包问题的解决方案，和发送过去的数据与源数据的md5校验以及python的os，socket模块的简单使用。
# 实现的功能
远程操作主机，实现系统中的部分命令的执行并返回执行的输出，上传本地文件到服务器运行的目录，从服务器运行目录下载文件到本地。
## 具体实现
### 上传/下载文件(过程差不多)
#### 客户端代码
``` python
import socket,json,hashlib,os
HOST = '127.0.0.1'
PORT = 8005
client = socket.socket()
client.connect((HOST, PORT))
while True:
    cmd = input(">>>").strip()
    if len(cmd)==0:
        continue
　  elif cmd.startswith('get'):
    client.send(cmd.encode("utf-8"))
    dateHeader_s2c = json.loads(client.recv(1024).decode('utf-8'))
    if dateHeader_s2c['status']=='Y':
        client.send(b'ok')
        md5 = hashlib.md5()
        filename = dateHeader_s2c['filename']
        accept_length=0
        response_length=dateHeader_s2c['size']
        while accept_length < response_length:
            with open(filename + ".s2c", 'wb') as f:
                accept_temp = client.recv(1024)
                md5.update(accept_temp)
                f.write(accept_temp)
                accept_length += len(accept_temp)
        else:
            client.send(b'ok')
            accept_md5 = client.recv(1024).decode('utf-8')
            if accept_md5 == md5.hexdigest():
                print('下载成功！')
            else:
                os.popen('rm %s' % filename + ".s2c") #删除已经下载的破碎文件
                print('文件破碎！请重新下载')
    else:
        print('请输入一个正确的文件名')
```
#### 服务器代码
``` python
import socket,os,hashlib,json
from . import server_config
server=socket.socket()
server.bind((server_config.HOST,server_config.PORT))
server.listen(5)
print ('Server start at: %s:%s' %(server_config.HOST,server_config.PORT))
print ('wait for connection...')
while True:
    conn, addr = server.accept()
    print ('Connected by ', addr)
    while True:
        #接收到客户端输入的命令
        cmd = conn.recv(1024).decode().strip()
        if len(cmd)==0:
            print('客户端断开链接．．．')
            break
        #响应客户端的下载文件请求
        if cmd.startswith('get'):
            dataHeader_s2c = {}
            fileName=cmd.split(' ')[-1]
            if os.path.isfile(fileName):
                dataHeader_s2c['status']='Y'
                dataHeader_s2c['filename']=fileName
                dataHeader_s2c['size']=os.stat(fileName).st_size #文件大小
                conn.send((json.dumps(dataHeader_s2c)).encode('utf-8'))
                conn.recv(1024) #防止粘包
                md5=hashlib.md5()
                with open(fileName,'rb') as f:
                    for line in f:
                        md5.update(line)
                        conn.send(line)
                    else:
                        conn.recv(1024)#防止粘包
                        conn.send(md5.hexdigest().encode('utf-8'))
            else:
                dataHeader_s2c['status']='N'
                conn.send((json.dumps(dataHeader_s2c)).encode('utf-8'))
```
#### 需要注意的点

    命令处理：客户端首先接收用户输入的命令，如果输入为空则跳出本轮循环，进入下一轮循环，重新等待用户的输入，如果输入不为空则将输入的内容发送给服务器。服务器接收到数据之后也会判断这个内容是否为空，如果内容为空就会认为客户端已经断开链接，从而直接跳出这一层的循环，等待并处理下一个接入的客户端。可能大家会有疑问，我们在客户端已经检查了用户的输入，可以确保发送给服务器的数据不为空，在服务器端又做一遍这样的检查，有什么用呢？其实python的socket模块做了特殊的设计，当客户端进程退出之前，他会自动向服务器发送一个空消息。所以服务器就是依据这一点来判断客户端是否断开的。
    编码与解码：在python3中socket的发送和接收的都是字节，所以我们在发送数据之前要将我们的数据编码（encode）成字节的形式才能发送，同理我们在客户端接收到的数据也是字节形式，所以我们要将其解码（decode）成字符串.
    头数据：为了方便客户端接收数据（如什么时候数据可以接收完等），服务器端需要在真正发送数据之前先发送一些与客户端需要的文件相关的头数据，如文件的大小等信息，这里由于客户端并不是只做文件的下载处理，所以这里的头数据还包含了一些标志数据。这里是先将这些数据放入一个字典，然后序列化成json数据，到客户端之后在进行反序列化。
    粘包问题：有时候我们需要连续几次发送不同的数据，在客户端也要连续几次来分别接收这些信息。但是这个时候事情往往不是按照我们想象的那样进行的，可能服务器的连续几次发送的信息被客户端的一次接收给接收到了，这个问题叫做粘包。这显然不是我们希望得到的，我们需要解决这个问题。解决这个问题的思路就是将这些连续发送的数据让它不连续，比如我们让一次发送与下一次发送中间休息（sleep）几秒.但是这种方法显然是不对的，这会极大的降低数据传输的速度。实际上我们解决的方法是在两次连续的发送中间来一次接收，或者让接收方在接收的时候恰恰接收当次发送的那么多数据，我们上面采用的都是前一种方法。
    md5比对：为了判断在数据传输的过程中是否发生错误，我们还需要对发送过去的数据和原数据进行一个md5值比对，如果不一致我们就删掉破损的文件，提示用户重新下载。

### 操作远程主机命令
#### 客户端代码
``` python
else:
    client.send(cmd.encode("utf-8"))
    dateHeader_s2c = json.loads(client.recv(1024).decode('utf-8'))
    response_length=dateHeader_s2c['size']
    accept_length=0
    accept_content=b''
    client.send(b'ok')
    while accept_length<response_length:
        accept_temp = client.recv(1024)
        accept_length+=len(accept_temp)
        accept_content+=accept_temp
    else:
        print(accept_content.decode('utf-8'))
```
#### 服务端代码
```python
else:
    dataHeader_s2c = {}
    res_content = os.popen(cmd).read()
    if len(res_content) == 0:
        res_content = "命令没有返回内容"
    dataHeader_s2c['type'] = 'cmd'
    dataHeader_s2c['size'] = len(res_content)
    conn.send(json.dumps(dataHeader_s2c).encode('utf-8'))
    conn.recv(1024)
    conn.send(res_content.encode("utf-8"))
```

#### 需要注意的点
其实这个过程和前面的文件上传下载差不多，上面要注意的点，这里都要注意这里的主要实现其实就是python中的os模块中的操作操作系统的接口res_content = os.popen(cmd).read()，这一popen需要传输一个系统的命令（字符串），同时返回的是一个类似文件的对象，我们read它之后得到的就是命令返回的内容（字符串）

这里还有一些命令是运行不了的，或者说可以执行但无法返回命令的输出。


# 我的demo
[demo](https://github.com/Andrewpqc/FakeSsh)
