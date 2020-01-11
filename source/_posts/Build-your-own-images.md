---
title: 构建镜像
comments: true
date: 2017-09-25 12:46:20
updated: 2017-11-16 12:46:20
tags: [Dockerfile]
categories: Docker
permalink:
---
构建docker镜像可以让我们保存对容器的修改并且可以再次使用，提供了自定义镜像的能力，使我们可以以软件的形式打包并分发服务及其运行环境.

# 通过容器构建镜像
可以通过docker的commit子命令基于一个容器创建一个新的镜像。
## commit命令
``` bash
$ docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]
```
## 指令说明

| OPTIONS |	USAGES |
| :--------: | :-----------: |
| -a,–author |	指定镜像的作者的信息(通常是作者的名字以及联系方式) |
| -m,–message |	记录信息，一般记录改变的信息 |
| -p,–pause |	是否停止容器 |
| -c,–change |	使用dockerfile的指令来配置生成的镜像 |

-c,–change选项是通过Dockerfile指令配置即将生成的镜像，支持的dockerfile指令有：CMD|ENTRYPOINT|ENV|EXPOSE|LABEL|ONBUILD|USER|VOLUME|WORKDIR

## 官网上的例子

``` bash
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS              NAMES
c3f279d17e0a        ubuntu:12.04        /bin/bash           7 days ago          Up 25 hours                            desperate_dubinsky
197387f1b436        ubuntu:12.04        /bin/bash           7 days ago          Up 25 hours                            focused_hamilton
$ docker commit c3f279d17e0a  svendowideit/testimage:version3
f5283438590d
$ docker images
REPOSITORY                        TAG                 ID                  CREATED             SIZE
svendowideit/testimage            version3            f5283438590d        16 seconds ago      335.7 MB
```
``` bash
$ docker ps
CONTAINER ID       IMAGE               COMMAND             CREATED             STATUS              PORTS              NAMES
c3f279d17e0a        ubuntu:12.04        /bin/bash           7 days ago          Up 25 hours                            desperate_dubinsky
197387f1b436        ubuntu:12.04        /bin/bash           7 days ago          Up 25 hours                            focused_hamilton
$ docker inspect -f "{{ .Config.Env }}" c3f279d17e0a
[HOME=/ PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin]
$ docker commit --change "ENV DEBUG true" c3f279d17e0a  svendowideit/testimage:version3
f5283438590d
$ docker inspect -f "{{ .Config.Env }}" f5283438590d
[HOME=/ PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin DEBUG=true]
```

``` bash
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS              NAMES
c3f279d17e0a        ubuntu:12.04        /bin/bash           7 days ago          Up 25 hours                            desperate_dubinsky
197387f1b436        ubuntu:12.04        /bin/bash           7 days ago          Up 25 hours                            focused_hamilton
$ docker commit --change='CMD ["apachectl", "-DFOREGROUND"]' -c "EXPOSE 80" c3f279d17e0a  svendowideit/testimage:version4
f5283438590d
$ docker run -d svendowideit/testimage:version4
89373736e2e7f00bc149bd783073ac43d0507da250e999f3f1036e0db60817c0
$ docker ps
CONTAINER ID        IMAGE               COMMAND                 CREATED             STATUS              PORTS              NAMES
89373736e2e7        testimage:version4  "apachectl -DFOREGROU"  3 seconds ago       Up 2 seconds        80/tcp             distracted_fermat
c3f279d17e0a        ubuntu:12.04        /bin/bash               7 days ago          Up 25 hours                            desperate_dubinsky
197387f1b436        ubuntu:12.04        /bin/bash               7 days ago          Up 25 hours                            focused_hamilton
```
事实上，官方不建议通过commit方式来创建新的镜像，这种创建方式是不透明的，你对新镜像做的操作无法记录下来，官方推荐的方式是使用 dockerfile创建镜像。

# 通过Dockerfile文件构建镜像
## 整体感受

Dockerfile是包含了一系列命令的文本文件。就像下面这样：
``` dockerfile
#First Dockerfile
FROM Ubuntu:14.04
MAINTAINER penname  hfsjhdss@outlook.com 
RUN apt-get update
RUN apt-get install –y nginx
EXPOSE 80
```
下面我们在桌面创建一个文件夹Test，在里面创建一个名为Dockerfile的文件，文件中的内容就是上面的例子中的内容。
然后使用$ docker build命令来开启镜像的创建：
``` bash
$ docker build –t="myfirstdockerfileimage" .
```
上面这条命令的意思是根据当前目录中的Dockerfile文件创建一个名为myfirstdockerfileimage的新镜像。当命令执行完毕之后，我们就可以使用docker images命令查看到我们新创建的镜像了。注意这条命令的最后一个选项是一个点，表示的是当前目录。下面我就详细的介绍一下Dockerfile文件各个指令的使用，以及docker的build子命令的使用和注意事项。
## Dockerfile主要指令

Dockerfile支持两种指令格式：注释和指令。注释是以#开头，后面是注释信息。指令是大写的指令名开头，后面是指令的参数。下面我们来看一下Dockerfile中的各个指令。

### FROM

通过FROM指令指定基础镜像。该镜像名必须是一个已经存在于本地的镜像。FROM必须是Dockerfile中第一条非注释指令，后续指令都是基于这个基础镜像执行。在Docker Hub上有非常多的高质量的官方镜像， 有可以直接拿来使用的服务类的镜像，如 nginx、redis、mongo、mysql、httpd、php、tomcat 等； 也有一些方便开发、构建、运行各种语言应用的镜像，如 node、openjdk、python、ruby、golang 等。 可以在其中寻找一个最符合我们最终目标的镜像为基础镜像进行定制。 如果没有找到对应服务的镜像，官方镜像中还提供了一些更为基础的操作系统镜像，如 ubuntu、debian、centos、fedora、alpine 等，这些操作系统的软件库为我们提供了更广阔的扩展空间。除了选择现有镜像为基础镜像外，Docker 还存在一个特殊的镜像，名为 scratch。这个镜像是虚拟的概念，并不实际存在，它表示一个空白的镜像。如果你以 scratch 为基础镜像的话，意味着你不以任何镜像为基础，接下来所写的指令将作为镜像第一层开始存在。不以任何系统为基础，直接将可执行文件复制进镜像的做法并不罕见，比如 swarm、coreos/etcd。对于 Linux 下静态编译的程序来说，并不需要有操作系统提供运行时支持，所需的一切库都已经在可执行文件里了，因此直接 FROM scratch 会让镜像体积更加小巧。使用 Go 语言 开发的应用很多会使用这种方式来制作镜像，这也是为什么有人认为 Go 是特别适合容器微服务架构的语言的原因之一。

### RUN

RUN 指令是用来执行命令行命令的。由于命令行的强大能力，RUN 指令在定制镜像时是最常用的指令之一。其格式有两种：

shell 格式：RUN <命令>，就像直接在命令行中输入的命令一样。刚才写的 Dockrfile 中的 RUN 指令就是这种格式。
exec 格式：RUN [“可执行文件”, “参数1”, “参数2”]，这更像是函数调用中的格式。
既然 RUN 就像Shell脚本一样可以执行命令，那么我们是否就可以像Shell脚本一样把每个命令对应一个RUN呢？比如这样：

``` dockerfile
FROM debian:jessie
RUN apt-get update
RUN apt-get install -y gcc libc6-dev make
RUN wget -O redis.tar.gz "http://download.redis.io/releases/redis-3.2.5.tar.gz"
RUN mkdir -p /usr/src/redis
RUN tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1
RUN make -C /usr/src/redis
RUN make -C /usr/src/redis install
```
之前说过，Dockerfile 中每一个指令都会建立一层，RUN 也不例外。每一个 RUN 的行为，就和刚才我们手工建立镜像的过程一样：新建立一层，在其上执行这些命令，执行结束后，commit 这一层的修改，构成新的镜像。而上面的这种写法，创建了 7 层镜像。这是完全没有意义的，而且很多运行时不需要的东西，都被装进了镜像里，比如编译环境、更新的软件包等等。结果就是产生非常臃肿、非常多层的镜像，不仅仅增加了构建部署的时间，也很容易出错。 这是很多初学 Docker 的人常犯的一个错误。

Union FS 是有最大层数限制的，比如 AUFS，曾经是最大不得超过 42 层，现在是不得超过 127 层。
上面的 Dockerfile 正确的写法应该是这样：
``` dockerfile
FROM debian:jessie
RUN buildDeps='gcc libc6-dev make' \
    && apt-get update \
    && apt-get install -y $buildDeps \
    && wget -O redis.tar.gz "http://download.redis.io/releases/redis-3.2.5.tar.gz" \
    && mkdir -p /usr/src/redis \
    && tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1 \
    && make -C /usr/src/redis \
    && make -C /usr/src/redis install \
    && rm -rf /var/lib/apt/lists/* \
    && rm redis.tar.gz \
    && rm -r /usr/src/redis \
    && apt-get purge -y --auto-remove $buildDeps
```
首先，之前所有的命令只有一个目的，就是编译、安装 redis 可执行文件。因此没有必要建立很多层，这只是一层的事情。因此，这里没有使用很多个 RUN 对一一对应不同的命令，而是仅仅使用一个 RUN 指令，并使用 && 将各个所需命令串联起来。将之前的 7 层，简化为了 1 层。在撰写 Dockerfile 的时候，要经常提醒自己，这并不是在写 Shell 脚本，而是在定义每一层该如何构建。并且，这里为了格式化还进行了换行。Dockerfile 支持 Shell 类的行尾添加 \ 的命令换行方式，以及行首 # 进行注释的格式。良好的格式，比如换行、缩进、注释等，会让维护、排障更为容易，这是一个比较好的习惯。此外，还可以看到这一组命令的最后添加了清理工作的命令，删除了为了编译构建所需要的软件，清理了所有下载、展开的文件，并且还清理了 apt 缓存文件。这是很重要的一步，我们之前说过，镜像是多层存储，每一层的东西并不会在下一层被删除，会一直跟随着镜像。因此镜像构建时，一定要确保每一层只添加真正需要添加的东西，任何无关的东西都应该清理掉。很多人初学 Docker 制作出了很臃肿的镜像的原因之一，就是忘记了每一层构建的最后一定要清理掉无关文件。

### EXPOSE

指定运行该镜像的容器使用的端口，可以一个expose后面跟多个端口，也可以使用多个expose:

``` bash
...
EXPOSE 80 8080
```

### CMD
``` bash
CMD[“executable”,”param1”,”param2”] (exec模式)
CMD command param1 param2 (shell模式)
CMD [“param1”,”param2”] (作为ENTRYPOINT指令的默认参数)
```
RUN指令指定的命令是在镜像构建时运行的，而CMD指定的命令是在容器运行时运行的，并且，如果我们在使用run来启动镜像对应的容器的时候，如果指定了容器启动运行的命令的话，CMD指令指定的命令将会被重写覆盖，而不会被执行，也就是说CMD指令是指定容器运行的默认行为．注意每个Dockerfile只能有一条CMD指令，如果指定了多条，则只有最后一条执行。

### ENTERYPOINT

同CMD指令一样，ENTRYPOINT指令也有exec和shell两种模式。在功能上，二者也很相似，两者的区别就是，ENTRYPOINT指令所指定的命令不会被docker run命令中指定的启动命令所覆盖，如果要覆盖，就需要在docker run命令中指定entrypoint选项，即:

``` bash
docker run –entrypoint……..
```
### LABLE

LABLE标签用来指定生成镜像的元数据标签信息，例如：

``` bash
LABLE version="1.0" description="this is a short description"
```
注意以前的表示作者信息的MAINTAINER指令已经被废弃，官方建议使用LABLE代替。

### COPY/ADD

ADD和COPY都是将文件和目录复制到使用dockerfile构建的镜像中，他们都支持两种参数，来源地址和目标地址。文件和目录的来源可以是本地地址，也可以是远程url。如果是本地地址，必须是构建目录中的相对地址，对于远程的，对于远程的url，docker并不推荐使用，更建议使用curl和wget之类的命令来获取文件。而目标路径需要指定镜像中的绝对路径。
这两个指令的区别：
ADD包含类似tar的解压功能，在一些安装tar软件包时会有所帮助。如果单纯复制文件，docker推荐使用COPY.。
例如：COPY inde.html /usr/share/nginx/html/ 把宿主机中与dockerfile同级目录下的index.html复制到镜像的/usr/share/nginx/html/下

### VOLUME

用来向基于镜像创建的容器添加卷，一个卷是可以存在一个或多个容器的特定目录，这个目录可以绕过联合文件系统，并提供如共享数据，对数据持久化的功能.

### WORKDIR

用来从镜像创建一个新容器时，在容器内部设置工作目录，ENTRYPOINT和CMD指定的命令都会在这个目录下执行。我们也可以使用这个指令在构建中为后续的指令指定工作目录，需要注意的是：WORKDIR一般使用绝对路径，如果使用相对路径，那么工作路径会一直传递下去，如：

``` bash
...
WORKDIR /a
WORKDIR b
WORKDIR c
RUN pwd
```
结果为 /a/b/c

### ENV

这个指令用于设置环境变量，环境变量的指令可以作用于构建过程中，在运行过程中同样有效。

### USER

用于指定镜像被什么样的用户去运行

### ONBUILD

可以为构建出来镜像添加触发器，当该镜像被其他镜像作为基础镜像时就会执行，及插入触发器中的指令

## 基于Dockerfile构建镜像

编写完成Dockerfile之后，可以通过docker build命令来创建镜像，该命令基本的格式如下：

``` bash
$ docker build [OPTIONS] 上下文路径
```
### 常用选项：

| OPTIONS |	USAGES |
| :-------: | :------------: |
| -t 　　　|　	指定最终生成的镜像的名称 |
| -f	   |如果使用非上下文路径下的Dockerfile，那么需要-f选项指定其路径 |
该命令会读取上下文路径下包括其子目录中的Dockerfile（或者是-f指定位置的Dockerfile）,并且将上下文路径下的内容发送给Docker服务端，由服务端来创建镜像。服务端会依据Dockerfile中的指令，从基础镜像开始运行一个容器,执行一条指令，对容器做出修改,执行类似于docker commit的操作，提交一个新的镜像层,再居于刚才提交的镜像运行一个新容器,执行Dockerfile中的下一条指令，直至所有的指令执行完毕。在根据Dockerfile构建镜像的过程中，会生成中间层镜像及其对应的容器，在这一过程中docker会自动删除生成的容器，而没有删除中间层镜像。这其实给了我们调试的能力，我们可以通过运行中间层镜像，来调试。

### 上下文(Context)是什么鬼？

首先我们要理解 docker build 的工作原理。Docker 在运行时分为 Docker 引擎（也就是服务端守护进程）和客户端工具。Docker 的引擎提供了一组 REST API，被称为 Docker Remote API，而如 docker 命令这样的客户端工具，则是通过这组 API 与 Docker 引擎交互，从而完成各种功能。因此，虽然表面上我们好像是在本机执行各种 docker 功能，但实际上，一切都是使用的远程调用形式在服务端（Docker 引擎）完成。也因为这种 C/S 设计，让我们操作远程服务器的 Docker 引擎变得轻而易举。当我们进行镜像构建的时候，并非所有定制都会通过 RUN 指令完成，经常会需要将一些本地文件复制进镜像，比如通过 COPY 指令、ADD 指令等。而 docker build 命令构建镜像，其实并非在本地构建，而是在服务端，也就是 Docker 引擎中构建的。那么在这种客户端/服务端的架构中，如何才能让服务端获得本地文件呢？
这就引入了上下文的概念。当构建的时候，用户会指定构建镜像上下文的路径，docker build 命令得知这个路径后，会将路径下的所有内容打包，然后上传给 Docker 引擎。这样 Docker 引擎收到这个上下文包后，展开就会获得构建镜像所需的一切文件。
如果在 Dockerfile 中这么写：
``` bash
... dockerfile
COPY ./package.json /app/
```
这并不是要复制执行 docker build 命令所在的目录下的 package.json，也不是复制 Dockerfile 所在目录下的 package.json，而是复制 上下文（context） 目录下的 package.json。因此，COPY 这类指令中的源文件的路径都是相对路径。这也是初学者经常会问的为什么 COPY ../package.json /app 或者 COPY /opt/xxxx /app 无法工作的原因，因为这些路径已经超出了上下文的范围，Docker 引擎无法获得这些位置的文件。如果真的需要那些文件，应该将它们复制到上下文目录中去。

如果观察 docker build 输出，我们其实可以看到了这个发送上下文的过程：
``` bash
$ docker build -t myimage .
```
输出：
``` bash
Sending build context to Docker daemon 2.048 kB
```
理解构建上下文对于镜像构建是很重要的，避免犯一些不应该的错误。比如有些初学者在发现 COPY /opt/xxxx /app 不工作后，于是干脆将 Dockerfile 放到了硬盘根目录去构建，结果发现 docker build 执行后，在发送一个几十 GB 的东西，极为缓慢而且很容易构建失败。那是因为这种做法是在让 docker build 打包整个硬盘，这显然是使用错误。

一般来说，应该会将 Dockerfile 置于一个空目录下，或者项目根目录下。如果该目录下没有所需文件，那么应该把所需文件复制一份过来。如果目录下有些东西确实不希望构建时传给 Docker 引擎，那么可以用 .gitignore 一样的语法写一个 .dockerignore，该文件是用于剔除不需要作为上下文传递给 Docker 引擎的。

那么为什么会有人误以为 . 是指定 Dockerfile 所在目录呢？这是因为在默认情况下，如果不额外指定 Dockerfile 的话，会将上下文目录下的名为 Dockerfile 的文件作为 Dockerfile。

这只是默认行为，实际上 Dockerfile 的文件名并不要求必须为 Dockerfile，而且并不要求必须位于上下文目录中，比如可以用 -f ../Dockerfile.php 参数指定某个文件作为 Dockerfile。

当然，一般大家习惯性的会使用默认的文件名 Dockerfile，以及会将其置于镜像构建上下文目录中。

### 构建缓存

由于每一步的构建过程都会将结果提交为镜像，docker会将之前的镜像看做缓存，如果我们再次运行下之前已经运行过的dockerfile的build命令，我们会发现，我们不会再次经历像第一次执行那样的长时间的等待，并且这一次运行的每一步都有Using cache的字样。这样就使我们的构建变得非常高效。但有时我们并不想使用缓存，如构建命令中包含apt-get update命令时，我们希望每次apt-get包都能刷新，这样就可以得到最新的版本。如果要跳过构建缓存，可以使用docker build –no-cache….即docker build命令中的—no-cache选项。

### 查看镜像构建过程
``` bash
$ docker history [image]
```
对于一个给定的镜像，我们可以通过上面的命令来查看其构建过程