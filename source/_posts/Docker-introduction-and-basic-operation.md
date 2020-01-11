---
title: Docker简介,安装及镜像基本操作
comments: true
date: 2017-08-24 17:05:31
updated: 2017-11-14 17:05:31
tags: [install Docker,Docker images]
categories: Docker
permalink:
---
![docker](/images/docker.jpeg)
# Docker简介
Docker是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的容器中，然后发布到任何流行的 Linux 机器上.Docker容器完全使用沙箱机制，相互之间不会有任何接口。使用docker容器可以实现开发，测试，部署环境的一致化，缩短产品的开发周期。Docker是以Docker容器为资源分割和调度的基本单位，封装整个软件的运行时环境，为开发者和系统管理员设计的，用于构建，发布和运行分布式应用的平台。



docker是按照C/S架构设计而成，由docker客户端（docker client）,docker守护进程（docker doamonn,充当着server的角色，响应docker客户端的指令），docker　registry组成。docker里面还有两个重要的概念：镜像（image）与容器(container)。镜像是容器的基础。而docker的容器技术是基于Linux内核的namespace和cgroups实现的。两者分别实现了资源的隔离和限制。镜像存放在registry(注册服务器)中。
# Docker安装
## 查看系统情况
``` bash
$ uname -r
```
Docker要求Ubuntu系统的内核版本高于３．１０。只有满足这一条件方可继续下面的步骤。

## 安装
``` bash
$ wget -qO- https://get.docker.com/ | sh
```
上面的命令会下载并安装最新版本的docker

## 查看是否安装成功
``` bash
$ sudo docker run hello-world
```
如果docker安装成功，上面的命令就会在docker　hub的公共仓库中去拉取hello-world这个镜像，并且启动镜像，生成一个容器，执行容器中预设的命令，然后立即退出容器。如果在这条命令的输出结果中出现的是关于docker的相关说明的话，那么你就安装成功了。注意：这里我们一定要以超级用户的身份运行这一命令，如果运行上述命令时漏掉了sudo,则会报下面的错误：docker:Connot not connect to the docker deamon.Is the docker deamon runing on this host? See ”docker run –help”，要想解决这个问题，请往下看。

## 在Docker用户组中添加自己
默认情况下，我们在执行docker命令的时候都要开启超级用户的权限，也就是说，每条命令都要输入sudo,这显然太麻烦了。好在docker在安装的时候给我们的系统中建了一个名为docker的用户组，只要是用户组中的用户，就可以直接使用docker了。方法是运行下面的这条命令：
``` bash
$  sudo usermod -aG docker <your name>
```
然后重启docker服务（$ sudo servire docker restart），注销,重新登录系统，这样以后输入dcoker命令就不用每次都输入sudo了。
## 配置镜像加速服务
现在我们已经完全安装好了docker,但是如果要想愉快的使用docker的话，还需要配置镜像加速服务。默认情况下，我们在拉取镜像时是在docker　hub这一个仓库中拉取的，由于docker hub仓库在国外，所以在国内拉取镜像时，速度及其的慢（翻墙除外）。好在国内有docker hub的镜像，我们可以通过简单的配置docker守护进程的启动选项即可免费使用这一镜像加速服务了。具体的配置就不讲了，因为在daocloud的网站上有配置脚本，并且提供了获取和启动脚本的命令，大家只需要到daocloud的网站上复制下这条命令，然后粘贴到终端并回车即可配置好，之后我们拉取镜像就用的是国内的镜像仓库，不用翻墙速度也超快。
下面的是我在daocloud上复制下的linux版本的加速命令：
``` bash
$ curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s
```
大家在配置的时候最好不要直接抄我这个，还是自己到daocloud网站上去复制吧，在加速器中。因为还存在版本的升级问题。
# Docker镜像基本操作
Docker镜像是一个只读的Docker容器模板，含有启动Docker容器所需文件系统结构及其内容，因此是启动Docker容器的基础。docker镜像的文件内容以及一些运行Docker容器的配置文件组成了Docker容器的静态文件系统运行环境——rootfs。可以这么理解，Docker镜像是Docker容器的静态视角，Docker容器是Dokcer镜像的运行状态。
## 获取镜像
获取镜像的方式有两种：
１．在docker hub或某个私有镜像仓库中拉取镜像(即下载镜像到本地)。
２．自己在基础镜像的基础上构建镜像。
由于即使是自己构建镜像也需要一个镜像作为基础镜像，而这个基础镜像一般是从docker hub的公共仓库中拉取的。所以我们在这里讲如何在docker hub中拉取镜像。
``` bash
$ docker pull [OPTIONS] NAME [:TAG]
```
常用选项OPTIONS:-a,--all-tags 会下载所有此仓库中的打了标签的镜像
如果拉取镜像的速度极慢，那么请参考docker的安装及基本概念中的镜像加速部分。

## 搜索镜像
其实我们一般在拉取镜像之前，会先以我们想要的镜像名字做一个搜索，然后更具搜索出来的信息来决定我们想要拉取的镜像。搜索的信息一般会包括这个镜像的星级，镜像是否为官方镜像以及此镜像的一个简短的描述。这些信息都可以作为我们拉取镜像的一个参考。
``` bash
$ docker search [OPTIONS] TERM
```
常用选项OPTIONS: --automated 只显示自动化构建出的镜像
         --no-trunc    不使用截断输出
         -s,--stars=0  规定显示结果的最低星级

由于构建镜像是docker中的一个非常重要的一部分，所以我在后面会专门写一篇博客来介绍。

## 推送镜像
我们不仅可以构建自己的镜像，还可以把我们构建的有适用性的高质量的镜像推送到docker hub上，贡献给社区的其他小伙伴(public)，或者自己保存(private),以便以后接着使用。说明:dockerhub的共公镜像存储服务是完全免费的。但仅支持每个用户一个私有仓库，如果你想保存更多的私有镜像的话，就可以使用docker hub的付费服务。在推送镜像之前，必须要有docker hub的账号。

命令行登录：
``` bash
$ docker login -p <docker hub pwd > -u <docker hub username>
```
或者
``` bash
$ docker login
```
下面会提示你输入用户名和密码。
登录成功之后就可以推送自己的镜像了：
``` bash
$ docker push NAME[:TAG]
```
成功之后，你可以登录docker hub,就可以看到这个镜像了，默认他是公开的，你可以把他转为私有。如果公开的话，你还可以在命令行搜索到你的镜像。当然，社区的其他人也可以搜索到。

这条命令并不会把整个镜像都提交上去，而是提交你在基础镜像的基础上所作出的修改的那一部分。

## tag命令添加镜像标签
为了方便在后续工作中使用特定的镜像，可以使用docker tag命令来为本地镜像任意添加新的标签，例如将ubuntu:latest打上标签：
``` bash
$ docker tag ubuntu:latest myubuntu:latest
```
当我们使用docker images列出本地主机上的镜像信息的时候，可以看到多了一个拥有myubuntu:latest标签的镜像，之后我们就可以直接使用myubuntu:latest来表示这个镜像了。尝试一下就可以发现，打上新标签的镜像与原来的镜像的id是一样的，加入我们要删除原镜像ubuntu:latest,那么myubuntu:latest也就跟着不见了。事实上打上新的标签并没有复制一个新的镜像，实际上只是给原镜像去了一个别名而已，他们实际指向的是同一个镜像，docker tag命令添加的标签实际上起到了类似链接的作用。

## 镜像历史
镜像文件由多个层组成，我们可以通过history子命令来查看各个层的创建信息
``` bash
$ docker history ubuntu:latest
```
## 导出镜像
如果要导出镜像到本地文件，可以使用docker save命令，例如：导出本地的ubuntu:latest镜像为文件ubuntu_latest.tar,使用如下命令：
``` bash
$ docker save -o ubuntu_latest ubuntu:latest
```
之后就可以通过复制ubuntu_latest.tar文件将该镜像分享给别人。
## 载入镜像
可以使用docker load将导出的tar文件再导入到本地镜像库，例如从文件ubuntu_latest.tar导入镜像到本地镜像列表，如下所示：
``` bash
$ docker load --input ubuntu_latest.tar
或者：
$ docker load < ubuntu_latest.tar
```
这将导入镜像及其相关的元数据信息(包括标签等)

## 查看镜像
``` bash
$ docker images [OPTIONS] [REPOSITORY]
```
上面的命令会列出我们系统中已经有的镜像，有下面这些条件：
　　　　　OPTIONS: -a,–all=false 默认为false,为true时会显示所有的镜像，包括中间层镜像
-f,–filter=[] 默认不过滤某个镜像
-q,–quit=false 为true时只显示镜像的唯一ＩＤ
REPOSITORY:　仓库名，制定要显示那个仓库的镜像
## 查看镜像的详细信息
``` bash
$ docker inspect [OPTIONS] IMAGE [IMAGE2.....]
```
docker的inspect命令可以输出所查看镜像的完整的详细的信息。其实这个命令也可以用来查看容器的信息。
## 删除镜像
``` bash
$ docker rmi [OPTIONS] IMAGE [IMAGE2….]
```
个选项会删除所指定的一个或多个镜像
OPTIONS: -f,–force 强制删除镜像
有时候我们想要删除某个仓库中的所有镜像，如过使用上面的这条命令的话就太麻烦了，这是我们可以使用下面这条命令：
``` bash
$ docker rmi $(docker images –q ubuntu)
```
上面的命令删除了ubuntu仓库中的所有镜像，$ docker images -q ubuntu 这条命令就会输出ubuntu仓库中的所有镜像的唯一id,而输出结果更好作为docker rmi命令的参数了。

# 正确区分registry和repository
Repository：仓库，一个repository里面存放的着相同种类不同标签的镜像，比如Tomcat下面有很多个版本的镜像，它们共同组成了Tomcat的Repository。粗浅的理解：比如一个基础镜像ubuntu分别制作成了各种不同的功能，而打上了不同的标签，这样每一个镜像就是不同的镜像，而他们都共同组成了ubuntu的repository

Registry：镜像注册服务器，比如DockerHub网站就是一个registry,Registry上有很多的Repository，Redis、Tomcat、MySQL等等Repository组成了Registry。

