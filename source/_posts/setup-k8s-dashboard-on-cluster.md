---
title: 为集群配置安装kubernetes-dashboard插件
comments: true
date: 2018-04-24 21:02:38
updated: 2018-04-24 21:02:38
tags: [kubernetes,dashboard]
categories: Kubernetes
permalink:
---
kubernetes-Dashboard是一个基于Web UI的kubernetes用户接口。你可以使用它在kubernetes集群中部署、调试容器化的应用，管理集群本身，创建、预览、更新集群中所拥有的资源对象，扩展和滚动更新你的应用。下面就来记录一下我是如何配置安装kuernetes-dashboard的。

kubernetes-dashboard 1.6.x以前和1.7.x的差距比较大，主要增加了一些https的认证。由于我们的集群的版本为1.7.5，下面打算安装kubernetes-dashboard V1.7.1,首先我们需要做的就是下载需要的镜像。
# 镜像准备
``` bash
#!/bin/bash
set -o errexit
set -o nounset
set -o pipefail

GCR_URL=gcr.io/google_containers
DOCKERHUB_URL=alleyj

images=(kubernetes-dashboard-init-amd64:V1.0.1
    kubernetes-dashboard-amd64:v1.7.1)

for imageName in ${images[@]} ; do
  docker pull $DOCKERHUB_URL/$imageName
  docker tag $DOCKERHUB_URL/$imageName $GCR_URL/$imageName
  docker rmi $DOCKERHUB_URL/$imageName
done
```

# 下载对应的yml文件
接下来我们需要一个kubernetes-dashboard.yaml的配置文件，可以直接在[dashboard的仓库中](https://github.com/kubernetes/dashboard/tree/master/src/deploy/recommended)的dashboard/src/deploy/recommended文件夹下下载。**注意:由于我们这里安装的是kubernetes-dashboard V1.7.1,所以我们需要下载对应版本的kubernetes-dashboard.yaml文件。具体做法是，调整repository的tag至v1.7.1**。查看文件的内容:
``` 
# Copyright 2015 Google Inc. All Rights Reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# Configuration to deploy release version of the Dashboard UI compatible with
# Kubernetes 1.7.
#
# Example usage: kubectl create -f <this_file>

# ------------------- Dashboard Secret ------------------- #

apiVersion: v1
kind: Secret
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard-certs
  namespace: kube-system
type: Opaque

---
# ------------------- Dashboard Service Account ------------------- #

apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kube-system

---
# ------------------- Dashboard Role & Role Binding ------------------- #

kind: Role
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: kubernetes-dashboard-minimal
  namespace: kube-system
rules:
  # Allow Dashboard to create and watch for changes of 'kubernetes-dashboard-key-holder' secret.
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["create", "watch"]
- apiGroups: [""]
  resources: ["secrets"]
  # Allow Dashboard to get, update and delete 'kubernetes-dashboard-key-holder' secret.
  resourceNames: ["kubernetes-dashboard-key-holder", "kubernetes-dashboard-certs"]
  verbs: ["get", "update", "delete"]
  # Allow Dashboard to get metrics from heapster.
- apiGroups: [""]
  resources: ["services"]
  resourceNames: ["heapster"]
  verbs: ["proxy"]

---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: RoleBinding
metadata:
  name: kubernetes-dashboard-minimal
  namespace: kube-system
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: kubernetes-dashboard-minimal
subjects:
- kind: ServiceAccount
  name: kubernetes-dashboard
  namespace: kube-system

---
# ------------------- Dashboard Deployment ------------------- #

kind: Deployment
apiVersion: extensions/v1beta1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kube-system
spec:
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      k8s-app: kubernetes-dashboard
  template:
    metadata:
      labels:
        k8s-app: kubernetes-dashboard
    spec:
      initContainers:
      - name: kubernetes-dashboard-init
        image: gcr.io/google_containers/kubernetes-dashboard-init-amd64:v1.0.0
        volumeMounts:
        - name: kubernetes-dashboard-certs
          mountPath: /certs
      containers:
      - name: kubernetes-dashboard
        image: gcr.io/google_containers/kubernetes-dashboard-amd64:v1.7.1
        ports:
        - containerPort: 9090
          protocol: TCP
        args:
          - --tls-key-file=/certs/dashboard.key
          - --tls-cert-file=/certs/dashboard.crt
          # Uncomment the following line to manually specify Kubernetes API server Host
          # If not specified, Dashboard will attempt to auto discover the API server and connect
          # to it. Uncomment only if the default does not work.
          # - --apiserver-host=http://my-address:port
        volumeMounts:
        - name: kubernetes-dashboard-certs
          mountPath: /certs
          readOnly: true
          # Create on-disk volume to store exec logs
        - mountPath: /tmp
          name: tmp-volume
        livenessProbe:
          httpGet:
            scheme: HTTPS
            path: /
            port: 8443
          initialDelaySeconds: 30
          timeoutSeconds: 30
      volumes:
      - name: kubernetes-dashboard-certs
        secret:
          secretName: kubernetes-dashboard-certs
      - name: tmp-volume
        emptyDir: {}
      serviceAccountName: kubernetes-dashboard
      # Comment the following tolerations if Dashboard must not be deployed on master
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule

---
# ------------------- Dashboard Service ------------------- #

kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kube-system
spec:
  ports:
    - port: 443
      targetPort: 8443
  selector:
    k8s-app: kubernetes-dashboard
```
查看上述的文件，可以看到这里需要的两个镜像都已经存在于本地，但是略有不同的是这里需要的`kubernetes-dashboard-init-amd64:v1.0.0`，而我们本地有的镜像tag为`v1.0.1`,所以我们这里更改一下这个yml文件，将`kubernetes-dashboard-init-amd64的tag`从`v1.0.0`改为`v1.0.1`.

# 创建相应的资源
## 初次尝试
``` bash
$ kc create -f kubernetes-dashboard.yml
```
查看kube-system中，kubernetes-dashboard所对应的pod的状态一直是这样的:
``` 
NAME                                              READY     STATUS     RESTARTS   AGE
kubernetes-dashboard-3554419144-j2qqp             0/1       Init:0/1   0          3m
```
我们discribe一下这个pod
``` bash
$ kubectl describe pod kubernetes-dashboard-3554419144-j2qqp -n kube-system
```
输出的Event如下:
``` bash
Events:
  FirstSeen	LastSeen	Count	From					SubObjectPath					Type		Reason			Message
  ---------	--------	-----	----					-------------					--------	------			-------
  29s		29s		1	default-scheduler									Normal		Scheduled		Successfully assigned kubernetes-dashboard-3554419144-j2qqp to izwz9f6pgul78p7die5tlzz
  29s		29s		1	kubelet, izwz9f6pgul78p7die5tlzz							Normal		SuccessfulMountVolume	MountVolume.SetUp succeeded for volume "tmp-volume" 
  29s		29s		1	kubelet, izwz9f6pgul78p7die5tlzz							Normal		SuccessfulMountVolume	MountVolume.SetUp succeeded for volume "kubernetes-dashboard-certs" 
  29s		29s		1	kubelet, izwz9f6pgul78p7die5tlzz							Normal		SuccessfulMountVolume	MountVolume.SetUp succeeded for volume "kubernetes-dashboard-token-5mhr7" 
  28s		28s		1	kubelet, izwz9f6pgul78p7die5tlzz	spec.initContainers{kubernetes-dashboard-init}	Normal		Pulled			Container image "gcr.io/google_containers/kubernetes-dashboard-init-amd64:v1.0.1" already present on machine
  28s		28s		1	kubelet, izwz9f6pgul78p7die5tlzz	spec.initContainers{kubernetes-dashboard-init}	Normal		Created			Created container
  28s		28s		1	kubelet, izwz9f6pgul78p7die5tlzz	spec.initContainers{kubernetes-dashboard-init}	Normal		Started			Started container
  27s		8s		3	kubelet, izwz9f6pgul78p7die5tlzz	spec.containers{kubernetes-dashboard}		Normal		Pulled			Container image "gcr.io/google_containers/kubernetes-dashboard-amd64:v1.7.1" already present on machine
  27s		8s		3	kubelet, izwz9f6pgul78p7die5tlzz	spec.containers{kubernetes-dashboard}		Normal		Created			Created container
  26s		7s		3	kubelet, izwz9f6pgul78p7die5tlzz	spec.containers{kubernetes-dashboard}		Normal		Started			Started container
  25s		4s		5	kubelet, izwz9f6pgul78p7die5tlzz	spec.containers{kubernetes-dashboard}		Warning		BackOff			Back-off restarting failed container
```
可以看到，initContainer kubernetes-dashboard-init容器全运行正常，kubernetes-dashboard容器运行失败。

接着我们来查看一下logs:
``` bash
$ kubectl logs kubernetes-dashboard-3554419144-j2qqp -n kube-system
```
输出如下:
``` 
2018/04/24 14:41:19 Using in-cluster config to connect to apiserver
2018/04/24 14:41:19 Using service account token for csrf signing
2018/04/24 14:41:19 No request provided. Skipping authorization
2018/04/24 14:41:19 Starting overwatch
2018/04/24 14:41:19 Successful initial request to the apiserver, version: v1.7.5
2018/04/24 14:41:19 New synchronizer has been registered: kubernetes-dashboard-key-holder-kube-system. Starting
2018/04/24 14:41:19 Starting secret synchronizer for kubernetes-dashboard-key-holder in namespace kube-system
2018/04/24 14:41:19 Initializing secret synchronizer synchronously using secret kubernetes-dashboard-key-holder from namespace kube-system
2018/04/24 14:41:19 Initializing JWE encryption key from synchronized object
2018/04/24 14:41:19 Creating in-cluster Heapster client
2018/04/24 14:41:19 Serving securely on HTTPS port: 8443
2018/04/24 14:41:19 open /certs/dashboard.crt: no such file or directory
```
可以看到，这里说没有`/certs/dashboard.crt`文件，这个证书文件本应该是在运行init容器时创建的，但是这里不知道为什么我们这里没有创建成功。

下面，我们删除刚才由`kubernetes-dashboard.yml`文件所创建的资源，然后采取openssl来创建自签名证书。
``` bash
$ kubectl delete -f kubenetes-dashboard.yml
```
## 再次尝试
``` bash
$ openssl req -newkey rsa:4096 -nodes -sha256 -keyout dashboard.key -x509 -days 365 -out dashboard.crt
```
依据提示，填写签发者信息:
```
Generating a 4096 bit RSA private key
......................++
....................................................................................................++
writing new private key to 'dashboard.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:CN
State or Province Name (full name) []:Hubei  
Locality Name (eg, city) [Default City]:Wuhan
Organization Name (eg, company) [Default Company Ltd]:Muxistudio
Organizational Unit Name (eg, section) []:be
Common Name (eg, your name or your server's hostname) []:iZwz9f6pgul78p7die5tlzZ
Email Address []:3480437308@qq.com
```
执行完之后，你的命令执行目录就会多出来两个文件
```
$ ll
```
``` 
-rw-r--r--. 1 root root 2086 Nov 14 09:59 dashboard.crt
-rw-r--r--. 1 root root 3272 Nov 14 09:59 dashboard.key
```
将这两个文件移动到`/certs/`目录之下，然后根据这两个文件使用下面的命令来创建secret,来替换掉kubernetes-dashboard.yml中定义的secret：
``` bash
$ kubectl create secret generic kubernetes-dashboard-certs --from-file=/certs -n kube-system
```
下面，我们再次依据`kubernetes-dashboard.yml`来创建dashboard所需要的资源对象:
``` bash
$ kubectl create -f kubernetes-dashboard.yml
```
输出如下:
``` 
serviceaccount "kubernetes-dashboard" created
role "kubernetes-dashboard-minimal" created
rolebinding "kubernetes-dashboard-minimal" created
deployment "kubernetes-dashboard" created
service "kubernetes-dashboard" created
Error from server (AlreadyExists): error when creating "dashboard.yml": secrets "kubernetes-dashboard-certs" already exists
```
从输出也可以看出来，其他资源已经创建成功，而`kubernetes-dashboard-certs`则被我们成功替换掉了。

下面，我们查看一下dashboard对应的pod的事件:
``` bash
$ kubectl describe pod kubernetes-dashboard-3554419144-jjj5q -n kube-system
```
事件如下：
``` 
Events:
  FirstSeen	LastSeen	Count	From					SubObjectPath					Type		Reason			Message
  ---------	--------	-----	----					-------------					--------	------			-------
  3m		3m		1	default-scheduler									Normal		Scheduled		Successfully assigned kubernetes-dashboard-3554419144-jjj5q to izwz9f6pgul78p7die5tlzz
  3m		3m		1	kubelet, izwz9f6pgul78p7die5tlzz							Normal		SuccessfulMountVolume	MountVolume.SetUp succeeded for volume "tmp-volume" 
  3m		3m		1	kubelet, izwz9f6pgul78p7die5tlzz							Normal		SuccessfulMountVolume	MountVolume.SetUp succeeded for volume "kubernetes-dashboard-token-jzx4v" 
  3m		3m		1	kubelet, izwz9f6pgul78p7die5tlzz							Normal		SuccessfulMountVolume	MountVolume.SetUp succeeded for volume "kubernetes-dashboard-certs" 
  3m		3m		1	kubelet, izwz9f6pgul78p7die5tlzz	spec.initContainers{kubernetes-dashboard-init}	Normal		Pulled			Container image "gcr.io/google_containers/kubernetes-dashboard-init-amd64:v1.0.1" already present on machine
  3m		3m		1	kubelet, izwz9f6pgul78p7die5tlzz	spec.initContainers{kubernetes-dashboard-init}	Normal		Created			Created container
  3m		3m		1	kubelet, izwz9f6pgul78p7die5tlzz	spec.initContainers{kubernetes-dashboard-init}	Normal		Started			Started container
  3m		3m		1	kubelet, izwz9f6pgul78p7die5tlzz	spec.containers{kubernetes-dashboard}		Normal		Pulled			Container image "gcr.io/google_containers/kubernetes-dashboard-amd64:v1.7.1" already present on machine
  3m		3m		1	kubelet, izwz9f6pgul78p7die5tlzz	spec.containers{kubernetes-dashboard}		Normal		Created			Created container
  3m		3m		1	kubelet, izwz9f6pgul78p7die5tlzz	spec.containers{kubernetes-dashboard}		Normal		Started			Started container
```
如上，一切正常。我们的kubernetes-dashboard插件就此安装完成。

# 暴露服务
下面，我们要访问服务器上的`kubernetes-dashboard`服务就还需要做下面的一步操作：将kubernetes-dashboard的service变为外部可路由。将service的类型由默认的ClusterIP改为NodePort.
执行下面的命令，更改service的配置文件:
``` bash
$ kubectl edit service kubernetes-dashboard -n kube-system
```
更改完成之后，我们查看服务的暴露的主机端口:
``` bash
$ kubectl get service kubernetes-dashboard -n kube-system
```
输出:
``` 
NAME                   CLUSTER-IP       EXTERNAL-IP   PORT(S)         AGE
kubernetes-dashboard   10.109.219.222   <nodes>       443:30087/TCP   52m
```
如上，服务暴露的主机端口为30087,我们访问`https://<主机公网IP>:30087/`即可访问到dashboard服务。注意，**这里使用的是HTTPS服务，并且使用的是自签名证书，所以在我们初次进入时，浏览器会因为无法识别证书的签署机构而阻止继续访问，这时我们只需要点击高级选项，标识网站为安全即可。**

