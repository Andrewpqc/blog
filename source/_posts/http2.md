---
title: HTTP常用首部字段
comments: true
date: 2018-07-15 23:10:58
updated: 2018-07-16 14:59:20
tags: [HTTP,Header]
categories: Network
permalink:
---
在上一篇文章[HTTP协议总结](https://andrewpqc.github.io/2018/07/15/http1/#more)中讲到了HTTP报文结构，无论是请求报文还是响应报文中都有首部。在HTTP报文众多的字段当中，HTTP首部字段包含的信息最为丰富。首部字段同时存在于请求和响应报文内，并涵盖HTTP报文相关的内容信息。使用首部字段是为了给客服端和服务器端提供报文主体大小、所使用的语言、认证信息等内容。下面我们就总结一下在HTTP请求和响应中常用的首部字段。

# 首部字段的结构和分类
## 结构

+ HTTP首部字段是由首部字段名和字段值构成的，中间用冒号":"分隔。   
+ 另外，字段值对应单个 HTTP 首部字段可以有多个值。
+ 当HTTP 报文首部中出现了两个或以上具有相同首部字段名的首部字段时，这种情况在规范内尚未明确，根据浏览器内部处理逻辑的不同，优先处理的顺序可能不同，结果可能并不一致。

如下是两个首部字段的例子:

| 首部字段名 | 冒号 | 字段值 |
| :---------------: | :-------------------------: | :-------------------: |
| Content-Type | : | text/html |
| Keep-Alive | : | timeout=30, max=120 |

## 分类
首部字段根据实际用途被分为以下4种类型：

| 类型 |　描述　|
| :-------------: | :------------------: |
| 通用首部字段 | 请求报文和响应报文两方都会使用的首部 |
| 请求首部字段 | 从客户端向服务器端发送请求报文时使用的首部。补充了请求的附加内容、客户端信息、响应内容相关优先级等信息 |
| 响应首部字段 | 从服务器端向客户端返回响应报文时使用的首部。补充了响应的附加内容，也会要求客户端附加额外的内容信息。 |
| 实体首部字段 | 针对请求报文和响应报文的实体部分使用的首部。补充了资源内容更新时间等与实体有关的的信息。 |

下面我按照这四种类型详细的总结一下。

# 通用首部字段

| 首部字段名 |	说明 |
| :--------: | :-----------: |
| Cache-Control  |	控制缓存的行为 |
| Connection  |	逐挑首部、连接的管理 |
| Date |	创建报文的日期时间 |
| Pragma | 	报文指令 |
| Trailer |	报文末端的首部一览 |
| Transfer-Encoding | 	指定报文主体的传输编码方式 |
| Upgrade |	升级为其他协议 |
| Via  |	代理服务器的相关信息 |
| Warning |	错误通知 |


## Cache-Control
通过指定首部字段 Cache-Control 的指令，就能操作缓存的工作机制。

### 可用的指令一览
可用的指令按请求和响应分类如下：
#### 缓存请求指令

| 指令 |	参数 |	说明 |
| :----------: | :---------: | :----------: |
| no-cache |	无 |	强制向服务器再次验证 |
| no-store	| 无 |	不缓存请求或响应的任何内容 |
| max-age = [秒] |	必需 |	响应的最大Age值 |
| max-stale( =[秒]) |	可省略	 | 接收已过期的响应 |
| min-fresh = [秒] |	必需 |	期望在指定时间内的响应仍有效 |
| no-transform	| 无 |	代理不可更改媒体类型 |
| only-if-cached |	无 |	从缓存获取资源 |
| cache-extension |	-  | 	新指令标记（token） |
#### 缓存响应指令

| 指令 |	参数 |  	说明 |
| :----------: | :----------: | :----------------: |
| public |	无 |	可向任意方提供响应的缓存 |
| private |	可省略 |	仅向特定用户返回响应 |
| no-cache |	可省略	| 缓存前必须先确认其有效性 |
| no-store |	无 |	不缓存请求或响应的任何内容 |
| no-transform	| 无 |	代理不可更改媒体类型 |
| must-revalidate |	无 |	可缓存但必须再向源服务器进行确认 |
| proxy-revalidate	| 无 |	要求中间缓存服务器对缓存的响应有效性再进行确认 |
| max-age = [秒]	| 必需 |	响应的最大Age值 |
| s-maxage = [秒] |	必需 |	公共缓存服务器响应的最大Age值 |
| cache-extension |	- |	新指令标记（token） |

### 表示能否缓存的指令
+ public 指令
``` 
Cache-Control: public
```
当指定使用 public 指令时，则明确表明其他用户也可利用缓存。

+ private 指令
```
Cache-Control: private
```
当指定 private 指令后，响应只以特定的用户作为对象，这与 public 指令的行为相反。缓存服务器会对该特定用户提供资源缓存的服务，对于其他用户发送过来的请求，代理服务器则不会返回缓存。

+ no-cache 指令
```
Cache-Control: no-cache
```
使用 no-cache 指令是为了防止从缓存中返回过期的资源。
客户端发送的请求中如果包含 no-cache 指令，则表示客户端将不会接收缓存过的响应。于是，“中间”的缓存服务器必须把客户端请求转发给源服务器。
如果服务器中返回的响应包含 no-cache 指令，那么缓存服务器不能对资源进行缓存。源服务器以后也将不再对缓存服务器请求中提出的资源有效性进行确认，且禁止其对响应资源进行缓存操作。
Cache-Control: no-cache=Location
由服务器返回的响应中，若报文首部字段 Cache-Control 中对 no-cache 字段名具体指定参数值，那么客户端在接收到这个被指定参数值的首部字段对应的响应报文后，就不能使用缓存。换言之，无参数值的首部字段可以使用缓存。只能在响应指令中指定该参数。

+ no-store 指令
```
Cache-Control: no-store
```
当使用 no-store 指令时，暗示请求（和对应的响应）或响应中包含机密信息。因此，该指令规定缓存不能在本地存储请求或响应的任一部分。
注意：no-cache 指令代表不缓存过期的指令，缓存会向源服务器进行有效期确认后处理资源；no-store 指令才是真正的不进行缓存。

### 指定缓存期限和认证的指令
+ s-maxage 指令
```
Cache-Control: s-maxage=604800（单位：秒）
```

s-maxage 指令的功能和 max-age 指令的相同，它们的不同点是 s-maxage 指令只适用于供多位用户使用的公共缓存服务器（一般指代理）。也就是说，对于向同一用户重复返回响应的服务器来说，这个指令没有任何作用。
另外，当使用 s-maxage 指令后，则直接忽略对 Expires 首部字段及 max-age 指令的处理。
+ max-age 指令
```
Cache-Control: max-age=604800（单位：秒）
```
当客户端发送的请求中包含 max-age 指令时，如果判定缓存资源的缓存时间数值比指定的时间更小，那么客户端就接收缓存的资源。另外，当指定 max-age 的值为0，那么缓存服务器通常需要将请求转发给源服务器。
当服务器返回的响应中包含 max-age 指令时，缓存服务器将不对资源的有效性再作确认，而 max-age 数值代表资源保存为缓存的最长时间。
应用 HTTP/1.1 版本的缓存服务器遇到同时存在 Expires 首部字段的情况时，会优先处理 max-age 指令，并忽略掉 Expires 首部字段；而 HTTP/1.0 版本的缓存服务器则相反。
+ min-fresh 指令
```
Cache-Control: min-fresh=60（单位：秒）
```
min-fresh 指令要求缓存服务器返回至少还未过指定时间的缓存资源。

+ max-stale 指令
```
Cache-Control: max-stale=3600（单位：秒）
```
使用 max-stale 可指示缓存资源，即使过期也照常接收。
如果指令未指定参数值，那么无论经过多久，客户端都会接收响应；如果指定了具体参数值，那么即使过期，只要仍处于 max-stale 指定的时间内，仍旧会被客户端接收。
+ only-if-cached 指令
```
Cache-Control: only-if-cached
```
表示客户端仅在缓存服务器本地缓存目标资源的情况下才会要求其返回。换言之，该指令要求缓存服务器不重新加载响应，也不会再次确认资源的有效性。

+ must-revalidate 指令
```
Cache-Control: must-revalidate
```
使用 must-revalidate 指令，代理会向源服务器再次验证即将返回的响应缓存目前是否仍有效。另外，使用 must-revalidate 指令会忽略请求的 max-stale 指令。

+ proxy-revalidate 指令
```
Cache-Control: proxy-revalidate
```
proxy-revalidate 指令要求所有的缓存服务器在接收到客户端带有该指令的请求返回响应之前，必须再次验证缓存的有效性。

+ no-transform 指令
```
Cache-Control: no-transform
```
使用 no-transform 指令规定无论是在请求还是响应中，缓存都不能改变实体主体的媒体类型。这样做可防止缓存或代理压缩图片等类似操作。

### Cache-Control 扩展
```
Cache-Control: private, community="UCI"
```
通过 cache-extension 标记（token），可以扩展 Cache-Control 首部字段内的指令。上述 community 指令即扩展的指令，如果缓存服务器不能理解这个新指令，就会直接忽略掉。

## Connection
Connection 首部字段具备以下两个作用：

+ 控制不再转发的首部字段
```
Connection: Upgrade
```
在客户端发送请求和服务器返回响应中，使用 Connection 首部字段，可控制不再转发给代理的首部字段，即删除后再转发（即Hop-by-hop首部）。

+ 管理持久连接
```
Connection: close
```
HTTP/1.1 版本的默认连接都是持久连接。当服务器端想明确断开连接时，则指定 Connection 首部字段的值为 close。
```
Connection: Keep-Alive
```
HTTP/1.1 之前的 HTTP 版本的默认连接都是非持久连接。为此，如果想在旧版本的 HTTP 协议上维持持续连接，则需要指定 Connection 首部字段的值为 Keep-Alive。

## Date
表明创建 HTTP 报文的日期和时间。
Date: Mon, 10 Jul 2017 15:50:06 GMT
HTTP/1.1 协议使用在 RFC1123 中规定的日期时间的格式。

## Pragma
Pragma 首部字段是 HTTP/1.1 版本之前的历史遗留字段，仅作为与 HTTP/1.0 的向后兼容而定义。
```
Pragma: no-cache
```
该首部字段属于通用首部字段，但只用在客户端发送的请求中，要求所有的中间服务器不返回缓存的资源。
所有的中间服务器如果都能以 HTTP/1.1 为基准，那直接采用 Cache-Control: no-cache 指定缓存的处理方式最为理想。但是要整体掌握所有中间服务器使用的 HTTP 协议版本却是不现实的，所以，发送的请求会同时包含下面两个首部字段：
```
Cache-Control: no-cache
Pragma: no-cache
```
##  Trailer
```
Trailer: Expires
```
首部字段 Trailer 会事先说明在报文主体后记录了哪些首部字段。可应用在 HTTP/1.1 版本分块传输编码时。

## Transfer-Encoding
```
Transfer-Encoding: chunked
```

规定了传输报文主体时采用的编码方式。
HTTP/1.1 的传输编码方式仅对分块传输编码有效。
## Upgrade
```
Upgrade: TSL/1.0
```
用于检测 HTTP 协议及其他协议是否可使用更高的版本进行通信，其参数值可以用来指定一个完全不同的通信协议。

## Via
```
Via: 1.1 a1.sample.com(Squid/2.7)
```
为了追踪客户端和服务器端之间的请求和响应报文的传输路径。
报文经过代理或网关时，会现在首部字段 Via 中附加该服务器的信息，然后再进行转发。
首部字段 Via 不仅用于追踪报文的转发，还可避免请求回环的发生。
## Warning
该首部字段通常会告知用户一些与缓存相关的问题的警告。
Warning 首部字段的格式如下：
```
Warning：[警告码][警告的主机:端口号] "[警告内容]"([日期时间])
```
最后的日期时间可省略。
HTTP/1.1 中定义了7种警告，警告码对应的警告内容仅推荐参考，另外，警告码具备扩展性，今后有可能追加新的警告码。

| 警告码 |	警告内容 |	说明 |
|:---------:| :---------: | :--------: |
| 110  |	Response is stale(响应已过期) |	代理返回已过期的资源 |
| 111 |	Revalidation failed(再验证失败) |	代理再验证资源有效性时失败（服务器无法到达等原因） |
| 112 |	Disconnection operation(断开连接操作)	| 代理与互联网连接被故意切断 |
| 113 |	Heuristic expiration(试探性过期)	| 响应的试用期超过24小时(有效缓存的设定时间大于24小时的情况下) |
| 199 |	Miscellaneous warning(杂项警告)	 | 任意的警告内容 |
| 214 |	Transformation applied(使用了转换) |	代理对内容编码或媒体类型等执行了某些处理时 |
| 299 |	Miscellaneous persistent warning(持久杂项警告) |	任意的警告内容 |

# 请求首部字段
| 首部字段名 |	说明 |
| :---------: | :----------: |
| Accept |	用户代理可处理的媒体类型 |
| Accept-Charset |	优先的字符集 |
| Accept-Encoding |	优先的内容编码 |
| Accept-Language |	优先的语言（自然语言） |
| Authorization	| Web认证信息 |
| Expect |	期待服务器的特定行为 |
| From |	用户的电子邮箱地址 |
| Host |	请求资源所在服务器 |
| If-Match |	比较实体标记（ETag）|
| If-Modified-Since |	比较资源的更新时间 |
| If-None-Match |	比较实体标记（与 If-Macth 相反） |
| If-Range |	资源未更新时发送实体 Byte 的范围请求 |
| If-Unmodified-Since |	比较资源的更新时间(与 If-Modified-Since 相反) |
| Max-Forwards |	最大传输逐跳数 |
| Proxy-Authorization |	代理服务器要求客户端的认证信息 |
| Range |	实体的字节范围请求 |
| Referer |	对请求中 URI 的原始获取方 |
| TE |	传输编码的优先级 |
| User-Agent |	HTTP 客户端程序的信息 |
## Accept
```
Accept: text/html, application/xhtml+xml, application/xml; q=0.5
```
Accept 首部字段可通知服务器，用户代理能够处理的媒体类型及媒体类型的相对优先级。可使用 type/subtype 这种形式，一次指定多种媒体类型。
若想要给显示的媒体类型增加优先级，则使用 q=[数值] 来表示权重值，用分号（;）进行分隔。权重值的范围 0~1（可精确到小数点后三位），且 1 为最大值。不指定权重值时，默认为 1。
## Accept-Charset
```
Accept-Charset: iso-8859-5, unicode-1-1; q=0.8
```
Accept-Charset 首部字段可用来通知服务器用户代理支持的字符集及字符集的相对优先顺序。另外，可一次性指定多种字符集。同样使用 q=[数值] 来表示相对优先级。

## Accept-Encoding
```
Accept-Encoding: gzip, deflate
```
Accept-Encoding 首部字段用来告知服务器用户代理支持的内容编码及内容编码的优先顺序，并可一次性指定多种内容编码。同样使用 q=[数值] 来表示相对优先级。也可使用星号（*）作为通配符，指定任意的编码格式。

## Accept-Language
```
Accept-Lanuage: zh-cn,zh;q=0.7,en=us,en;q=0.3
```
告知服务器用户代理能够处理的自然语言集（指中文或英文等），以及自然语言集的相对优先级，可一次性指定多种自然语言集。同样使用 q=[数值] 来表示相对优先级。

## Authorization
```
Authorization: Basic ldfKDHKfkDdasSAEdasd==
```
告知服务器用户代理的认证信息（证书值）。通常，想要通过服务器认证的用户代理会在接收到返回的 401 状态码响应后，把首部字段 Authorization 加入请求中。共用缓存在接收到含有 Authorization 首部字段的请求时的操作处理会略有差异。

## Expect
```
Expect: 100-continue
```
告知服务器客户端期望出现的某种特定行为。

## From
```
From: Deeson_Woo@163.com
```
告知服务器使用用户代理的电子邮件地址。

## Host
```
Host: andrewpqc.xyz
```
告知服务器，请求的资源所处的互联网主机和端口号。
Host 首部字段是 HTTP/1.1 规范内唯一一个必须被包含在请求内的首部字段。
若服务器未设定主机名，那直接发送一个空值即可  Host: 。
## If-Match
形如 If-xxx 这种样式的请求首部字段，都可称为条件请求。服务器接收到附带条件的请求后，只有判断指定条件为真时，才会执行请求。
```
If-Match: "123456"
```
首部字段 If-Match，属附带条件之一，它会告知服务器匹配资源所用的实体标记（ETag）值。这时的服务器无法使用弱 ETag 值。
服务器会比对 If-Match 的字段值和资源的 ETag 值，仅当两者一致时，才会执行请求。反之，则返回状态码 412 Precondition Failed 的响应。
还可以使用星号（*）指定 If-Match 的字段值。针对这种情况，服务器将会忽略 ETag 的值，只要资源存在就处理请求。
## If-Modified-Since
```
If-Modified-Since: Mon, 10 Jul 2017 15:50:06 GMT
```
首部字段 If-Modified-Since，属附带条件之一，用于确认代理或客户端拥有的本地资源的有效性。
它会告知服务器若 If-Modified-Since 字段值早于资源的更新时间，则希望能处理该请求。而在指定 If-Modified-Since 字段值的日期时间之后，如果请求的资源都没有过更新，则返回状态码 304 Not Modified 的响应。
## If-None-Match
```
If-None-Match: "123456"
```
首部字段 If-None-Match 属于附带条件之一。它和首部字段 If-Match 作用相反。用于指定 If-None-Match 字段值的实体标记（ETag）值与请求资源的 ETag 不一致时，它就告知服务器处理该请求。

## If-Range
```
If-Range: "123456"
```
首部字段 If-Range 属于附带条件之一。它告知服务器若指定的 If-Range 字段值（ETag 值或者时间）和请求资源的 ETag 值或时间相一致时，则作为范围请求处理。反之，则返回全体资源。
下面我们思考一下不使用首部字段 If-Range 发送请求的情况。服务器端的资源如果更新，那客户端持有资源中的一部分也会随之无效，当然，范围请求作为前提是无效的。这时，服务器会暂且以状态码 412 Precondition Failed 作为响应返回，其目的是催促客户端再次发送请求。这样一来，与使用首部字段 If-Range 比起来，就需要花费两倍的功夫。
## If-Unmodified-Since
```
If-Unmodified-Since: Mon, 10 Jul 2017 15:50:06 GMT
```
首部字段 If-Unmodified-Since 和首部字段 If-Modified-Since 的作用相反。它的作用的是告知服务器，指定的请求资源只有在字段值内指定的日期时间之后，未发生更新的情况下，才能处理请求。如果在指定日期时间后发生了更新，则以状态码 412 Precondition Failed 作为响应返回。

## Max-Forwards
```
Max-Forwards: 10
```
通过 TRACE 方法或 OPTIONS 方法，发送包含首部字段 Max-Forwards 的请求时，该字段以十进制整数形式指定可经过的服务器最大数目。服务器在往下一个服务器转发请求之前，Max-Forwards 的值减 1 后重新赋值。当服务器接收到 Max-Forwards 值为 0 的请求时，则不再进行转发，而是直接返回响应。

## Proxy-Authorization
```
Proxy-Authorization: Basic dGlwOjkpNLAGfFY5
```
接收到从代理服务器发来的认证质询时，客户端会发送包含首部字段 Proxy-Authorization 的请求，以告知服务器认证所需要的信息。
这个行为是与客户端和服务器之间的 HTTP 访问认证相类似的，不同之处在于，认证行为发生在客户端与代理之间。
## Range
```
Range: bytes=5001-10000
```
对于只需获取部分资源的范围请求，包含首部字段 Range 即可告知服务器资源的指定范围。
接收到附带 Range 首部字段请求的服务器，会在处理请求之后返回状态码为 206 Partial Content 的响应。无法处理该范围请求时，则会返回状态码 200 OK 的响应及全部资源。
## Referer
```
Referer: http://www.sample.com/index.html
```
首部字段 Referer 会告知服务器请求的原始资源的 URI。

## TE
```
TE: gzip, deflate; q=0.5
```
首部字段 TE 会告知服务器客户端能够处理响应的传输编码方式及相对优先级。它和首部字段 Accept-Encoding 的功能很相像，但是用于传输编码。
首部字段 TE 除指定传输编码之外，还可以指定伴随 trailer 字段的分块传输编码的方式。应用后者时，只需把 trailers 赋值给该字段值。TE: trailers
## User-Agent
```
User-Agent: Mozilla/5.0 (Windows NT 6.1; WOW64; rv:13.0) Gecko/20100101
```
首部字段 User-Agent 会将创建请求的浏览器和用户代理名称等信息传达给服务器。
由网络爬虫发起请求时，有可能会在字段内添加爬虫作者的电子邮件地址。此外，如果请求经过代理，那么中间也很可能被添加上代理服务器的名称。
# 响应首部字段
| 首部字段名 |	说明 |
| :-----------: | :-------------------: |
| Accept-Ranges |	是否接受字节范围请求 |
| Age |	推算资源创建经过时间 |
| ETag | 	资源的匹配信息 |
| Location |	令客户端重定向至指定 URI |
| Proxy-Authenticate |	代理服务器对客户端的认证信息 |
| Retry-After |	对再次发起请求的时机要求 |
| Server |	HTTP 服务器的安装信息 |
| Vary |	代理服务器缓存的管理信息 |
| WWW-Authenticate |	服务器对客户端的认证信息 |
## Accept-Ranges
```
Accept-Ranges: bytes
```
首部字段 Accept-Ranges 是用来告知客户端服务器是否能处理范围请求，以指定获取服务器端某个部分的资源。
可指定的字段值有两种，可处理范围请求时指定其为 bytes，反之则指定其为 none。
## Age
```
Age: 1200
```

首部字段 Age 能告知客户端，源服务器在多久前创建了响应。字段值的单位为秒。
若创建该响应的服务器是缓存服务器，Age 值是指缓存后的响应再次发起认证到认证完成的时间值。代理创建响应时必须加上首部字段 Age。
## ETag
```
ETag: "usagi-1234"
```
首部字段 ETag 能告知客户端实体标识。它是一种可将资源以字符串形式做唯一性标识的方式。服务器会为每份资源分配对应的 ETag 值。
另外，当资源更新时，ETag 值也需要更新。生成 ETag 值时，并没有统一的算法规则，而仅仅是由服务器来分配。
ETag 中有强 ETag 值和弱 ETag 值之分。强 ETag 值，不论实体发生多么细微的变化都会改变其值；弱 ETag 值只用于提示资源是否相同。只有资源发生了根本改变，产生差异时才会改变 ETag 值。这时，会在字段值最开始处附加 W/： ETag: W/"usagi-1234"。
## Location
```
Location: http://www.sample.com/sample.html
```
使用首部字段 Location 可以将响应接收方引导至某个与请求 URI 位置不同的资源。
基本上，该字段会配合 3xx ：Redirection 的响应，提供重定向的 URI。
几乎所有的浏览器在接收到包含首部字段 Location 的响应后，都会强制性地尝试对已提示的重定向资源的访问。
## Proxy-Authenticate
```
Proxy-Authenticate: Basic realm="Usagidesign Auth"
```
首部字段 Proxy-Authenticate 会把由代理服务器所要求的认证信息发送给客户端。
它与客户端和服务器之间的 HTTP 访问认证的行为相似，不同之处在于其认证行为是在客户端与代理之间进行的。
## Retry-After
```
Retry-After: 180
```
首部字段 Retry-After 告知客户端应该在多久之后再次发送请求。主要配合状态码 503 Service Unavailable 响应，或 3xx Redirect 响应一起使用。
字段值可以指定为具体的日期时间（Mon, 10 Jul 2017 15:50:06 GMT 等格式），也可以是创建响应后的秒数。
## Server
```
Server: Apache/2.2.6 (Unix) PHP/5.2.5
```
首部字段 Server 告知客户端当前服务器上安装的 HTTP 服务器应用程序的信息。不单单会标出服务器上的软件应用名称，还有可能包括版本号和安装时启用的可选项。

## Vary
```
Vary: Accept-Language
```
首部字段 Vary 可对缓存进行控制。源服务器会向代理服务器传达关于本地缓存使用方法的命令。
从代理服务器接收到源服务器返回包含 Vary 指定项的响应之后，若再要进行缓存，仅对请求中含有相同 Vary 指定首部字段的请求返回缓存。即使对相同资源发起请求，但由于 Vary 指定的首部字段不相同，因此必须要从源服务器重新获取资源。
## WWW-Authenticate
```
WWW-Authenticate: Basic realm="Usagidesign Auth"
```
首部字段 WWW-Authenticate 用于 HTTP 访问认证。它会告知客户端适用于访问请求 URI 所指定资源的认证方案（Basic 或是 Digest）和带参数提示的质询（challenge）。

# 实体首部字段
| 首部字段名 |	说明 |
| :------------: | :------------------: |
| Allow	| 资源可支持的 HTTP 方法 |
| Content-Encoding |	实体主体适用的编码方式 |
| Content-Language |	实体主体的自然语言 |
| Content-Length |	实体主体的大小（单位：字节） |
| Content-Location |	替代对应资源的 URI |
| Content-MD5 |	实体主体的报文摘要 |
| Content-Range |	实体主体的位置范围 |
| Content-Type |	实体主体的媒体类型 |
| Expires |	实体主体过期的日期时间 |
| Last-Modified |	资源的最后修改日期时间 |
## Allow
```
Allow: GET, HEAD
```
首部字段 Allow 用于通知客户端能够支持 Request-URI 指定资源的所有 HTTP 方法。
当服务器接收到不支持的 HTTP 方法时，会以状态码 405 Method Not Allowed 作为响应返回。与此同时，还会把所有能支持的 HTTP 方法写入首部字段 Allow 后返回。
## Content-Encoding
```
Content-Encoding: gzip
```
首部字段 Content-Encoding 会告知客户端服务器对实体的主体部分选用的内容编码方式。内容编码是指在不丢失实体信息的前提下所进行的压缩。
主要采用这 4 种内容编码的方式（gzip、compress、deflate、identity）。
## Content-Language
```
Content-Language: zh-CN
```
首部字段 Content-Language 会告知客户端，实体主体使用的自然语言（指中文或英文等语言）。

## Content-Length
```
Content-Length: 15000
```
首部字段 Content-Length 表明了实体主体部分的大小（单位是字节）。对实体主体进行内容编码传输时，不能再使用 Content-Length首部字段。

## Content-Location
```
Content-Location: http://www.sample.com/index.html
```
首部字段 Content-Location 给出与报文主体部分相对应的 URI。和首部字段 Location 不同，Content-Location 表示的是报文主体返回资源对应的 URI。

## Content-MD5
```
Content-MD5: OGFkZDUwNGVhNGY3N2MxMDIwZmQ4NTBmY2IyTY==
```
首部字段 Content-MD5 是一串由 MD5 算法生成的值，其目的在于检查报文主体在传输过程中是否保持完整，以及确认传输到达。

## Content-Range
```
Content-Range: bytes 5001-10000/10000
```
针对范围请求，返回响应时使用的首部字段 Content-Range，能告知客户端作为响应返回的实体的哪个部分符合范围请求。字段值以字节为单位，表示当前发送部分及整个实体大小。

## Content-Type
```
Content-Type: text/html; charset=UTF-8
```
首部字段 Content-Type 说明了实体主体内对象的媒体类型。和首部字段 Accept 一样，字段值用 type/subtype 形式赋值。参数 charset 使用 iso-8859-1 或 euc-jp 等字符集进行赋值。

## Expires
```
Expires: Mon, 10 Jul 2017 15:50:06 GMT
```
首部字段 Expires 会将资源失效的日期告知客户端。
缓存服务器在接收到含有首部字段 Expires 的响应后，会以缓存来应答请求，在 Expires 字段值指定的时间之前，响应的副本会一直被保存。当超过指定的时间后，缓存服务器在请求发送过来时，会转向源服务器请求资源。
源服务器不希望缓存服务器对资源缓存时，最好在 Expires 字段内写入与首部字段 Date 相同的时间值。
## Last-Modified
```
Last-Modified: Mon, 10 Jul 2017 15:50:06 GMT
```
首部字段 Last-Modified 指明资源最终修改的时间。一般来说，这个值就是 Request-URI 指定资源被修改的时间。但类似使用 CGI 脚本进行动态数据处理时，该值有可能会变成数据最终修改时的时间。

# 为 Cookie 服务的首部字段
| 首部字段名 |	说明 |	首部类型 |
| :--------: | :-----------: | :---------------: |
| Set-Cookie |	开始状态管理所使用的 Cookie 信息 |	响应首部字段 |
| Cookie |	服务器接收到的 Cookie 信息 |	请求首部字段 |
## Set-Cookie
```
Set-Cookie: status=enable; expires=Mon, 10 Jul 2017 15:50:06 GMT; path=/;
```

下面的表格列举了 Set-Cookie 的字段值。

| 属性 |	说明 |
| :--------: | :----------------: |
| NAME=VALUE |	赋予 Cookie 的名称和其值（必需项） |
| expires=DATE |	Cookie 的有效期（若不明确指定则默认为浏览器关闭前为止）|
| path=PATH |	将服务器上的文件目录作为Cookie的适用对象（若不指定则默认为文档所在的文件目录）|
| domain=域名 |	作为 Cookie 适用对象的域名 （若不指定则默认为创建 Cookie的服务器的域名）|
| Secure |	仅在 HTTPS 安全通信时才会发送 Cookie |
| HttpOnly |	加以限制，使 Cookie 不能被 JavaScript 脚本访问 |

### expires 属性
Cookie 的 expires 属性指定浏览器可发送 Cookie 的有效期。
当省略 expires 属性时，其有效期仅限于维持浏览器会话（Session）时间段内。这通常限于浏览器应用程序被关闭之前。
另外，一旦 Cookie 从服务器端发送至客户端，服务器端就不存在可以显式删除 Cookie 的方法。但可通过覆盖已过期的 Cookie，实现对客户端 Cookie 的实质性删除操作。
### path 属性
Cookie 的 path 属性可用于限制指定 Cookie 的发送范围的文件目录。

### domain 属性
通过 Cookie 的 domain 属性指定的域名可做到与结尾匹配一致。比如，当指定 example.com 后，除example.com 以外，www.example.com 或 www2.example.com 等都可以发送 Cookie。
因此，除了针对具体指定的多个域名发送 Cookie 之 外，不指定 domain 属性显得更安全。
8.1.4 secure 属性
Cookie 的 secure 属性用于限制 Web 页面仅在 HTTPS 安全连接时，才可以发送 Cookie。

### HttpOnly 属性
Cookie 的 HttpOnly 属性是 Cookie 的扩展功能，它使 JavaScript 脚本无法获得 Cookie。其主要目的为防止跨站脚本攻击（Cross-site scripting，XSS）对 Cookie 的信息窃取。
通过上述设置，通常从 Web 页面内还可以对 Cookie 进行读取操作。但使用 JavaScript 的 document.cookie 就无法读取附加 HttpOnly 属性后的 Cookie 的内容了。因此，也就无法在 XSS 中利用 JavaScript 劫持 Cookie 了。
## Cookie
```
Cookie: status=enable
```
首部字段 Cookie 会告知服务器，当客户端想获得 HTTP 状态管理支持时，就会在请求中包含从服务器接收到的 Cookie。接收到多个 Cookie 时，同样可以以多个 Cookie 形式发送。

# 其他首部字段
HTTP 首部字段是可以自行扩展的。所以在 Web 服务器和浏览器的应用上，会出现各种非标准的首部字段。
以下是最为常用的首部字段。

## X-Frame-Options
```
X-Frame-Options: DENY
```
首部字段 X-Frame-Options 属于 HTTP 响应首部，用于控制网站内容在其他 Web 网站的 Frame 标签内的显示问题。其主要目的是为了防止点击劫持（clickjacking）攻击。首部字段 X-Frame-Options 有以下两个可指定的字段值：

DENY：拒绝
SAMEORIGIN：仅同源域名下的页面（Top-level-browsing-context）匹配时许可。（比如，当指定 http://sample.com/sample.html 页面为 SAMEORIGIN 时，那么 sample.com 上所有页面的 frame 都被允许可加载该页面，而 example.com 等其他域名的页面就不行了）

## X-XSS-Protection
```
X-XSS-Protection: 1
```
首部字段 X-XSS-Protection 属于 HTTP 响应首部，它是针对跨站脚本攻击（XSS）的一种对策，用于控制浏览器 XSS 防护机制的开关。首部字段 X-XSS-Protection 可指定的字段值如下:

0 ：将 XSS 过滤设置成无效状态
1 ：将 XSS 过滤设置成有效状态
## DNT
```
DNT: 1
```
首部字段 DNT 属于 HTTP 请求首部，其中 DNT 是 Do Not Track 的简称，意为拒绝个人信息被收集，是表示拒绝被精准广告追踪的一种方法。首部字段 DNT 可指定的字段值如下：

0 ：同意被追踪
1 ：拒绝被追踪
由于首部字段 DNT 的功能具备有效性，所以 Web 服务器需要对 DNT做对应的支持。

## P3P
```
P3P: CP="CAO DSP LAW CURa ADMa DEVa TAIa PSAa PSDa IVAa IVDa OUR BUS IND
```
首部字段 P3P 属于 HTTP 响应首部，通过利用 P3P（The Platform for Privacy Preferences，在线隐私偏好平台）技术，可以让 Web 网站上的个人隐私变成一种仅供程序可理解的形式，以达到保护用户隐私的目的。
要进行 P3P 的设定，需按以下操作步骤进行：

步骤 1：创建 P3P 隐私
步骤 2：创建 P3P 隐私对照文件后，保存命名在 /w3c/p3p.xml
步骤 3：从 P3P 隐私中新建 Compact policies 后，输出到 HTTP 响应中



