---
title: Docker数据管理
comments: true
date: 2017-09-20 09:23:01
updated: 2017-11-15 09:23:01
tags: [Docker Data Manage]
categories: Docker
permalink:
---
生产环境中使用docker，往往需要对数据进行持久化，或者在多个容器之间进行数据共享，这必然涉及容器的数据管理操作。容器中管理数据主要有两种方式：

- 数据卷
- 数据卷容器

# 数据卷
## 什么是数据卷？

数据卷是经过特殊设计的目录，可以绕过联合文件系统，为一个或多个容器提供访问。数据卷设计的目的，在于数据的永久化，它完全独立于容器的生存周期，因此docker不会在容器删除时删除其挂载的数据卷，也不会存在垃圾收集机制，对容器引用的数据卷进行处理.

![数据卷示意图](/images/datavolume.jpg)

上图可以告诉我们：
- docker的数据卷是独立于docker的存在，他存在于dockerhost也就是宿主机中，因此他与docker容器的生存周期是分离的。
- docker数据卷本质上是存在于docker宿主机的文件系统中
- docker数据卷可以是目录也可以是文件
- docker容器可以利用数据卷的技术与宿主机进行数据共享
- 同一个目录或文件可以支持多个容器的访问，这样其实实现了容器间数据的共享和交换。

## 数据卷的特点

- 数据卷在容器启动时初始化，如果容器使用的镜像在挂载点包含了数据，这些数据会可拷贝到新初始化的数据卷中。
- 数据卷可以在容器之间共享和重用
- 可以对数据卷中的内容直接进行修改
- 数据卷的变化不会影响镜像的更新
- 卷会一直存在，即使挂载数据卷的容器已经删除

## 创建数据卷
### docker volume子命令

docker 1.9引入了新的子命令docker volume,用户可以使用这个命令创建查看和删除数据卷，与此同时，传统的-v参数创建volume的方式也得到了保留。
``` bash
Usage:	docker volume COMMAND
Options:
      --help   Print usage
Commands:
  create      Create a volume
  inspect     Display detailed information on one or more volumes
  ls          List volumes
  prune       Remove all unused volumes
  rm          Remove one or more volumes
```
创建
``` bash
$ docker volume create --name volume1
```
在用户使用docker创建volume的时候，采用的是默认的local volumedriver,所以volume的文件系统默认使用宿主机的文件系统，如果用户需要创建其他文件系统的volume，则需要使用其他的volumedriver.

docker在创建volume的时候会在宿主机的/var/lib/docker/volume/中创建一个以volume ID为名的目录，并且将volume中的内容存储在名为_data的目录下。

使用docker volume inspect <volume name>命令可以获得该volume包括其在宿主机中该文件夹的位置等信息。
``` bash
$ docker volume inspect volume1
```
输出:
``` bash
[
    {
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/volume1/_data",
        "Name": "volume1",
        "Options": {},
        "Scope": "local"
    }
]
```
### 为容器添加数据卷

用户在使用docker run或者docker create创建新容器的时候，可以使用-v选项为容器添加volume(可以多次使用-v选项，为容器添加多个数据卷),用户可以将自行创建或者由docker创建的的volume挂载到容器中，也可以将宿主机上的目录或者文件作为volume挂载到容器中。
#### 宿主机的目录或文件作为数据卷
``` bash
$ docker run  –v  ~/datavolume:/data –it Ubuntu /bin/bash
```
-v选项就可以为启动的容器添加数据卷，~/datavolume为宿主机中的目录（这里也可以是一个文件，但是不管值文件夹还是文件都必须使用绝对路径），如果当前宿主机中没有这样的目录，那么容器启动时会自动在当前创建这个目录，这里的话，就是在当前用户的家目录下创建名为datavolume的目录。冒号后面的/data就是对应的容器中的目录。

我们如果在启动的容器的/data目录创建一个文件，并且在文件内写上一些内容。然后退出容器，查看宿主机的~/datavolume目录，则会发现，其中的内容就和我们在容器的/data目录中创建的内容一毛一样。也就是说两者之间的数据实现了同步的共享。
#### 随机名字的数据卷
``` bash
$ docker run -d -v /data ubuntu /bin/bash
```
以上的命令创建了一个随机名字的volume,并挂载到容器中的/data目录。要想知道它所挂载的数据卷在宿主机中的位置，可以通过docker volume inspect <volume name>查看。
#### 指定名字的数据卷
``` bash
$ docker volume create --name volume1
$ docker run -d -v volume1:/data ubuntu /bin/bash
```
以上两条命令首先是创建了一个名为volume1的数据卷，然后将其挂载在容器的/data目录。如果不执行第一条命令，直接执行第二条命令的话，docker会代替用户创建名为volume1的volume，并且将其挂载在容器中的/data目录。
#### 为数据卷添加访问权限
``` bash
$ docker run  –v  ~/datavolume:/data:ro Ubuntu  /bin/bash
```
在-v选项的参数后面添加了一个：ro(read only),这样的话就让这个数据卷变得只读了。
注：在退出容器后，可以用docker的inspect命令来查看数据卷的情况，以及读写权限
### 在dockerfile中创建数据卷

VOLUME命令
如：
``` dockerfile
#添加多个数据卷
VOLUME[“/datavolume1”,”/datavolume2”]
#添加一个数据卷
VOLUME /date
```
上面的命令所构建的镜像在创建容器时就会在容器的根目录挂载对应数据卷，并且在宿主机的文件系统中随机创建目录分别与容器内的数据卷对应。

与docker run -v不同的是VOLUME不能挂载主机中指定的文件夹，这主要是为了Dockerfile的可移植性。因为不能保证所有的宿主机都有对应的文件夹。

可以用docker inspect命令来查看宿主机上与这两个数据卷对应的目录。但是每次用此镜像运行的容器中的数据卷对应的宿主机的目录是不相同的。这样的话这个镜像所构建的容器们之间就无法进行数据共享。这就有了docker的数据卷容器。
# 数据卷容器

![数据卷容器示意图](/images/datavolume2.jpg)
## 什么是数据卷容器？

一个容器挂载数据卷，其他容器通过挂载这个容器实现数据共享，前面的挂载数据卷的容器就叫做数据卷容器。
## 实验
``` dockerfile
#data volume container experience image
FORM Ubuntu:latest
VALUME[“/datavolume1”,”/datavolume2”]
CMD[“/bin/bash”]
EXPOSE 80
```
如上，我们创建了这样的dockerfile，我们再根据这一dockerfile构建一个镜像image1
然后：
``` bash
$ docker run -it --name ct1 image1
```
我们根据这个镜像创建了一个容器ct1，由上面的内容可知ct1内挂载了两个数据卷/datavolume1,/datavolume2.并且现在docker宿主机上也有两个目录与这两个数据卷相对应。下面我们以ct1为数据卷容器在运行两个容器：
``` bash
$ docker run -it --volumes-from:ct1 --name ct2 ubuntu /bin/bash	
$ docker run -it --volumes-from:ct1 --name ct3 ubuntu /bin/bash
```
这样ct1，ct2，ct3就都拥有了数据卷，他们之间包括和宿主机之间就可以实现数据的共享。并且容器创建好之后，我们可以删除此中的数据卷容器ct1，那么ct2和ct3中的数据卷仍然可以正常使用。虽然数据卷容器在这个容器集群的构建中扮演着重要的角色，但是他只是进行了配置的传递，而容器ct1本身却不起作用，所以即使删除ct1，其他容器可以照样使用。

事实上，当这个集群建好之后，我们删除任意的一个容器，都不会对数据卷产生任何影响。即使删除了所有的挂载的容器，包括ct1,ct2,ct3，数据卷也不会被自动删除，如果要删除一个数据卷，必须在删除最后一个还挂载着它的容器时显式使用docker rm -v　CONTAINER_ID命令来指定。
## 删除数据卷

如果创建容器时从容器中挂载了volume，在/var/lib/docker/volumes/下会生成与volume对应的目录，使用docker rm删除容器并不会删除与volume对应的目录，这些目录会占据不必要的空间。即使可以手动删除，但是由于这些随机生成的目录名称是无意义的随机字符串，要知道他们是否与被删除的容器对应也是十分麻烦的。所以在删除容器时需要对容器中的volume妥善处理。在删除容器时，一并删除volume有一下几种方法：
``` bash
#对于自己命名的volume,可以直接像下面这样删除即可,这里只有当没有任何容器使用该volume的时候才会删除成功
$ docker volume rm <volume name>
#在删除容器时，一并删除其volume(推荐使用)
$ docker rm -v <container_name>
#在运行容器时使用`docker run --rm`,--rm选项会在容器停止运行时删除容器以及容器所挂载的volume
```
## 容器与宿主机的数据交换

使用docker cp命令。
``` bash
Usage:	docker cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH|-
	docker cp [OPTIONS] SRC_PATH|- CONTAINER:DEST_PATH
Copy files/folders between a container and the local filesystem
Options:
  -L, --follow-link   Always follow symbol link in SRC_PATH
      --help          Print usage
```
上述的docker cp命令只适合纯粹的在容器与宿主机之间交换数据，但是它的灵活性比volume小得多，它不支持数据在容器和宿主机之间的同步。