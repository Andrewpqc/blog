---
title: 木犀后端分享——网络爬虫
comments: true
date: 2017-11-26 12:58:51
updated: 2017-11-26 12:58:51
tags: [urllib,urllib3,requests,bs4,xpath,phantomjs,selenium]
categories: Python
permalink:
---

# 网络基础知识
## 先来两张图撑场子！

![计算机网络体系结构](/images/网络体系结构.png)

![信息处理流程](/images/数据处理流程.png)

计算机网络体系结构是分层的，分层的目的就是为了各司其职，每一层协议就负责处理好自己的工作，而不用去担心别人的事情。
## TCP/IP协议
>从字面意义上讲，有人可能会认为TCP/IP是指TCP和IP两种协议。实际生活当中有时也确实就是指这两种协议。然而在很多情况下，它只是利用 IP 进行通信时所必须用到的协议群的统称。具体来说，IP或ICMP、TCP或UDP、TELNET或FTP、以及HTTP等都属于TCP/IP协议。他们与TCP或IP的关系紧密，是互联网必不可少的组成部分。TCP/IP 一词泛指这些协议，因此，有时也称 TCP/IP为网际协议群。

就我本人的理解，TCP/IP就代表着一大堆的协议，这些协议就是具体负责网络中信息的传递。他们确保了数据可以被准确无误的传输到目的地。既传的远又传的准。也就是说，TCP/IP就帮你**完美的解决了数据传输的问题**，其他的协议就不需要再考虑数据传不远，传不到的问题了。

更多详细的内容看－>[TCP/IP协议](http://www.jianshu.com/p/9f3e879a4c9c)，上面的引用也出自此处。


## HTTP协议
>HTTP协议（HyperText Transfer Protocol，超文本传输协议）是用于从WWW服务器传输超文本到本地浏览器的传输协议。它可以使浏览器更加高效，使网络传输减少。它不仅保证计算机正确快速地传输超文本文档，还确定传输文档中的哪一部分，以及哪部分内容首先显示(如文本先于图形)等。
>HTTP是客户端浏览器或其他程序与Web服务器之间的应用层通信协议。在Internet上的Web服务器上存放的都是超文本信息，客户机需要通过HTTP协议传输所要访问的超文本信息。HTTP包含命令和传输信息，不仅可用于Web访问，也可以用于其他因特网/内联网应用系统之间的通信，从而实现各类应用资源超媒体访问的集成。

HTTP协议是构建在TCP/IP协议之上的应用层的协议，它不必再关心数据的是否可以准确无误的到达目的地的问题，它关心和解决的是**在什么时候传什么数据给谁**的问题。

更多详细内容请看－>[HTTP协议](http://www.jianshu.com/p/6e9e4156ece3),上面的引用也出自此处。
# 网络爬虫简介
**网络爬虫**:又被称为网页蜘蛛，网络机器人等。是一种按照一定的规则，**自动**地抓取万维网信息的程序或者脚本。大家可以理解为在网络上爬行的一只蜘蛛。互联网就比作一张大网，而爬虫便是在这张网上爬来爬去的蜘蛛，如果它遇到资源，那么它就会抓取下来。想抓取什么？这个由你来控制它。爬虫技术是数据挖掘,测试技术的重要组成部分，是搜索引擎技术的核心。

说白了,爬虫技术就是一种**自动化**去请求下载并处理网络信息资源(html页面,css,js,pdf,word,excel，图片，音视频等)的技术。在实际中处理的比较多的网络信息资源是html页面，然后获取页面中有价值的文字信息。

要学好网络爬虫我们就需要解决两个问题:怎样下载？怎样处理？

# 下载

## 去哪里下载？
网络信息资源分布在服务提供者的服务器上，我们要去下载它，第一个要解决的问题就是要知道它在哪里。
### URL,URI,URN是什么鬼？
**URL**:(Uniform Resource Locator,统一资源定位符)是一个Web地址，用来在Web上定位一个文档，或者调用一个CGI程序来为客户端生成一个文档。
**URI**:(Uniform Resource Identifier,统一资源标识符)是URL的超集，可以应对将来可能出现的标识符命名约定。一个URL是一个简单的URI,它使用已有的协议或方案(http,ftp等)作为地址的一部分。
**URN**:(Uniform Resource Name,统一资源名称)URN用来描述非URL的URI,只作为可能会用到的XML标识符。

`URI = URL + URN`
现在唯一使用的URI只有URL,而很少听到URI和URN.

我们在浏览器的地址栏里输入的网站地址就是URL。就像每家每户都有一个门牌地址一样，每个网页也都有一个Internet地址。当你在浏览器的地址框中输入一个URL或是单击一个超级链接时，URL就确定了要浏览的地址。浏览器通过超文本传输协议(HTTP)，将Web服务器上站点的网页代码提取出来，并翻译成漂亮的网页。
### URL的组成
URL使用这种格式：
`protocol_schema://net_location/path;params?query#frag`

| URL组件 | 描述 |
| :------------------: | :---------------------: |
| protocol_schema | 网络协议或下载方案 |
| net_location | 服务器所在地(也许含有用户信息) |
| path | 使用'/'分割的文件的路径或CGI应用的路径 |
| params | 可选参数 |
| query | 连接符'&'分割的一系列键值对 |
| frag | 指定文档内特定锚的部分 |

net_location可以进一步拆分成多个组件:
`user:password@host:port`

| net_location组件 | 描述 |
| :------------------: | :--------------------: |
| user | 用户名 |
| password | 用户密码 |
| host | 运行web服务器的节点地址(必须的) |
| port | 端口号(如果没有则默认为80) |

在net_location的四个组件中host最为重要，port只有在web服务器运行其他非默认端口号时才会使用，用户名和密码只有在使用FTP连接是才有可能用到，而即便是FTP，大多数的连接都是匿名的，这时不需要用户名和密码。
## 用什么下载？
### urllib
urllib是python标准库提供的一个高级的Web通信库，支持基本的Web协议，如Http,Ftp,Gopher,同时也支持对本地文件的访问。具体来说urllib模块的功能就是利用这些协议从因特网，局域网，本地主机上下载数据。
#### 导入
```python
import urllib.request
from urllib import request

#下面的这种导入是错误的：
#import urllib
```
#### 下载并保存你的第一个网页
``` python
from urllib import request
file=request.urlopen("http://www.baidu.com")
myfirstpage=file.read()
with open("myfirstpage.html","w") as f:
    f.write(myfirstpage)
```
urlopen()传入一个URL string,会打开这个string所指向的url,下载对应的网页，返回该网页的文件对象(这里我把它赋值给了file变量)。如果没有给定协议或者下载方案，或者传入‘file’方案，urlopen()会打开一个本地文件。

urlopen()返回的文件对象(即上面代码中的file)还有一些实用方法：

| urlopen()返回的对象的方法 | 描述 |
| :----------------------: | :-------------------------: |
| file.read([bytes]) | 从file中读出所有或者bytes个字节，以字符串返回 |
| file.readline() | 从file中读取一行，以字符串返回 |
| file.readlines() | 从file中读出所有行，以列表返回，每一行作为列表中的一项 |
| file.close() | 关闭file的url连接 |
| file.fileno() | 返回file的文件句柄 |
| file.info() | 获得file的MIME头文件 |
| file.geturl() | 返回当前请求的url |
| file.getcode() | 返回响应的状态码 |

例子：
```python
from urllib import request
f=request.urlopen('http://www.baidu.com')
print(f.fileno())
print(f.getcode())
print(f.geturl())
print(f.info())
f.close()
```
输出如下：
```
3
200
http://www.baidu.com
Date: Wed, 29 Nov 2017 03:58:52 GMT
Content-Type: text/html; charset=utf-8
Transfer-Encoding: chunked
Connection: Close
Vary: Accept-Encoding
Set-Cookie: BAIDUID=B06CDC6F062BA8B6D50F401E830F6B5A:FG=1; expires=Thu, 31-Dec-37 23:55:55 GMT; max-age=2147483647; path=/; domain=.baidu.com
Set-Cookie: BIDUPSID=B06CDC6F062BA8B6D50F401E830F6B5A; expires=Thu, 31-Dec-37 23:55:55 GMT; max-age=2147483647; path=/; domain=.baidu.com
Set-Cookie: PSTM=1511927932; expires=Thu, 31-Dec-37 23:55:55 GMT; max-age=2147483647; path=/; domain=.baidu.com
Set-Cookie: BDSVRTM=0; path=/
Set-Cookie: BD_HOME=0; path=/
Set-Cookie: H_PS_PSSID=1440_19036_21116_18560_17001_25178_25145_22157; path=/; domain=.baidu.com
P3P: CP=" OTI DSP COR IVA OUR IND COM "
Cache-Control: private
Cxy_all: baidu+14ee1d71cc2cf84997fda3c20bcc1684
Expires: Wed, 29 Nov 2017 03:58:18 GMT
X-Powered-By: HPHP
Server: BWS/1.1
X-UA-Compatible: IE=Edge,chrome=1
BDPAGETYPE: 1
BDQID: 0xe0db52f000003afc
BDUSERID: 0
```

#### 更优雅的下载并保存
``` python
from urllib import request
#下载并保存
filename,mime_hdrs=request.urlretrieve(url='http://www.douban.com',filename="mysecondpage.html")
#清除缓存
request.urlcleanup()
```
urlretrieve()函数只需要传入资源对应的url和要将其保存在本地的位置，就可以实现下载并保存。这个方法不仅可以实现html页面的下载保存，对于所有网络信息资源都可以，包括图片，视音频等。

urlretrieve()执行的过程中会产生一些缓存，如果我们想要清除这些缓存信息，可以使用urlcleanup()进行清除。

#### url的编码
一般来说，必须要对某些不能打印的或者不被web服务器作为有效URL接收的特殊字符串进行转换。在一个URL中，逗号，下划线，句号，斜线，字母，数字这类符号不需要转化，其他的均需要转化。转换过程中那些url不能使用的字符前面会被加上%,同时转换成16进制，例如"="将被转换成'%3d','3d'就是'='的ASCLL码的16进制。urllib.request中提供的url转换的api就三个`quote()`,`unquote()`,`unquote_to_bytes()`，后两个做相反的工作。
``` python
from urllib import request
sourceUrl="http://www.baidu.com/?key1=hhh&key2=a b"
quoteUrl=request.quote(sourceUrl)
# http%3A//www.baidu.com/%3Fkey1%3Dhhh%26key2%3Da%20b
unquoteUrl=request.unquote(quoteUrl)
# http://www.baidu.com/?key1=hhh&key2=a b
unquotetobyte=request.unquote_to_bytes(quoteUrl)
# b'http://www.baidu.com/?key1=hhh&key2=a b'
```
Tips：如果你在写爬虫时遇到类似下面的保错：
UnicodeEncodeError: 'ascii' codec can't encode characters in position 14-15: ordinal not in range(128)
基本上就说明你的url需要转换了。如果你的url有中文，那就一定错了。一般来说，我们在浏览器地址栏中复制url时，浏览器就已经帮你做好转换了，所以一般不会遇到这种问题。但是，在自己构造url时就需要注意了。
#### url参数的编码
``` python
from urllib.parse import urlencode
d={'a':1,'b':"h?h",'c':"哈哈"}
print(urlencode(d))
#输出　c=%E5%93%88%E5%93%88&a=1&b=h%3Fh
```
urllib.parse.urlencode()可以从一个字典构造出查询字符串，并且自动做了url编码。

#### urllib请求头的添加
在urllib中添加请求头有两种方式，使用`build_opener()`,使用`add_header()`
1.使用build_opener()
``` python
from urllib import request
headers=[("Accept","text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8"),
    ("User-Agent","Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.89 Safari/537.36")]
opener=request.build_opener()
opener.addheader=headers
data=opener.open("http://www.baidu.com").read()
```
2.使用add_header()
``` python
from urllib import request
headers=[("Accept","text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8"),
    ("User-Agent","Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.89 Safari/537.36")]
req=request.Request("http://www.baidu.com")
req.add_header(headers)
data=request.urlopen(req).read()
```
#### 代理服务器的设置
``` python
from urllib import request
proxy=request.ProxyHandler({"http":"localhost:9999"})
opener=request.build_opener(proxy,request.HTTPHandler)
request.install_opener(opener)
data=request.urlopen("http://www.baidu.com").read()
```

### 神器——urllib3
[urllib3](https://urllib3.readthedocs.io/en/latest/index.html)
>urllib3 is a powerful, sanity-friendly HTTP client for Python. Much of the Python ecosystem already uses urllib3 and you should too. urllib3 brings many critical features that are missing from the Python standard libraries:
- Thread safety.
- Connection pooling.
- Client-side SSL/TLS verification.
- File uploads with multipart encoding.
- Helpers for retrying requests and dealing with HTTP redirects.
- Support for gzip and deflate encoding.
- Proxy support for HTTP and SOCKS.
- 100% test coverage.


#### 基本使用
##### 第一个urllib3例子
``` python
import urllib3
http=urllib3.PoolManager()
response=http.request('GET',"http://www.baidu.com")
print(response.headers)
print(response.status)
print(response.data.decode("utf-8"))
```
输出：
``` 
HTTPHeaderDict({'Set-Cookie': 'BAIDUID=DAB4C909697193545FA8395524CF0963:FG=1; expires=Thu, 31-Dec-37 23:55:55 GMT; max-age=2147483647; path=/; domain=.baidu.com, BIDUPSID=DAB4C909697193545FA8395524CF0963; expires=Thu, 31-Dec-37 23:55:55 GMT; max-age=2147483647; path=/; domain=.baidu.com, PSTM=1511944442; expires=Thu, 31-Dec-37 23:55:55 GMT; max-age=2147483647; path=/; domain=.baidu.com', 'Server': 'BWS/1.1', 'Cache-control': 'no-cache', 'Last-Modified': 'Wed, 22 Nov 2017 02:22:00 GMT', 'P3P': 'CP=" OTI DSP COR IVA OUR IND COM "', 'Accept-Ranges': 'bytes', 'Date': 'Wed, 29 Nov 2017 08:34:02 GMT', 'Content-Type': 'text/html', 'X-UA-Compatible': 'IE=Edge,chrome=1', 'Connection': 'Keep-Alive', 'Content-Length': '14613', 'Pragma': 'no-cache', 'Vary': 'Accept-Encoding'})
200
<!DOCTYPE html><!--STATUS OK-->
<html>
<head>
......此处省略部分输出
```
##### 发起请求
如上面的例子所示，你需要通过一个PoolManager实例来发起请求，这个实例对象会处理所有的和连接池，线程安全有关的细节。然后在这个PoolManager对象上调用request()方法，就可以发起请求。request()方法返回一个HttpResponse对象。通过该对象可以获取响应的内容。
##### 接收数据
request()方法返回的HttpResponse对象有status,data和headers三个属性，分别来获取响应的状态码，响应的数据，和响应头。其中data属性获取的是bytes类型的数据，我们要使用它就要先对它进行解码(decode).

##### 请求头的设置
``` python
import urllib3
http=urllib3.PoolManager()
response=http.request('GET',"http://www.baidu.com",headers={'X-Something': 'value'})
```
请求头的信息在request()方法的headers参数中设置。

##### 查询参数的设置
对于GET,HEAD,DELETE请求，你可以直接将参数作为一个字典传个request()的fields参数，就像下面这样：
``` python
import urllib3
http=urllib3.PoolManager()
response=http.request('GET',"http://www.baidu.com",fields={'arg': 'value'})
```

但对于POST和PUT请求，你需要手动的编码查询参数：
``` python
import urllib3
from urllib.parse import urlencode
http=urllib3.PoolManager()
encoded_args = urlencode({'arg': 'value'})
url = 'http://httpbin.org/post?' + encoded_args
response = http.request('POST', url)
```

##### 提交表单数据
就表单数据而言，主要使用的是POST,PUT两种提交方式，urllib3会自动的编码fields参数提供的字典。
``` python
...
response = http.request('POST','http://httpbin.org/post',fields={'field': 'value'})
```
##### 发送JSON给服务器
``` python
import json
import urllib3
http=urllib3.PoolManager()
data = {'attribute': 'value'}
encoded_data = json.dumps(data).encode('utf-8')
r = http.request('POST','http://httpbin.org/post',body=encoded_data,headers={'Content-Type':'application/json'})
```
将需要发送给服务器的json数据编码后传递给request()的body参数，然后在请求头中设置Content-Type字段为application/json.

##### 发送文件或二进制数据
``` python
with open('example.txt') as fp:
    file_data = fp.read()
r = http.request('POST','http://httpbin.org/post',fields={'filefield': ('example.txt', file_data,'text/plain')})
```
在fields中的filefield对应的元组中，文件名的指定并不是严格必须的。为了匹配浏览器的行为，强烈建议在这个元组中传递第三个参数来指明文件的MIME类型。

对于原始二进制数据(raw binary data)的发送可以简单的指定body参数。同样为了匹配浏览器的行为，建议要设置请求头中的Content-Type字段。
``` python
with open('example.jpg', 'rb') as fp:
     binary_data = fp.read()
r = http.request('POST','http://httpbin.org/post',body=binary_data,headers={'Content-Type': 'image/jpeg'})
```

##### 设置证书验证
官方强烈建议我们总是使用[SSL](https://zh.wikipedia.org/wiki/傳輸層安全性協定)证书验证。这样能够保证我们与服务器之间的通信的安全。默认情况下，urllib3不验证HTTPS请求。
为了启用验证你需要一些根证书。最简单和最可靠的方法是使用**certifi**包。这个包提供了Mozilla的根证书包。在使用之前需要安装：
``` bash
$ pip3 install certifi
```
如果你在安装urllib3时使用了下面的命令，那么你的系统中就已经安装了certifi:
``` bash
$ pip3 install urllib3[secure]
```
这条命令在安装urllib3的同时会安装certifi.
如果你使用的是python2可能会需要其他的一些包。

一旦你安装了证书验证所需要的依赖，你在创建PoolManager对象的时候，就可以传入相应的参数，在请求的时候来启用证书验证。
``` python
import certifi
import urllib3
http=urllib3.PoolManager(cert_reqs="CERT_REQUIRED",ca_certs=certifi.where())
#下面请求https://google.com
http.request("GET",'https://google.com')#没有任何错误

#下面请求https://expired.badssl.com
http.request("GET",'https://expired.badssl.com')
#抛出urllib3.exceptions.SSLError：[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed (_ssl.c:645)的错误，说明该网站没有配置安全证书。
```
在实例化PoolManager对象时做出上述配置后，PoolManager对象就会自动处理证书验证，如果验证失败就会抛出urllib3.exceptions.SSLError的异常。

如果需要的话，你也可以使用操作系统提供的证书，只需要将上述的ca_certs参数的值指定为你的系统中安全证书包所在的绝对路径即可。例如，在大多数的Linux操作系统中，安全证书存储在`/etc/ssl/certs/ca-certificates.crt`中。其他的操作系统可能会有些许不同。上述验证改为使用操作系统提供的证书后如下：
``` python
import certifi
import urllib3
http=urllib3.PoolManager(cert_reqs="CERT_REQUIRED",ca_certs="/etc/ssl/certs/ca-certificates.crt"
#下面请求https://google.com
http.request("GET",'https://google.com')#没有任何错误

#下面请求https://expired.badssl.com
http.request("GET",'https://expired.badssl.com')
#抛出urllib3.exceptions.SSLError：[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed (_ssl.c:645)的错误，说明该网站没有配置安全证书。
```
[在python2中为urllib3应用添加证书验证功能](https://urllib3.readthedocs.io/en/latest/user-guide.html#certificate-verification)

##### 设置超时时间
设置超时时间可以让你控制你的请求最长等待的时间，超过你设置的时间服务器还没有响应，那么就抛出异常。
简单的,你可以给PoolManager对象的request()方法的timeout参数制定一个浮点数来制定本次请求的超时时间。就像下面这样：
``` python
import urllib3
http=urllib3.PoolManager()
http.request("GET","http://www.baidu.com",timeout=2.5)
```
如果你需要更加细粒度的控制超时时间，你可以使用一个TimeOut实例来分别指定连接超时和读取超时：
``` python
import urllib3
http=urllib3.PoolManager()
#只限制连接超时时间为2.0秒，对读取时间不做限制
http.request("GET","http://www.google.com",
        timeout=urllib3.Timeout(connect=2.0))

#限制连接超时时间为1.0秒，读取超时时间为2.0秒
http.request("GET","http://www.baidu.com",
        timeout=urllib3.Timeout(connect=1.0,read=2.0))
```
如果你需要对所有的请求做同样的超时设置，那么你可以直接在PoolManager的层面上做配置：
``` python
import urllib3
http1=urllib3.PoolManager(timeout=3.0)
http2=urllib3.PoolManager(timeout=
    urllib3.Timeout(connect=1.0,read=2.0))
```
这样某一个PoolManager对象的所有的request()使用的默认超时设置就是它的PoolManager的超时设置。**当然你仍然可以在request()方法中重载这个配置。**

##### 设置重试次数
urllib3在默认情况下，再一次请求中，能够自动的重试３次，自动跟进３次重定向。你通过request()方法的retries参数控制重试：
``` python
#下面的情况是重试次数为１０，最多跟进３次重定向
http.request("GET","http://www.bidu.com",retries=10)
```
如果你要禁用重试**和**重定向跟进的功能的话，只需要将retries指定为False:
``` python
#禁用重定向跟进和重试功能
http.request("GET","http://www.google.com",retries=False)
```

如果你要禁用重定向跟进，但是保留默认的３次重试的话，你只需要将request()的redirect参数指定为False：
``` python
#禁用重定向的跟进功能，保留了３次的重试功能
http.request("GET","http://www.baidu.com",redirect=False)
```
同设置超时时间一样，如果你需要对重试次数和重定向跟进次数做更加细粒度的控制的话，你需要使用一个Retry实例，例如下面的例子就是最多做三次重试，两次重定向跟进：
``` python
http.request("GET","http://www.google.com",retires=
    urllib3.Retry(3,redirect=2))
```
你可以通过下面的配置，来禁掉应重定向次数过多而造成的抛出错误这一行为，转而返回302的状态码：
``` python
r=http.request("GET","http://www.baidu.com",
    retries=urllib3.Retry(redirect=2,raise_on_redirect=False))
print(r.status)
#这里如果重定向到次数超过两次的话，程序不会抛出错误，而是状态码返回302
```
同样的，如果你想要为某一个PoolManager对象的所有request()配置同样的重试次数和重定向跟进次数的话，你可以在PoolManager层面上做配置：
``` python
http1=urllib3.PoolManager(retries=False)

http2=urllib3.PoolManager(retries=urllib3.Retry(5,redirect=2))
```
这样某一个PoolManager对象的所有的request()使用的默认重试次数和重定向跟进次数就是它的PoolManager的重定向次数和重定向跟进次数。**当然你仍然可以在request()方法中重载这个配置。**

##### 错误处理
``` python
try:
    http.request("GET","http://www.google.com",retries=False)
except urllib3.exceptions.NewConnectorError as f:
    print("Connection failed!",f)
```
异常处理在爬虫中非常重要!更多关于urllib3中的异常可以查看其源码或者[看这里](https://urllib3.readthedocs.io/en/latest/reference/index.html#module-urllib3.exceptions)

##### 日志
依靠标准库的logging,可以实现日志的记录：
``` python
logging.getLogger("urllib3").setLevel(logging.WARNING)
```

#### 进阶话题
##### 自定义池行为
PoolManager类自动帮你管理着ConnectionPool类的实例创建工作，一个ConnectionPool管理着发送给一个host的所有请求。默认情况下一个PoolManager最多管理10个ConnectionPool.如果在你的程序中需要向不止10个host同时发送请求的话，你可以更改以适量增加一个PoolManager最多可以管理的ConnectionPool数目，这样做可以提高urllib3的性能。但是同时这也会带来更多的内存和套接字消耗。更改的方法如下：
``` python
import urllib3
http=urllib3.PoolManager(num_pools=50)
```
这样就把一个PoolManager实例最多可以管理的ConnectionPool数目从默认的10改为了50。

同样的，一个ConnectionPool类管理着一个由多个HttpConnection实例组成的http连接池，每一个HttpConnection实例将会用于一个请求。当请求完成之后，连接就会返回到连接池中。默认情况下只有一个连接将会被保存以重用。如果你需要同时向一个host发送很多请求的话，你可以更改以适量增加一个连接池中将会被保存以重用的连接数目，这样有助于提高性能。更改的方法如下：
``` python
import urllib3
http=urllib3.PoolManager(maxsize=10)

#或者,使用下面这种方式单独实例化一个对goole.com这一host的连
#接池,并且限定连接池中保存以重用的连接数为１０
http=urllib3.HttpCoonnectionPool("google.com",maxsize=10)
```
默认情况下，对某一个host的一个新的请求发起了，如果此时这个host所对应的连接池中没有可用的连接，那么就会创建一个新的连接。然而如果此时连接池中被保存以重用的连接的数目不小于maxsize设定的值的话，这个新创建的连接将不会被保存以重用。也就是说，maxsize指定的数字不是决定一个连接池中最多可以存在的连接数目的多少，它仅仅指定了这个连接池中被保存以重用的连接数的最大值。但是，如果你指定了参数block=True的话，那么maxsize的值就也限制了某个host所对应的连接池中的最大连接个数。
``` python
http = urllib3.PoolManager(maxsize=10, block=True)
# Alternatively
http = urllib3.HTTPConnectionPool('google.com', maxsize=10, block=True)
```
这样的话，每一个新的请求将会被阻塞直到对应连接池中有一个连接可用为止。这样可以有效防止在多线程应用中，请求某一host的连接过于泛滥(多)的问题。

##### 流式处理大额响应
当我们请求的是一个大文件，比方说是一部电影的数据的话，我们就需要对响应的内容做流式处理：
``` python
import urllib3
http=urllib3.PoolManager()
r=http.request("GET","http://httpbin.org/bytes/1024",
    preload_content=False)
for chunk in r.stream(32):
    print(chunk)
r.release_conn()
```
在request()方法中，将preload_content参数设为False意味着urllib3将会流式处理响应的内容。request()方法返回的HTTPResponse对象的stream()方法可以让你对响应的内容做迭代。

当你使用了`preload_content=False`这一选项时，你最后应该调用HTTPResponse对象的release_conn()方法来释放本次连接，让其返回连接池以重用。

你也可以把这个HTTPResponse对象当做一个类文件对象，这允许你做缓冲处理。
``` python
r = http.request(
     'GET',
     'http://httpbin.org/bytes/1024',
     preload_content=False)
r.read(4)
```
直接对read()方法的调用将会阻塞，直到有更多的响应内容可用。
``` python
import io
reader=io.BufferedReader(r,8)
reader.read(4)
r.release_conn()
```
你可以利用这个类文件对象来做一些事情，比如用codecs来解码响应内容：
``` python
import codecs,json
reader=codecs.getreader("utf-8")
r=http.request("GET","http://httpbin.org/ip",preload_content=False)
data=json.load(reader(r))
r.release_conn()
```

##### 设置网络代理
你可以使用ProxyManager通过HTTP代理来传输你的请求：
``` python
import urllib3
proxy=urllib3.ProxyManager("http://localhost:3128")
proxy.request("GET","http://google.com/")
```
ProxyManager的用法和PoolManager是一样的

你也可以使用SOCKSProxyManager来连接到SOCK4或者SOCKS5代理。为了启用SOCKS代理，你需要安装[PySocks](https://pypi.python.org/pypi/PySocks)或者安装urllib3的socks扩展:
``` bash
$ pip3 install PySocks
#或者
$ pip3 install urllib3[socks]
```
一旦你安装了PySocks，你就可以在你的代码中使用SOCKSProxyManager:
``` python
from urllib3.contrib.socks import SOCKSProxyManager
proxy=SOCKSProxyManager('socks5://localhost:8889/')
proxy.request('GET', 'http://google.com/')
```

##### 其他
剩下几个主题可能大家不会遇到，这里就不再介绍了，丢个[链接](https://urllib3.readthedocs.io/en/latest/advanced-usage.html#custom-ssl-certificates-and-client-certificates)


### 神器－－requests
[requests](http://www.python-requests.org/en/master/): HTTP for Humans
requests是另外一个HTTP客户端编程的神器，它构建在urllib3的基础之上，不仅继承了urllib3的优秀特质，并且全面自动的支持HTTP/1.1请求，带持久 Cookie和Session等,对我们用户来讲，它还拥有更加人性化的API设计。






[Cookie处理](http://docs.python-requests.org/zh_CN/latest/user/quickstart.html#cookie)
## 反爬虫者与反反爬虫者之间的恩怨情仇
爬虫对于网站拥有者来说并不是一个令人高兴的存在，因为爬虫的肆意横行意味着自己的网站资料泄露，资源消耗，甚至是自己刻意隐藏在网站的隐私的内容也会泄露。面对这样的状况，作为网站的维护者或者拥有者，要么抵御爬虫，通过各种反爬虫的手段阻挡爬虫，要么顺从爬虫，自动提供可供爬虫使用的接口。事实上，大多数的网站既会采取一些必要的反爬虫措施，也会提供一些开放的api供开发者获取数据。但是绝大多数的开放平台所提供的api都有各种各样的限制：无法完全满足你的需求，需要收费等。所以很多人更愿意自己到网站上去爬取数据，在这种情况之下伟大的反爬虫运动与反反爬虫运动之间的斗争就开始了。

### 各种反爬虫手段及因对措施
#### User-Agent
`原理`：对于比较简陋的爬虫程序来讲一般没有设置请求头，而浏览器等客户端工具一般会自动帮我们加上请求头，那么服务器就可以检查客户端发来的请求的请求头中的某些字段是否存在从而识别出客户端是爬虫程序。一般情况下，服务器会检查请求头的User-Agent字段。

`应对`：这个应对的方法比较简单，我们只需要为请求添加请求头即可(主要是添加User-Agent字段)。这个过程在爬虫中叫做**浏览器伪装**。在这里给大家推荐一个简单实用的python包[fake-useragent](https://pypi.python.org/pypi/fake-useragent)，它可以方便的帮你生成User-Agent信息，这样你就不需要自己到处去复制了。

#### IP限制
`原理`：如果是个人编写的爬虫，没做特殊处理的话，IP是固定的，那么服务器发现某个IP请求的频率超过了某一阈值，就可以判断这个客户端是爬虫程序了，网站的管理或者运维人员，一般的处理方式就是暂时封掉该IP,那么也就是说这个IP发出的请求在短时间(一般是数小时)内不能再访问这个网站了，也就暂时挡住了爬虫。

`应对`：对于这种手段我们一般采取两种处理方式：设置延时和使用网络代理。**设置延时**很简单，就是让请求与请求之间停留一小段时间，让请求的频率降下来，这样就达不到请求频率的阈值，也就不会触发服务器的封IP行为了。使用**HTTP代理**之后，在服务器上显示的就是你的代理服务器的IP地址了，即使是封掉了IP封掉的也是代理服务器的地址，这时你换一个代理服务器就OK了。在实际中，我们会先准备大量的可用IP,从而建立一个IP池，每次请求都从IP池中任意选取一个IP去访问。
代理IP从[这儿](http://www.xicidaili.com/wn/)找,在选的时候大家尽量选择验证时间比较短，反应速度比较快的ip.

#### Ajax动态加载
`原理`：对于网站来说，使用Ajax动态加载技术可以提高网站的工作效率，提升用户体验，在用户需要某个数据时(某一特定条件发生时)才发送这些数据到客户端，然后通过js在不刷新整个页面的情况下将这些数据渲染到页面上。但对于数据采集者来说，这却带来了巨大的麻烦。使用传统的请求工具无法得到想要的完整的数据。下面给大家看几个动态加载页面的例子：[特定时间后页面变化](http://pythonscraping.com/pages/javascript/ajaxDemo.html)，[用户点击后页面变化](https://www.zhihu.com/#signin)

`应对`：对于动态加载页面的爬取最好的处理方式是：**PhantomJs+Selenium**,这个在下面的高级主题中介绍。

#### 验证码反爬虫
`原理`：有些网站会对站内特殊的数据做额外的保护，你只有正确填写验证码之后才能访问到该网页，在这种情况下爬取的难度就非常大了。但是也不是不能爬取，主要要看它使用的是何种验证方式。
`应对`：如果是普通的填写它给出的图片上的字符，那么可以使用图像识别的技术来处理，但其实现在的验证码往往都加了许多的干扰线，噪点之类的，连人类有时候都有可能识别错误，何况是机器呢！更厉害的是像12306的那种请点击以下所有包含海洋的图片，或者知乎的请点击下图中所有倒立的汉字等等，我只能说遇到类似这样的爬虫任务的话，你就只能认命了！

# 处理
在上面的一部分中，我们讨论了如何下载，下面要讨论的则是下载到网页之后我们怎样从网页中提取数据的问题了。
## 正则表达式
在计算机的世界中很多东西归根结底都可以归结到字符串的处理上，其中在某一字符串中提取另一种模式的字符串又是其中的重要组成部分。正则表达式解决的就是模式的描述的问题。python中的re模块提供了许多优秀的API,使Python语言拥有全部的正则表达式功能。详情参考[这里](http://cuiqingcai.com/977.html)
## BeautifulSoup
[BeautifulSoup](https://www.crummy.com/software/BeautifulSoup/bs4/doc/)
## Xpath
[Xpath](http://lxml.de/)
# 高级主题
## 动态页面的爬取
先看两个例子:[自动登录发表文章]()，[搜索并下载歌曲]()。体会下phantomjs+selenium可以做啥！
### phantomJS 
[phantomjs](http://phantomjs.org/quick-start.html)是一个无界面的,可脚本编程的WebKit浏览器引擎。它原生支持多种web标准：DOM操作，CSS选择器，JSON，Canvas 以及SVG。可以帮助我们像浏览器一样渲染JS处理的页面。

这么使用phantomjs呢？看－>[这里](http://javascript.ruanyifeng.com/tool/phantomjs.html)或[这里](http://cuiqingcai.com/2577.html).

### selenium python bindings
首先推荐一个[博客](https://huilansame.github.io/huilansame.github.io/page3/),这个人的博客写的全是selenium python的内容。

[selenium](http://selenium-python.readthedocs.io/)是一个web测试自动化的工具。Selenium Python bindings 提供了一个简单的API，让你使用Selenium WebDriver来编写功能/校验测试。通过Selenium Python的API，你可以非常直观的使用Selenium WebDriver的所有功能。Selenium Python bindings 使用非常简洁方便的API让你去使用像Firefox, IE, Chrome, Remote等等这样的Selenium WebDrivers（Selenium web驱动器）。在生产环境中我们一般使用phantomjs这样的轻量级浏览器作为selenium web驱动。
#### 安装
可以从PyPI的官方库中下载该selenium支持库：
``` bash
$ pip3 install selenium
```
#### 你的第一个selenium应用
``` python
from selenium import webdriver
from selenium.webdriver.common.keys import Keys

driver = webdriver.PhantomJS(executable_path='/usr/local/bin/phantomjs')
driver.get("http://www.python.org")
assert "Python" in driver.title
elem = driver.find_element_by_name("q")
elem.clear()
elem.send_keys("pycon")
elem.send_keys(Keys.RETURN)
assert "No results found." not in driver.page_source
driver.close()
```

#### 使用普通浏览器
``` python
from selenium import webdriver
from selenium.webdriver.common.keys import Keys

driver = webdriver.Chrome()
# driver = webdriver.FireFox()
# driver =webdriver.IE()
driver.get("http://www.python.org")
assert "Python" in driver.title
elem = driver.find_element_by_name("q")
elem.clear()
elem.send_keys("pycon")
elem.send_keys(Keys.RETURN)
assert "No results found." not in driver.page_source
driver.close()
```
使用普通的浏览器需要安装浏览器驱动程序，并且保证这些驱动程序在你的环境变量中。常见的浏览器及其驱动下载地址如下：

| 浏览器 | 驱动下载地址 |
| :-----------: | :----------------------: |
| Chrome | https://sites.google.com/a/chromium.org/chromedriver/downloads |
| Edge | https://developer.microsoft.com/en-us/microsoft-edge/tools/webdriver/ |
| Firefox |	https://github.com/mozilla/geckodriver/releases |
| Safari | https://webkit.org/blog/6900/webdriver-support-in-safari-10/ |
#### 打开一个页面
你想做的第一件事也许是使用WebDriver打开一个链接。常规的方法是调用get方法:
``` python
driver.get("http://www.google.com")
```
WebDriver 将等待，直到页面完全加载完毕（其实是等到 onload 方法执行完毕）， 然后返回继续执行你的脚本.

#### 元素定位
selenium with python给我们提供了大量的用于在页面中定位元素的API，有了这些API，你可以在页面上找到任何你需要的元素。详细内容看－>[这里](https://selenium-python-zh.readthedocs.io/en/latest/locating-elements.html)

#### 页面交互
##### 填写表单
首先找到表单元素对象，然后对表单元素对象调用send_keys(data)方法就可以把data填写到input框中。
``` python
driver.get("http://auth.muxixyz.com/login/")
username_elem=driver.find_element_by_id("username")
username_elem.send_keys("阿超")
password_elem=driver.find_element_by_id("password")
password_elem.send_keys("this is my pwd")
driver.find_element_by_id("submit").click()
WebDriverWait(driver,3).until(lambda x:x.find_element_by_xpath("//a[@href='http://share.muxixyz.com/']"))

#截取当前页面的图片
driver.driver.get_screenshot_as_file("1.png")
with open("1.html",'w') as f:
    #获取当前页面的源码
    f.write(driver.page_source)
```
上述代码就是以我的账号密码登录木犀内网，然后将跳转之后的页面的图截下来保存为1.png，将当前页面的源码保存为1.html.
#### 键盘模拟
[键盘模拟](http://selenium-python-zh.readthedocs.io/en/latest/api.html#module-selenium.webdriver.common.keys)
当调用一个元素对象的send_keys()方法的时候，不光可以向这个元素传递数据，也可以传递selenium.webdriver.common.keys．Keys的实例来模拟键盘的操作
，这可以用来测试网站的快捷键设置是否正确。
``` python
from selenium.webdriver.common.keys import Keys
driver.get("http://www.douban.com")
driver.get_screenshot_as_file("douban.png")
username = driver.find_element_by_name("form_email")
password = driver.find_element_by_name("form_password")
submit=driver.find_element_by_class_name("bn-submit")
username.send_keys("13636038496")
password.send_keys("this is my pwd")
submit.click()

#上面的三行代码也可以写成下面这样
# username.send_keys("13636038496")
# password.send_keys("this is my pwd"，Keys.RETURN)
```
一般网站的表单填写完了之后，都可以直接通过按下回车键提交表单,上面注释的代码就是模拟了这个过程。
#### 窗口切换
``` python
currentWin = driver.current_window_handle
#跳转到另一个新页面
driver.find_element_by_xpath("//p[@id='nv']/a[3]").click()
time.sleep(1)
#获取所有窗口的句柄
handles = driver.window_handles
for i in handles:
    if currentWin == i:
        continue
    else:
        #将driver与新的页面绑定起来
        driver = driver.switch_to_window(i)
```

``` python
driver.switch_to_window("windowName")
driver.switch_to_frame("frameName")
alert = driver.switch_to_alert()
```

#### 历史记录和定位
``` python
driver.forward()
driver.back()
```
#### cookie处理
``` python
#设置cookie
cookie = {'name':'foo','value':'bar'}
driver.add_cookie(cookie)

#输出当前url下可用的cookie
driver.get_cookies()
```


#### 设置等待
现在的app有许多都在使用ajax(Asynchronous Javascript And XML)技术,这使得在一个页面中的元素的出现时间会产生差异，这给元素的定位带来了不小的困难。如果一个元素还没有出现在DOM中，那么你用一个定位函数去定位这个元素的时候就会产生`ElementNotVisibleException`的错误。使用等待可以解决这个问题。Selenium Webdriver提供了两种等待：显式等待(Explicit Waits)和隐式等待(Implicit Waits).

##### 隐式等待
``` python
from selenium import webdriver

driver = webdriver.Chrome()
driver.implicitly_wait(30)  # 隐性等待，最长等30秒
driver.get('https://huilansame.github.io')

print(driver.current_url)
driver.quit()
```
隐形等待是设置了一个最长等待时间，如果在规定时间内网页加载完成，则执行下一步，否则一直等到时间截止，然后执行下一步。注意这里有一个弊端，那就是程序会一直等待整个页面加载完成，也就是一般情况下你看到浏览器标签栏那个小圈不再转，才会执行下一步，但有时候页面想要的元素早就在加载完成了，但是因为个别js之类的东西特别慢，我仍得等到页面全部完成才能执行下一步，我想等我要的元素出来之后就下一步怎么办？有办法，这就要看selenium提供的另一种等待方式——显性等待wait了。
Tip:隐性等待对整个driver的周期都起作用，所以只要设置一次即可

##### 显式等待
显示等待是你在你的代码中设置的等待某一个条件发生之后才继续向下执行的等待。selenium提供了一些方便的方法来帮助你设置你的等待，并且可以让等待只花所需要的时间。
``` python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

driver = webdriver.Chrome()
driver.get("http://somedomain/url_that_delays_loading")
try:
    element = WebDriverWait(driver, 10).until(
        EC.presence_of_element_located((By.ID, "myDynamicElement"))
    )
finally:
    driver.quit()
```
上面的例子中的意思是通过id来获取"myDynamicElement"元素，如果当前DOM中没有这个元素，那么就等待，最长等待的时间是10秒，如果十秒钟之后该元素仍然没有出现，则抛出`TimeoutException`错误。如果在等待的过程中该元素加载出来了，那么该函数就立即返回，停止等待。默认情况下，selenium webdriver每500毫秒，检查一下预期条件。

selenium with python提供了许多的常用的期望条件来帮助你操控你的浏览器。常用的预期条件如下：
``` 
selenium.webdriver.support.expected_conditions（模块）

这两个条件类验证title，验证传入的参数title是否等于或包含于driver.title
title_is
title_contains

这两个人条件验证元素是否出现，传入的参数都是元组类型的locator，如(By.ID, 'kw')
顾名思义，一个只要一个符合条件的元素加载出来就通过；另一个必须所有符合条件的元素都加载出来才行
presence_of_element_located
presence_of_all_elements_located

这三个条件验证元素是否可见，前两个传入参数是元组类型的locator，第三个传入WebElement
第一个和第三个其实质是一样的
visibility_of_element_located
invisibility_of_element_located
visibility_of

这两个人条件判断某段文本是否出现在某元素中，一个判断元素的text，一个判断元素的value
text_to_be_present_in_element
text_to_be_present_in_element_value

这个条件判断frame是否可切入，可传入locator元组或者直接传入定位方式：id、name、index或WebElement
frame_to_be_available_and_switch_to_it

这个条件判断是否有alert出现
alert_is_present

这个条件判断元素是否可点击，传入locator
element_to_be_clickable

这四个条件判断元素是否被选中，第一个条件传入WebElement对象，第二个传入locator元组
第三个传入WebElement对象以及状态，相等返回True，否则返回False
第四个传入locator以及状态，相等返回True，否则返回False
element_to_be_selected
element_located_to_be_selected
element_selection_state_to_be
element_located_selection_state_to_be

最后一个条件判断一个元素是否仍在DOM中，传入WebElement对象，可以判断页面是否刷新了
staleness_of
```

``` python
from selenium.webdriver.support import expected_conditions as EC

wait = WebDriverWait(driver, 10)
element = wait.until(EC.element_to_be_clickable((By.ID, 'someid')))
```

``` python
from selenium import webdriver
from selenium.webdriver.support.wait import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.common.by import By

driver = webdriver.Firefox()
driver.implicitly_wait(10)  # 隐性等待和显性等待可以同时用，但要注意：等待的最长时间取两者之中的大者
driver.get('https://huilansame.github.io')
locator = (By.LINK_TEXT, 'CSDN')

try:
    WebDriverWait(driver, 20, 0.2).until(EC.presence_of_element_located(locator))
    print(driver.find_element_by_link_text('CSDN').get_attribute('href'))
finally:
    driver.close()
```

如果上述提供的期待条件没有满足你的需求，你也可以自定义期望条件：可以通过一个实现了`__call__()`方法的类来自定义一个等待的期望条件。这个`__call__()`方法在不匹配的情况下，返回False即可。
``` python
class element_has_css_class(object):
  """An expectation for checking that an element has a particular css class.

  locator - used to find the element
  returns the WebElement once it has the particular css class
  """
  def __init__(self, locator, css_class):
    self.locator = locator
    self.css_class = css_class

  def __call__(self, driver):
    element = driver.find_element(*self.locator)   # Finding the referenced element
    if self.css_class in element.get_attribute("class"):
        return element
    else:
        return False

# Wait until an element with id='myNewInput' has class 'myCSSClass'
wait = WebDriverWait(driver, 10)
element = wait.until(element_has_css_class((By.ID, 'myNewInput'), "myCSSClass"))
```
除了上面介绍的几种等待的方式，使用`time.sleep()`也可以实现等待的效果，但是这种方法比较low,推荐不要使用。

#### PhantomJS请求配置
一般来讲，如果是做爬虫的话，使用的web driver都是PhantomJS，相较于大家桌面上的浏览器的话，它更轻量级，所以速度更快。在使用PhantomJS时，可以对其进行一些配置,比如设置请求头，设置网络代理等。
```python
from fake_useragent import UserAgent
from selenium import webdriver
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.common.desired_capabilities import DesiredCapabilities
ua = UserAgent()
#配置对象DesiredCapabilities
dcap = dict(DesiredCapabilities.PHANTOMJS)
#从USER_AGENTS列表中随机选一个浏览器头，伪装浏览器
dcap["phantomjs.page.settings.userAgent"] = ua.random
# 不载入图片，爬页面速度会快很多
dcap["phantomjs.page.settings.loadImages"] = True
# 设置代理
service_args = ['--proxy=127.0.0.1:9999','--proxy-type=socks5']
#打开带配置信息的phantomJS浏览器
driver = webdriver.PhantomJS(phantomjs_driver_path, desired_capabilities=dcap,service_args=service_args)

# 隐式等待5秒，可以自己调节
driver.implicitly_wait(5)
# 设置10秒页面超时返回，类似于requests.get()的timeout选项，driver.get()没有timeout选项
# 以前遇到过driver.get(url)一直不返回，但也不报错的问题，这时程序会卡住，设置超时选项能解决这个问题。
driver.set_page_load_timeout(10)
# 设置10秒脚本超时时间
driver.set_script_timeout(10)

driver.get("http://www.baidu.com")
```




## 道德问题
在互联网这个复杂的环境中，搜索引擎本身的爬虫，出于个人目的的爬虫，商业爬虫肆意横行，肆意掠夺网上的或者公共或者私人的资源。显然数据的收集并不是为所欲为，有一些协议或者原则还是需要每一个人注意。
### robots协议
一般情况下网站的根目录下存在着一个robots.txt的文件，用于告诉爬虫那些文件夹或者哪些文件是网站的拥有者或者管理员不希望被搜索引擎和爬虫浏览的，或者是不希望被非人类的东西查看的。但是不仅仅如此，在这个文件中，有时候还会指明sitemap的位置，爬虫可以直接寻找sitemap而不用费力去爬取网站。




