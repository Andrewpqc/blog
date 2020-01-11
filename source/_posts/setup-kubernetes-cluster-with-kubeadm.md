---
title: 用Kubeadm搭建一个k8s 1.7.5的单节点集群
comments: true
date: 2018-04-23 22:09:21
updated: 2018-04-26 17:40:21
tags: [kubernetes,kubeadm]
categories: [Kubernetes]
permalink:
---
[团队](http://www.muxixyz.com)近期打算自己开发一个基于kubernetes的应用部署、监控、维护、管理的云平台MAE(Muxi APP Engine)，思路来自于赵老板的一篇[博客](http://zxc0328.github.io/2017/05/27/mae/#more)。可以让用户直接在可视化的环境下完成应用的部署、集群维护的一个服务。开发首先得需要一个kubernetes集群，这几天一直在弄kubernetes的东西。今天就先总结一下集群搭建的基本过程，记录一下搭建过程中遇到的坑，希望能够给后来者以参考。

# 基本环境
由于是学生团队，搭建过程基于的环境比较简陋：
- 阿里云学生机一台，CentOS 7.4 ,64位的操作系统
- CPU:1核
- 内存：2G
- 带宽：1Mbps

# 前期准备
## 节点命名
阿里云的机器的hostname默认都是一长串由字母和数字组成的字符串，没有什么规律。为了便于以后的管理，我们可以使用[hostnamectl命令](https://access.redhat.com/documentation/zh-cn/red_hat_enterprise_linux/7/html/networking_guide/sec_configuring_host_names_using_hostnamectl)来给我们的节点取一个具有语义性的名字。具体操作,在root用户下运行下面的命令：
``` bash
$ hostnamectl set-hostname "k8s-master"
```
因为我们这里只有一台主机，所以就给它命名成k8s-master.

## 禁用防火墙
如果各个主机启用了防火墙，需要开放Kubernetes各个组件所需要的端口，可以查看Installing kubeadm中的[Check required ports](https://kubernetes.io/docs/setup/independent/install-kubeadm/#check-required-ports)一节。 这里简单起见在各节点禁用防火墙：
``` bash
$ systemctl stop firewalld
$ systemctl disable firewalld
```
创建/etc/sysctl.d/k8s.conf文件，添加如下内容：
```
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
```
执行`sysctl -p /etc/sysctl.d/k8s.conf`使修改生效。

## 禁用SELINUX
``` bash
$ setenforce 0
```
``` bash
vim /etc/selinux/config
```
修改成这样：
```
SELINUX=disabled
```
修改`/etc/selinux/config`文件的过程也可以直接这样:
``` bash
$ sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
```

## 关闭系统的Swap
Kubernetes 1.8开始要求关闭系统的Swap。如果不关闭，默认配置下kubelet将无法启动。可以通过kubelet的启动参数`--fail-swap-on=false`更改这个限制。 我们这里关闭系统的Swap:
``` bash
$ swapoff -a
```
修改`/etc/fstab`文件，注释掉`SWAP`的自动挂载，使用`free -m`确认swap已经关闭。
`swappiness`参数调整，修改/etc/sysctl.d/k8s.conf添加下面一行：
``` 
vm.swappiness=0
```
执行sysctl -p /etc/sysctl.d/k8s.conf使修改生效。

# 安装并配置docker
## 安装
docker属于kubernetes的基础设施,一般来说docker 1.12是比较稳定的，我们要在阿里云学生机上下载docker需要先添加docker的镜像源，像下面这样:
``` bash
$ cat >/etc/yum.repos.d/docker.repo <<EOF
[dockerrepo]
name=Docker Repository
baseurl=https://yum.dockerproject.org/repo/main/centos/7
enabled=1
gpgcheck=1
gpgkey=https://yum.dockerproject.org/gpg
EOF
```
然后通过下面的命令将服务器上的软件包信息先在本地缓存,以提高搜索安装软件的速度：
```
$ yum makecache
```
现在你可以通过下面的命令来查看你新添加的软件源中可以安装的docker的版本：
``` bash
$ yum list docker-engine showduplicates
```
这里我们选择安装docker 1.12.6
``` bash
$ yum install docker-engine-1.12.6-1.el7.centos.x86_64
```
启动docker:
``` bash
$ systemctl enable docker
$ systemctl start docker
```
## 配置
下面我们需要到`docker　hub`镜像仓库中拉取镜像，为了加快拉取镜像的速度，我们在这里给`docker`配置一下加速器。国内的阿里云、daoclode、时速云等均有免费的docker加速器提供，大家可以自行google。这里我采用阿里云的镜像加速服务。因为在下面的kubeadm初始化master节点的时候，需要保证docker的`cgroup driver`类型与`kubelet`启动时使用的`cgroup driver`一致(`cgroupfs`或者`systemd`),所以这里也一并展示一下配置docker的`cgroup driver`的方法：
``` bash
mkdir -p /etc/docker
cat > /etc/docker/daemon.json <<EOF
{
  "registry-mirrors": ["https://sw9esv3f.mirror.aliyuncs.com"]，
  "exec-opts": ["native.cgroupdriver=systemd"]
}
EOF
```
因为修改了配置文件，所以这里需要重启一下docker.
``` bash
systemctl daemon-reload
systemctl restart docker
```
# 安装k8s相关镜像及组件
k8s集群分为master节点和node节点，master节点主要用于对于集群进行管理。k8s安装一般有两种安装方式，第一种为官方提供的工具kubeadm安装，第二种为二进制文件安装，此处主要介绍第一种
## 相关镜像下载
由于使用kubeadm在安装的过程中会使用一些谷歌开源的镜像，但是国内无法访问到gcr.io，所以一般情况下我们需要翻墙拉取镜像，但是有以下前辈已经拉取到我们所需的镜像，并且传到了dockerhub上，我们可以直接使用这上面的镜像。并且刚才配置了加速器，所以拉取的速度也是很快的。这里就推荐两个docker hub上的仓库:[alleyj](https://hub.docker.com/search/?isAutomated=0&isOfficial=0&page=1&pullCount=0&q=alleyj&starCount=0),[mirrorgooglecontainers](https://hub.docker.com/u/mirrorgooglecontainers/)
这里以alleyj的仓库为例，先需要从这个仓库里pull我们需要的镜像，然后使用`docker tag`命令将镜像的前缀改为`gcr.io/google_containers`,最后使用`docker rmi`去掉之前的镜像记录。可以使用如下脚本完成以上操作:
``` bash
#!/bin/bash
set -o errexit
set -o nounset
set -o pipefail

KUBE_VERSION=v1.7.5
KUBE_PAUSE_VERSION=3.0
ETCD_VERSION=3.0.17
DNS_VERSION=1.14.4

GCR_URL=gcr.io/google_containers
DOCKERHUB_URL=alleyj

images=(kube-proxy-amd64:${KUBE_VERSION}
kube-scheduler-amd64:${KUBE_VERSION}
kube-controller-manager-amd64:${KUBE_VERSION}
kube-apiserver-amd64:${KUBE_VERSION}
pause-amd64:${KUBE_PAUSE_VERSION}
etcd-amd64:${ETCD_VERSION}
k8s-dns-sidecar-amd64:${DNS_VERSION}
k8s-dns-kube-dns-amd64:${DNS_VERSION}
k8s-dns-dnsmasq-nanny-amd64:${DNS_VERSION})


for imageName in ${images[@]} ; do
  docker pull $DOCKERHUB_URL/$imageName
  docker tag $DOCKERHUB_URL/$imageName $GCR_URL/$imageName
  docker rmi $DOCKERHUB_URL/$imageName
done
```
kubernetes各个版本所需要的镜像版本可以参见－>[这里](https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-init/#running-kubeadm-without-an-internet-connection)
## 组件下载
这里我们还需要下载的组件是`kubelet`,`kubectl`,`kubeadm`,`kubernetes-cni`。首先添加yum源:
``` bash
cat >> /etc/yum.repos.d/kubernetes.repo <<EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=0
EOF
```
然后可以通过下面的命令来查看该镜像源中可以安装的对应组件的版本：
``` bash
$ yum list kubeadm showduplicates
$ yum list kubelet showduplicates
$ yum list kubectl showduplicates
$ yum list kubernetes-cni showduplicates
```
这里，我们选择下面的版本安装.
``` bash
yum install -y kubernetes-cni-0.5.1-0.x86_64 
yum install -y kubelet-1.7.5-0.x86_64 
yum install -y kubectl-1.7.5-0.x86_64 
yum install -y kubeadm-1.7.5-0.x86_64
```
启动kubelet.
``` bash
systemctl enable kubelet && systemctl start kubelet
```
## 配置kubelet
kubelet负责在节点上启动，终止，重启容器。我们需要对其进行响应的配置。
``` bash
cat > /etc/systemd/system/kubelet.service.d/20-pod-infra-image.conf <<EOF
[Service]
Environment="KUBELET_EXTRA_ARGS=--pod-infra-container-image=gcr.io/google_containers/pause-amd64:3.0"
EOF
```
上面配置的是kubelet的启动参数，指定了pause容器镜像。这个pause容器在pod中担任Linux命名空间共享的基础,启用pid命名空间，开启init进程。更多关于pause容器的内容，可以看－>[这里](https://jimmysong.io/posts/what-is-a-pause-container/)
在kubeadm初始化master节点的过程中，我们还需要设置两个环境变量:
``` bash
export KUBE_REPO_PREFIX="gcr.io/google_containers"
export KUBE_ETCD_IMAGE="gcr.io/google_containers/etcd-amd64:3.0.17"
```
这两个环境变量指定了kubernetes所需的系统镜像的前缀，和etcd镜像的完整名称。由于这些镜像现在都以gcr.io/google\_containers为前缀存在于本地，所以不用考虑说以gcr.io/google_containers为前缀拉取不到镜像的问题。

配置完毕，我们重新启动kubelet:
``` bash
systemctl daemon-reload
systemctl restart kubelet
```
# kubeadm初始化master节点
到目前为止，我们搭建单节点集群所需要的所有软件，镜像全部安装完毕。下面我们就是要使用kubeadm来初始化集群。这个地方就开始遇到各种坑了。首先，在`kubeadm init`之前我们需要**保证本机的`docker`和`kubelet`服务都处于运行状态**.所以我们先来确认一下这两个服务是否都跑起来了:
``` bash
$ systemctl status docker
$ systemctl status kubelet
```
如果你在运行上面的两个命令后看到的都是绿色，那么很幸运，你跳过了这个坑。不幸的是，笔者的kubelet一直无法跑起来。`systemctl status kubelet`的输出如下所示：
```
● kubelet.service - kubelet: The Kubernetes Node Agent
   Loaded: loaded (/etc/systemd/system/kubelet.service; enabled; vendor preset: disabled)
  Drop-In: /etc/systemd/system/kubelet.service.d
           └─10-kubeadm.conf, 20-pod-infra-image.conf
   Active: activating (auto-restart) (Result: exit-code) since 日 2018-04-22 10:26:38 CST; 1s ago
     Docs: http://kubernetes.io/docs/
  Process: 21169 ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_SYSTEM_PODS_ARGS $KUBELET_NETWORK_ARGS $KUBELET_DNS_ARGS $KUBELET_AUTHZ_ARGS $KUBELET_CADVISOR_ARGS $KUBELET_CGROUP_ARGS $KUBELET_EXTRA_ARGS (code=exited, status=255)
 Main PID: 21169 (code=exited, status=255)

4月 22 10:26:38 iZwz9f6pgul78p7die5tlzZ kubelet[21169]: --seccomp-profile-root string                         <Warning: Alpha feature> ...comp")
4月 22 10:26:38 iZwz9f6pgul78p7die5tlzZ kubelet[21169]: --stderrthreshold severity                            logs at or above this thr...ult 2)
4月 22 10:26:38 iZwz9f6pgul78p7die5tlzZ kubelet[21169]: -v, --v Level                                             log level for V logs
4月 22 10:26:38 iZwz9f6pgul78p7die5tlzZ kubelet[21169]: --version version[=true]                              Print version information and quit
4月 22 10:26:38 iZwz9f6pgul78p7die5tlzZ kubelet[21169]: --vmodule moduleSpec                                  comma-separated list of p...ogging
4月 22 10:26:38 iZwz9f6pgul78p7die5tlzZ systemd[1]: kubelet.service: main process exited, code=exited, status=255/n/a
4月 22 10:26:38 iZwz9f6pgul78p7die5tlzZ kubelet[21169]: --volume-plugin-dir string                            The full path of the dire...xec/")
4月 22 10:26:38 iZwz9f6pgul78p7die5tlzZ kubelet[21169]: F0422 10:26:38.470722   21169 server.go:145] unknown flag: --require-kubeconfig
4月 22 10:26:38 iZwz9f6pgul78p7die5tlzZ systemd[1]: Unit kubelet.service entered failed state.
4月 22 10:26:38 iZwz9f6pgul78p7die5tlzZ systemd[1]: kubelet.service failed.
```
其实，现在看来这个地方也不叫做坑，上面的输出倒数第3行已经说的很清楚了`unknown flag: --require-kubeconfig`。这说明kubelet的启动参数中出现了一个它不认的flag.(笔者装的时候一直在查看kubelet的状态，就是没有好好看log，导致这里花费了许多时间。这说明仔细看log真的很重要!!!)。下面我们就去看看kubelet启动参数的配置文件：
``` bash
$ cat /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
```
文件内容如下:
``` 
[Service]
Environment="KUBELET_KUBECONFIG_ARGS=--kubeconfig=/etc/kubernetes/kubelet.conf --require-kubeconfig=true"
Environment="KUBELET_SYSTEM_PODS_ARGS=--pod-manifest-path=/etc/kubernetes/manifests --allow-privileged=true"
Environment="KUBELET_NETWORK_ARGS=--network-plugin=cni --cni-conf-dir=/etc/cni/net.d --cni-bin-dir=/opt/cni/bin"
Environment="KUBELET_DNS_ARGS=--cluster-dns=10.96.0.10 --cluster-domain=cluster.local"
Environment="KUBELET_AUTHZ_ARGS=--authorization-mode=Webhook --client-ca-file=/etc/kubernetes/pki/ca.crt"
Environment="KUBELET_CADVISOR_ARGS=--cadvisor-port=0"
Environment="KUBELET_CGROUP_ARGS=--cgroup-driver=systemd"
ExecStart=
ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_SYSTEM_PODS_ARGS $KUBELET_NETWORK_ARGS $KUBELET_DNS_ARGS $KUBELET_AUTHZ_ARGS $KUBELET_CADVISOR_ARGS $KUBELET_CGROUP_ARGS $KUBELET_EXTRA_ARGS
```
果真，在KUBELET\_KUBECONFIG_ARGS中有一个参数`--require-kubeconfig=true`，去掉即可。
下面重新启动kubelet:
``` bash
systemctl daemon-reload
systemctl restart kubelet
```
查看kubelet的状态，发现kubelet仍然没有运行起来:<(
下面查看一下系统log,看看究竟发生了啥：
``` bash
$ tail -n 10 /var/log/messages 
```
输出如下:
``` bash
Apr 22 10:46:38 iZwz9f6pgul78p7die5tlzZ kubelet: Flag --authorization-mode has been deprecated, This parameter should be set via the config file specified by the Kubelet's --config flag. See https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/ for more information.
Apr 22 10:46:38 iZwz9f6pgul78p7die5tlzZ kubelet: Flag --client-ca-file has been deprecated, This parameter should be set via the config file specified by the Kubelet's --config flag. See https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/ for more information.
Apr 22 10:46:38 iZwz9f6pgul78p7die5tlzZ kubelet: Flag --cadvisor-port has been deprecated, The default will change to 0 (disabled) in 1.12, and the cadvisor port will be removed entirely in 1.13
Apr 22 10:46:38 iZwz9f6pgul78p7die5tlzZ kubelet: Flag --cgroup-driver has been deprecated, This parameter should be set via the config file specified by the Kubelet's --config flag. See https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/ for more information.
Apr 22 10:46:38 iZwz9f6pgul78p7die5tlzZ kubelet: Flag --fail-swap-on has been deprecated, This parameter should be set via the config file specified by the Kubelet's --config flag. See https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/ for more information.
Apr 22 10:46:38 iZwz9f6pgul78p7die5tlzZ kubelet: I0422 10:46:38.716591   21994 feature_gate.go:226] feature gates: &{{} map[]}
Apr 22 10:46:38 iZwz9f6pgul78p7die5tlzZ kubelet: F0422 10:46:38.716693   21994 server.go:218] unable to load client CA file /etc/kubernetes/pki/ca.crt: open /etc/kubernetes/pki/ca.crt: no such file or directory
Apr 22 10:46:38 iZwz9f6pgul78p7die5tlzZ systemd: kubelet.service: main process exited, code=exited, status=255/n/a
Apr 22 10:46:38 iZwz9f6pgul78p7die5tlzZ systemd: Unit kubelet.service entered failed state.
Apr 22 10:46:38 iZwz9f6pgul78p7die5tlzZ systemd: kubelet.service failed.
```

输出的第三行有这样一条：`unable to load client CA file /etc/kubernetes/pki/ca.crt: open /etc/kubernetes/pki/ca.crt: no such file or directory`
关于这个客户端证书，笔者自己也没有怎么搞明白，后面需要接着研究。但是貌似这个证书是在`kubeadm init`的过程中创建的。所以笔者采取的方法是首先进行`kubeadm init`操作。
``` bash
$ kubeadm init --apiserver-advertise-address=<你的服务器ip> --kubernetes-version=v1.7.5 --pod-network-cidr=10.244.0.0/12 
```
输出：
```
[kubeadm] WARNING: kubeadm is in beta, please do not use it for production clusters.
[init] Using Kubernetes version: v1.7.5
[init] Using Authorization modes: [Node RBAC]
[preflight] Skipping pre-flight checks
[kubeadm] WARNING: starting in 1.8, tokens expire after 24 hours by default (if you require a non-expiring token use --token-ttl 0)
[certificates] Using the existing CA certificate and key.
[certificates] Using the existing API Server certificate and key.
[certificates] Using the existing API Server kubelet client certificate and key.
[certificates] Using the existing service account token signing key.
[certificates] Using the existing front-proxy CA certificate and key.
[certificates] Using the existing front-proxy client certificate and key.
[certificates] Valid certificates and keys now exist in "/etc/kubernetes/pki"
[kubeconfig] Using existing up-to-date KubeConfig file: "/etc/kubernetes/admin.conf"
[kubeconfig] Using existing up-to-date KubeConfig file: "/etc/kubernetes/kubelet.conf"
[kubeconfig] Using existing up-to-date KubeConfig file: "/etc/kubernetes/controller-manager.conf"
[kubeconfig] Using existing up-to-date KubeConfig file: "/etc/kubernetes/scheduler.conf"
[apiclient] Created API client, waiting for the control plane to become ready
```
初始化的过程卡在了上面输出的地方。其实想想这个过程绝对是会卡住的，毕竟现在`kubelet`都没有跑起来。如果你的kubelet已经跑起来了，但是这个`kubeadm init`还是会卡住，那么就有可能是你的**docker的cgroup driver的类型与kubelet启动参数中指定的cgroup driver类型不一致导致的**,你需要做的就是统一这两者之间的类型。

现在，让我们`Ctrl+C`终止上述的`kubeadm init`过程，然后重新启动我们的kubelet:
``` bash
systemctl start kubelet
```
查看kubelet的状态，kubelet已经跑起来了，乐乐乐:>)

下面我们再次`kubeadm init`,注意这次我们需要加上`--skip-preflight-checks`跳过前期检查。
``` bash
$ kubeadm init --apiserver-advertise-address=<你的服务器ip> --kubernetes-version=v1.7.5 --pod-network-cidr=10.244.0.0/12 --skip-preflight-checks
```
输出：
```
[kubeadm] WARNING: kubeadm is in beta, please do not use it for production clusters.
[init] Using Kubernetes version: v1.7.5
[init] Using Authorization modes: [Node RBAC]
[preflight] Skipping pre-flight checks
[kubeadm] WARNING: starting in 1.8, tokens expire after 24 hours by default (if you require a non-expiring token use --token-ttl 0)
[certificates] Using the existing CA certificate and key.
[certificates] Using the existing API Server certificate and key.
[certificates] Using the existing API Server kubelet client certificate and key.
[certificates] Using the existing service account token signing key.
[certificates] Using the existing front-proxy CA certificate and key.
[certificates] Using the existing front-proxy client certificate and key.
[certificates] Valid certificates and keys now exist in "/etc/kubernetes/pki"
[kubeconfig] Using existing up-to-date KubeConfig file: "/etc/kubernetes/admin.conf"
[kubeconfig] Using existing up-to-date KubeConfig file: "/etc/kubernetes/kubelet.conf"
[kubeconfig] Using existing up-to-date KubeConfig file: "/etc/kubernetes/controller-manager.conf"
[kubeconfig] Using existing up-to-date KubeConfig file: "/etc/kubernetes/scheduler.conf"
[apiclient] Created API client, waiting for the control plane to become ready
[apiclient] All control plane components are healthy after 40.004549 seconds
[token] Using token: xxxxxxxxxxxxxxxxx
[apiconfig] Created RBAC rules
[addons] Applied essential addon: kube-proxy
[addons] Applied essential addon: kube-dns

Your Kubernetes master has initialized successfully!

To start using your cluster, you need to run (as a regular user):

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  http://kubernetes.io/docs/admin/addons/

You can now join any number of machines by running the following on each node
as root:

  kubeadm join --token xxxxx x.x.x.x:6443
```
这下,`kubeadm init`终于成功了、了、了...

正如上面的输出所示，如果你需要让非root用户能够使用你的集群，你需要以该非root用户身份执行下面的命令:
``` bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
现在你可以使用下面的命令来查看你的集群的运行状况了:
```
$ kubectl get pods --all-namepaces
$ kubectl get nodes 
$ kubectl get cs
```
在笔者的机器上，kube-system中的kube-dns对应的pod的状态一直处于异常。这个就需要配置`overlay network`来解决。

# 配置overlay network
笔者开始的时候采用的是flannel来配置的`overlay network`,但是flannel并没有解决kube-dns对应的pod的异常状态，并且网上有声音说flannel与阿里云的主机兼容性不好。所以后面是根据[文章](https://medium.com/@jmarhee/note-on-troubleshooting-kubeadm-managed-kubernetes-cluster-setups-2b2c8248a4e8)来配置的Weave。但是目前团队的生产环境的集群的`overlay network`使用的是flannel，跑的也挺好的。所以我感觉这个东西也是具有很大的差异性的:>),这里就一并写一下使用flannel、weave来配置集群`overlay network`的命令。下面的命令都是可以直接使用的。
## 使用flannel
``` bash
$ kubectl --namespace kube-system apply -f https://raw.githubusercontent.com/coreos/flannel/v0.8.0/Documentation/kube-flannel-rbac.yml
$ rm -rf kube-flannel.yml 
$ wget https://raw.githubusercontent.com/coreos/flannel/v0.8.0/Documentation/kube-flannel.yml
$ sed -i 's/quay.io\/coreos\/flannel:v0.8.0-amd64/registry.cn-hangzhou.aliyuncs.com\/szss_k8s\/flannel:v0.8.0-amd64/g' ./kube-flannel.yml
$ kubectl --namespace kube-system apply -f ./kube-flannel.yml
```

## 使用weave
``` bash
$ export kubever=$(kubectl version | base64 | tr -d '\n')
$ kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$kubever"
```

由于我是改用weave才就kube-dns的pod才正常工作，所以我推荐大家使用weave来配置。

# 最后一个问题
至此，我们的单节点kubernetes集群就搭建完成了。大家在部署应用的时候可能还会遇到另外的一个问题。当你创建了deployment,service之后，查看你的应用的pod，你会发现所有的pod都处于`pending`的状态，`kubectl describe`发现这些pod都没有被调度。这是怎么回事呢？原来，我们现在只有一个节点，而kubernets的调度策略默认是不支持在master节点上放置应用pod的，具体的可以查看这个[issue]((https://github.com/kubernetes/kubernetes/issues/49440),可以使用下面的命令，来去掉这样的调度限制:
```
$ kubectl taint nodes <nodeName> node-role.kubernetes.io/master:NoSchedule-
```
# 总结
大致总结一下安装kubernetes集群的流程：
- 准备系统环境
- 安装Docker
- 安装Kubernetes相关组件
- Kubeadm init Master节点
- 安装Overlay Network
- Join Node节点

这里，由于只有一台主机，所以最后一步没有进行。在整个安装过程中，大致流程是不会改变的。但是可能还是会遇到各种各样的问题。这一次安装的过程让我深刻的体会到认真查看log的重要性，还有依据log去google的重要性。不管什么问题，只要你坚持，最终肯定会找到解决问题的方法。最后，感谢万能的互联网。


