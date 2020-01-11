---
title: 终端翻墙备忘录
comments: true
date: 2018-04-30 10:17:09
updated: 2018-04-30 10:17:09
tags: [privoxy,polipo]
categories: Linux
permalink:
---
这几天要开始用GO写东西了，但是由于GFW的存在，在终端中有一些包用`go get`无法下载到。虽然本人有可以使用的ssr服务(服务默认跑在本地的1080端口)，但是ssr使用的是socks5代理。在命令行终端下，ssr无法正常工作。因为在终端下wget、curl、git、brew等命令行工具使用的都是http协议。这样的话，我们需要做就是将socks5代理转成http代理。下面就介绍两个常用的转换的工具。
# socks5转http
## privoxy
>Privoxy is a non-caching web proxy with advanced filtering capabilities for enhancing privacy,modifying web page data and HTTP headers, controlling access, and removing ads and other obnoxious Internet junk. Privoxy has a flexible configuration and can be customized to suit individual needs and tastes. It has application for both stand-alone systems and multi-user networks.

### 安装
``` bash
sudo apt-get install privoxy
```
### 配置
在`/etc/privoxy/config`文件中添加如下的一条配置：
``` 
forward-socks5 / 127.0.0.1:1080 .
```
这条配置的意思是将到达privoxy的http流量以socks5的形式转发至本地的1080端口(ssr服务跑在该端口)，将从本地1080端口传回的socks5流量以http形式转发至终端程序.(这句话完全是我自己的理解)
### 重启并确认
因为更改了配置文件，所以要重启provixy服务：
``` bash
$ service privoxy restart
```
确认privoxy服务已经启动：
``` bash
$ service privoxy status
```
privoxy服务默认跑在本地的8118端口。

## polipo
>Polipo is single-threaded, non blocking caching web proxy that has very modest resource needs. 

### 安装
``` bash
sudo apt-get install polipo
```
### 配置
在`/etc/polipo/config`文件中做如下的配置：
``` 
# This file only needs to list configuration variables that deviate
# from the default values.  See /usr/share/doc/polipo/examples/config.sample
# and "polipo -v" for variables you can tweak and further information.

logSyslog = true
logFile = /var/log/polipo/polipo.log

proxyAddress = "0.0.0.0"

socksParentProxy = "127.0.0.1:1080"
socksProxyType = socks5

chunkHighMark = 50331648
objectHighMark = 16384

dnsQueryIPv6 = no
```
### 重启并确认
因为更改了配置文件，所以要重启polipo服务：
``` bash
$ service polipo restart
```
确认polipo服务已经启动：
``` bash
$ service polipo status
```
polipo服务默认跑在本地的8123端口。


# 终端设置
当我们把协议转换服务架设好之后，下一步要做的就是在终端中设置代理了.
## 当前终端设置代理
``` bash
export http_proxy=`http://127.0.0.1:8118`
export https_proxy='http://127.0.0.1:8118'
```
注意上面使用的是privoxy的端口，如果是polipo则改为对应的端口。下同。
我们在当前终端中设置上面的两个环境变量即可设置代理。但是这种方式只在当前终端有效，当新启动一个终端后就需要重新设置。

## 在.bashrc文件中设置代理
我们可以在当前用户的`.bashrc`文件中设置如下的两个alias：
``` bash
alias proxy="export http_proxy=http://localhost:8118;export https_proxy=http://localhost:8118" 
alias unproxy="unset http_proxy;unset https_proxy"
```
让配置立即生效:
``` bash
source .bashrc
```
我们在终端中通过运行`proxy`命令来启用终端代理，运行`unproxy`就不用代理。这样就可以在代理与非代理之间切换自如了。