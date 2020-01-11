---
title: Docker网络管理
comments: true
date: 2017-09-20 12:33:41
updated: 2017-11-16 12:33:41
tags: [Network in Docker]
categories: Docker
permalink:
---
在实践中，经常需要多个服务组件容器之间共同协作，这往往需要多个容器之间能够相互访问到对方的服务。同时，为了容器的安全有时我们也要避免容器之间的访问。

# 容器与用户的连接
## 端口映射

要想外部用户访问到服务器主机中容器中的服务，那么就要把提供服务的容器的端口映射到服务器主机的端口上。这样用户访问服务器主机的对应端口就可以访问到容器中的服务了。这是通过run命令的两个选项实现的
``` bash
docker run [-p]|[-P]………
-P –publish-all=ture|false 默认为false
```
用大写的P将会对容器所暴露的所有的端口进行映射,命令如下：
``` bash
$ docker run –P -i –t IMAGE [COMMAND] [ARG…]
```
这会将镜像所暴露的端口随机映射到49000~49900的端口。

用小写的p时我们可以指定映射那些端口，对映射关系进行详细操作，总共有4种指定的方式：

containerPort只指定需要映射的容器的端口，将他随机映射到一个宿主机的端口

``` bash
$ docker run –p 80 –i –t Ubuntu /bin/bash
```
hostport:containerport两者皆指定，

``` bash
$ docker run –p 8080:80 –i –t Ubuntu /bin/bash
```
ip:containerport

``` bash
$ docker run –p 0.0.0.0:80 –I –t Ubuntu /bin/bash
```
ip:hostport:containerport

``` bash
$ docker run –p 0.0.0.0:8080:80 –I –t Ubuntu /bin/bash
```
这样我们就可以通过docker的ps,inspect子命令查看到端口映射的情况了。

# 容器与容器之间的互联
首先，无论是同一台主机上的容器之间，还是跨主机的容器之间都可以通过访问对方容器所在的主机的映射的端口访问到对方容器的服务。

## 同一台主机上的容器的互联

### 通过ip+端口互联

容器有自己的网络和ip地址，使用docker inspect＋容器ID就可以查看到相关信息。默认情况之下，在同一台物理主机中的容器之间可以通过对方容器的ip和端口访问到对方的服务。但是这样有一个弊端：随着容器的停止与启动，容器的ip可能会发生变化。这就意味着每一次你重启了集群中的一个容器，你就必须要查看这个启动的容器新的ip，然后更改访问这个容器中服务的容器的配置。这显然太麻烦了。好在docker为我们提供了更加稳定可靠的方式来维护容器的连通。那就是：docker run的–link选项，

### 容器名便捷互访

#### 自定义容器名

虽然镜像启动后，容器会自动获得一个随机的容器名，但是为了方便和易记，我们往往会为容器自定义一个名字：

``` bash
$ docker run -itd --name myname ubuntu /bin/bash
```
如上，通过–name参数来指定容器的名称为myname.

#### 容器互联

使用–link参数能够让容器之间安全的进行交互。比如:
``` bash
$ docker run -d --name db mysql
```
上面我们以mysql镜像创建了名为db的容器，下面我们就让我们的应用程序容器与它链接起来：
``` bash
$ docker run -itd -P --name myweb --link db:mydb ubuntu /bin/bash
```
如上，我们以ubuntu镜像创建了一个名为myweb的容器，它通过–link参数与上面的db容器建立了容器名互访连接的关系，他在myweb容器中将被称之为mydb.
注意–link参数的格式：
``` bash
... --link <CONTAINER_NAME>:<ALIAS> ...
```
CONTAINER_NAME是所要链接的容器名，ALIAS是在容器中使用的所要链接容器的别名.
下面我们就可以在myweb容器中执行如下命令：

``` bash
$ ping mydb
```
可以发现可以顺利ping通。假如db容器的80端口还提供了静态网站服务，那么在myweb容器中可以通过如下命令访问到这一服务：

``` bash
$ curl mydb:80
```
注意：ping和curl这两个命令可能在容器中并没有安装，所以要先安装后才可以使用。
之后无论你怎样重启容器，只要对方容器还在运行，你就可以通过这种方式在源容器中访问到目标容器。

# 高级网络管理
## docker自带的网络

当我们安装docker的时候，会自动创建三个网络，我们可以使用下面的命令来查看主机中当前存在的网络：
``` bash
$ docker network ls
```
输出：

``` bash
NETWORK ID          NAME                DRIVER              SCOPE
5a0d02e77f3c        bridge              bridge              local
8349004e42e8        host                host                local
92e7ec631cff        none                null                local
```
NETWORK ID列是对应网络的唯一ID。NAME列是对应网络的名字。DRIVER是对应网络所使用的驱动。SCOPE是当前网络的作用域，这里的local说明当前网络作用域为本地。
上面的bridge,host,none就是docker安装时自动创建的三个网络。

### bridge

bridge网络是docker容器默认使用的网络，除非在容器启动时显式的指定使用其他网络，如下：
``` bash
$ docker run -itd --name ct1 ubuntu
$ docker run -itd --name ct2 --network host ubuntu
```
上面的两条命令用ubuntu镜像分别启动了两个容器ct1,ct2.其中ct1在默认的bridge网络中，ct2在我们指定的host网络中。可以通过docker inspect ct1和docker inspect ct2来查看当前容器所属的网络：
``` bash
$ docker inspect ct1
```
部分输出：
``` bash
"Networks": {   
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "5a0d02e77f3c89ec59b9432be752a35f6cc28c260022398751495ee563809570",
                    "EndpointID": "a60ca83c9da616064ca9fa112f80a05549bc2ff408483381979b6921bdba46ef",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.2",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:11:00:02"
                }
            }
```
对ct2执行同样的命令，同一部分的输出为：

``` bash
"Networks": {
                "host": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "8349004e42e8dda028d04b62a19df90ecf9655409ad0f1e8847a70f5b49fdd1f",
                    "EndpointID": "f3f957ca950edc0c12ab94e57d72d0799c8287ee638179ba63349cc87f27731e",
                    "Gateway": "",
                    "IPAddress": "",
                    "IPPrefixLen": 0,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": ""
                }
            }
```
由上可见，确实ct1,ct2两个容器分属于不同的网络。
bridge网络的底层就是docker0这一虚拟网络设备，当容器运行在bridge网络中时，docker的守护进程就是通过docker0来链接到这些容器的，并且这些容器支持通过ip来访问对方。其实在你安装docker以后，docker0这一网络设备就已经成为你的主机的网络栈的一部分。你可以通过如下命令查看到docker0:

``` bash
$ ip addr show
```
### host

运行在host网络中的容器，在网络方面与宿主机之间没有隔离，不用做端口映射，外界通过访问主机ip加对应端口访问到容器中的服务。直接使用 Docker host 的网络最大的好处就是性能，如果容器对网络传输效率有较高要求，则可以选择 host 网络。当然不便之处就是牺牲一些灵活性，比如要考虑端口冲突问题，Docker host 上已经使用的端口就不能再用了。Docker host 的另一个用途是让容器可以直接配置 host 网路。比如某些跨 host 的网络解决方案，其本身也是以容器方式运行的，这些方案需要对网络进行配置，比如管理 iptables。

### none

故名思议，none 网络就是什么都没有的网络。挂在这个网络下的容器除了 lo，没有其他任何网卡。封闭意味着隔离，一些对安全性要求高并且不需要联网的应用可以使用 none 网络。比如某个容器的唯一用途是生成随机密码，就可以放到 none 网络中避免密码被窃取。

## 自定义网络

### 定义网络

在实践中使用最多的网络是默认的bridge网络，其实我们还可以自定义网络：
``` bash
$ docker network create --driver bridge mynetwork
```
如上我以bridge为驱动创建了一个自定义网络mynetwork,我们可以通过下面的命令查看当前主机中的网络：

``` bash
$ docker network ls
```
这时我们就可以发现除了docker自带的bridge,none,host三种网络之外，我们的自定义网络mynetwork也出现在输出中。

### 使用自定义网络运行容器

下面我们使用默认的bridge运行ct1,ct2,使用我们的自定义网络再运行起来一个ct3容器：

``` bash
$ docker run -d --name ct1 ubuntu
$ docker run -d --name ct2 ubuntu
$ docker run -d --name ct3 --network mynetwork ubuntu
```
这样我们就得到了两个运行在bridge网络中的容器ct1,ct2，和一个运行在我自定义的网络mynetwork中的ct3.（可以使用docker inspect命令在Networks一项中查看到容器所属于的网络）

### 不同网络中容器的互访

我们可以在ct1,ct2,ct3中相互的ping对方，可以发现位于默认bridge网络中的ct1,ct2可以相互的ping通，但是，ct1,ct2和位于mynetwork中的ct3确无法ping通。这就说明在相同网络中的容器可以通过ip相互访问，但是跨网络则不行。那如果需要不同网络中的容器能够相互访问怎么办呢？可以使用下面的命令实现将某一个容器加入某一个网络中，那么这个容器就可以访问都该网络中的容器了

``` bash
$ docker network connect mynetwork ct2
```
如上，我们通过上述命令将ct2加入到了mynetwork网络中，这时我们可以查看一下ct2所属于的网络：
``` bash
$ docker inspect ct2
```
部分输出如下：
``` bash
"Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "5a0d02e77f3c89ec59b9432be752a35f6cc28c260022398751495ee563809570",
                    "EndpointID": "b138dd2a40e7e194fd3c055c78dc4db6778b80c767fbe32a68096507ac039fa5",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.3",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:11:00:03"
                },
                "mynetwork": {
                    "IPAMConfig": {},
                    "Links": null,
                    "Aliases": [
                        "455c3ec1534d"
                    ],
                    "NetworkID": "69d07788867935f590056da4d1c60591ca5459029715029e30c54f440b59b968",
                    "EndpointID": "afda2df82f371f3022e367e9b806c479d6ba06f5b6888b90c4bdcf2d48fffc8d",
                    "Gateway": "172.18.0.1",
                    "IPAddress": "172.18.0.3",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:12:00:03"
                }
            }
```
可见，ct2现在既属于bridge网络，又属于mynetwork网络。并且在bridge网络和在mynetwork网络中的ip不相同。在ct3中要访问ct2那么就需要使用ct2在mynetwork中的ip，同理要在ct1中访问ct2则要使用ct2在bridge网络中的ip.

### 查看某一网络中的容器

我们可以通过命令来查看某一网络中的所有的容器，如下我们查看bridge网络中的所有容器：

``` bash
$ docker network inspect bridge
```
输出：

``` bash
[
    {
        "Name": "bridge",
        "Id": "5a0d02e77f3c89ec59b9432be752a35f6cc28c260022398751495ee563809570",
        "Created": "2017-09-19T20:50:37.834660229+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "Containers": {
            "455c3ec1534dde68712fbb441e8fa97ed830430595d92a15ec7cbee801d22e39": {
                "Name": "ct2",
                "EndpointID": "b138dd2a40e7e194fd3c055c78dc4db6778b80c767fbe32a68096507ac039fa5",
                "MacAddress": "02:42:ac:11:00:03",
                "IPv4Address": "172.17.0.3/16",
                "IPv6Address": ""
            },
            "7b7063b9630d7945415063cf2dd0275414cb32cf5622c77ac27b96a10903ae57": {
                "Name": "ct1",
                "EndpointID": "111c9a1c47272f854457530fb8ee077947728144b3191b80c15e6223e865a9e5",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
```
从Containers一项中我们可以发现目前在bridge网络中的容器为ct1和ct2,这和目前的实际状况是一致的。

### 容器退出某一网络

我们也可以使一个容器退出某一网络，比如现在使ct2退出bridge:

``` bash
$ docker network disconnect bridge ct2
```
这时我们再次查看ct2的所属的网络就只有mynetwork了。

``` bash
$ docker inspect ct2
```
部分输出;

``` bash
"Networks": {
                "mynetwork": {
                    "IPAMConfig": {},
                    "Links": null,
                    "Aliases": [
                        "455c3ec1534d"
                    ],
                    "NetworkID": "69d07788867935f590056da4d1c60591ca5459029715029e30c54f440b59b968",
                    "EndpointID": "afda2df82f371f3022e367e9b806c479d6ba06f5b6888b90c4bdcf2d48fffc8d",
                    "Gateway": "172.18.0.1",
                    "IPAddress": "172.18.0.3",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:12:00:03"
                }
            }
```