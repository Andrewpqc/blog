---
title: 在k8s-dashboard中集成heapster
comments: true
date: 2018-04-25 21:26:38
updated: 2018-04-25 21:26:38
tags: [kubernetes,heapster]
categories: Kubernetes
permalink:
---
前面，我们在k8s集群中安装了kubernetes-dashboard插件，进入到了dashboard的界面中去了。但是我们查看kubernetes-dashboard的log,会发现有下面这样的记录：
``` 
2018/04/29 01:56:10 Getting config category
2018/04/29 01:56:10 Getting discovery and load balancing category
2018/04/29 01:56:10 Getting lists of all workloads
2018/04/29 01:56:10 No metric client provided. Skipping metrics.
2018/04/29 01:56:10 No metric client provided. Skipping metrics.
2018/04/29 01:56:10 No metric client provided. Skipping metrics.
2018/04/29 01:56:10 No metric client provided. Skipping metrics.
2018/04/29 01:56:10 No metric client provided. Skipping metrics.
2018/04/29 01:56:10 Getting pod metrics
2018/04/29 01:56:10 No metric client provided. Skipping metrics.
2018/04/29 01:56:10 No metric client provided. Skipping metrics.
2018/04/29 01:56:10 No metric client provided. Skipping metrics.
2018/04/29 01:56:10 [2018-04-29T01:56:10Z] Outcoming response to 10.32.0.1:49022 with 200 status code
2018/04/29 01:56:38 Metric client health check failed: the server could not find the requested resource (get services heapster). Retrying in 30 seconds.
```
从上面的log中我们大致可以得出如下的信息:当前系统中没有用于获取监控信息指标的客户端(metric client)，所以kubernetes-dashboard的处理方式是跳过这一步。同时，对metric client的健康检查失败了。这些问题不会导致dashboard无法工作，只是kubernetes-dashboard获取不到系统以及各个pod的监控数据。这里的解决方案就是安装另外一个k8s插件－－heapster.

# heapster简介
Heapster是容器集群监控和性能分析工具，天然的支持Kubernetes和CoreOS。
Kubernetes有个出名的监控agent---cAdvisor。在每个kubernetes Node上都会运行cAdvisor，它会收集本机以及容器的监控数据(cpu,memory,filesystem,network,uptime)。
Heapster是一个收集者，将每个Node上的cAdvisor的数据进行汇总，然后导到第三方工具(如InfluxDB)。
![](/images/heapster.png)

Heapster首先从K8S Master获取集群中所有Node的信息，然后通过这些Node上的kubelet获取有用数据，而kubelet本身的数据则是从cAdvisor得到。所有获取到的数据都被推到Heapster配置的后端存储中，并还支持数据的可视化。现在后端存储 + 可视化的方法，如InfluxDB + grafana。

下面我们就在k8s集群中配置安装heapster.
# 安装
我们仍然采用容器化的方式，依托于k8s集群本身的方式来安装heapster.
## 镜像准备
我们需要下面的三个镜像
```
heapster-influxdb-amd64:v1.3.3
heapster-grafana-amd64:v4.4.3
heapster-amd64:v1.4.0
```
本来这些镜像本来在gcr.io上，但是由于国内网络无法直接访问到。我们这里依然采用网友上传到docker hub上的镜像。使用下面的脚本即可:
``` bash
#!/bin/bash

images=(heapster-amd64:v1.4.0 heapster-influxdb-amd64:v1.3.3 heapster-grafana-amd64:v4.4.3)
for imageName in ${images[@]} ; do
  docker pull alleyj/$imageName
  docker tag alleyj/$imageName gcr.io/google_containers/$imageName
  docker rmi alleyj/$imageName
done
```
这样，我们的镜像就准备完了。下面我们就需要获得对应的manifest文件了。
## manifest文件准备
这里，我们一共需要4个manifest文件，这四个文件可以在[heapster的github仓库](https://github.com/kubernetes/heapster)获得。在目录`heapster/deploy/kube-config/influxdb/`下有三个yml文件，分别对应的是heapster、influxdb、grafana。另外还需要的一个yml文件是一个基于rbac的角色绑定文件，在`heapster/deploy/kube-config/rbac/`目录下。

这样我们就准备好了这些文件，置于heapster目录之下：
```
heapster-rbac.yaml
heapster.yaml
grafana.yaml
influxdb.yaml
```
## 安装
查看这些文件，看看所使用的镜像版本是否是你上面所下载的镜像的版本。确认无误之后运行下面的命令即可配置完成:
``` bash
$ kubectl create -f heapster/
```
这样，我们再次进入到k8s-dashboard就可以看到各种以图表形式展示的系统，各个pod的实时的监控数据了。