---
title: Kubernetes-dashboard的身份认证
comments: true
date: 2018-04-25 21:24:02
updated: 2018-04-28 23:24:02
tags: [kubernetes,dashboard]
categories: Kubernetes
permalink:
---
上一篇文章中，我们成功配置安装了kubernetes-dashboard插件，但是这里似乎来了另外一个问题:我们怎样进入到dashboard?

![login](/images/dashboard.login.png)
如上图，`kubernetes-dashboard`提供了两种验证方式:`kubeconfig`、`token`。这两种验证方式都是怎么回事呢?诶，好像有一个`skip`,我们点击看看。直接点击`skip`,我们进入到了dashboard的界面，但是似乎我们什么都做不了，页面给出了提醒，我们没有权限查看和操作集群里面的资源。该怎么办呢？下面就让我们一起来看看kubernetes里面的身份认证和权限管理吧!

要了解k8s中的身份认证和权限管理我们就必须先来了解k8s中的RBAC(Role-based access control)授权模式。
# RBAC in K8s
`RBAC Authorization`的基本概念是`Role`和`RoleBinding`。`Role`是一些`permission`的集合；而`RoleBinding`则是将`Role`授权给某些`User`、某些`Group`或某些`ServiceAccount`。K8s官方博客[RBAC Support in Kubernetes](https://kubernetes.io/blog/2017/04/rbac-support-in-kubernetes)一文的中的配图对此做了很生动的诠释：

![picture](/images/rbac2.png)
从上图中我们可以看到：
Role:`pod-reader` 拥有Pod的`get`和`list` permissions；
RoleBinding:`pod-reader`将`Role`:`pod-reader`授权给右边的`User`、`Group`和`ServiceAccount`。

与`Role`和`RoleBinding`对应的是，K8s还有`ClusterRole`和`ClusterRoleBinding`的概念，它们不同之处在于：`ClusterRole`和`ClusterRoleBinding`是针对整个`Cluster`范围内有效的，无论用户或资源所在的`namespace`是什么；而`Role`和`RoleBinding`的作用范围是局限在某个k8s namespace中的。

kubernetes在安装之初就已经生成了许多`role`、`rolebinding`、`clusterrole`和`clusterrolebinding`,它们也是属于kubernetes资源的一部分，所以可以通过`get`、`describe`等命令查看，如下：
``` bash
[root@iZwz9f6pgul78p7die5tlzZ dashboard]# kc get role -n kube-system
NAME                                             AGE
extension-apiserver-authentication-reader        5d
kubernetes-dashboard-minimal                     2d
system::leader-locking-kube-controller-manager   5d
system::leader-locking-kube-scheduler            5d
system:controller:bootstrap-signer               5d
system:controller:cloud-provider                 5d
system:controller:token-cleaner                  5d
weave-net                                        5d
[root@iZwz9f6pgul78p7die5tlzZ dashboard]# kc describe role extension-apiserver-authentication-reader -n kube-system
Name:		extension-apiserver-authentication-reader
Labels:		kubernetes.io/bootstrapping=rbac-defaults
Annotations:	rbac.authorization.kubernetes.io/autoupdate=true
PolicyRule:
  Resources	Non-Resource URLs	Resource Names				Verbs
  ---------	-----------------	--------------				-----
  configmaps	[]			[extension-apiserver-authentication]	[get]
```

下面截取了`kubernetes-dashboard.yml`文件的一部分：
``` yml 
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
```
上面这个yml文件做了这么一件事情：在`kube-system`命名空间中创建了一个名为`kubernetes-dashboard`的`ServiceAccount`。同时，在`kube-system`命名空间中创建了一个名为`kubernetes-dashboard-minimal`的`Role`，并且定义了这个Role的权限.然后同样是在`kube-system`命名空间中创建了一个`RoleBinding`,将上面的`Role`与`ServiceAccount`绑定在一起了。这样`kubernetes-dashboard`就有了`kubernetes-dashboard-minimal`所定义的权限了。**有一点需要注意：这里的`kubernetes-dashboard`这个ServiceAccount是当用户直接点击`skip`进入到`dashboard`时所使用的账户。**

# 测试环境中的认证
如果是在测试环境中，我们图个简单，不考虑安全性的情况之下。可以考虑让外部用户直接点击`skip`进入到dashboard，并且拥有所有的权限。这一点可以通过将`cluster-admin`这个拥有全集群最高权限的`ClusterRole`绑定到默认使用的`ServiceAccount－－kubernetes-dashboard`，具体的做法是:
创建文件:`dashboard-admin.yml`,填写下列内容:
``` yml
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: kubernetes-dashboard
  labels:
    k8s-app: kubernetes-dashboard
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: kubernetes-dashboard
  namespace: kube-system
```
执行下面的命令来创建一个ClusterRoleBinding：
``` bash
$ kubectl create -f dashboard-admin.yml
```
这样，我们就可以直接点击`skip`进入dashboard,并且拥有全部的权限。
# 基于token的认证
下面我们来说说token这种方式。点击选择:Token单选框，提示你输入token。token从哪里获取，我们从来没有生成过token？其实当前K8s中已经有了很多token：
``` 
[root@iZwz9f6pgul78p7die5tlzZ dashboard]# kc get secret -n kube-system
NAME                                     TYPE                                  DATA      AGE
attachdetach-controller-token-vkm75      kubernetes.io/service-account-token   3         5d
bootstrap-signer-token-k59x3             kubernetes.io/service-account-token   3         5d
bootstrap-token-ff4535                   bootstrap.kubernetes.io/token         5         5d
certificate-controller-token-z3qpm       kubernetes.io/service-account-token   3         5d
daemon-set-controller-token-3lbtj        kubernetes.io/service-account-token   3         5d
default-token-8372f                      kubernetes.io/service-account-token   3         5d
deployment-controller-token-pb2wl        kubernetes.io/service-account-token   3         5d
disruption-controller-token-jwjlt        kubernetes.io/service-account-token   3         5d
endpoint-controller-token-t3h87          kubernetes.io/service-account-token   3         5d
generic-garbage-collector-token-sm8lt    kubernetes.io/service-account-token   3         5d
heapster-token-zqld8                     kubernetes.io/service-account-token   3         6h
horizontal-pod-autoscaler-token-v6wjc    kubernetes.io/service-account-token   3         5d
job-controller-token-hfl7h               kubernetes.io/service-account-token   3         5d
kube-dns-token-rfkvx                     kubernetes.io/service-account-token   3         5d
kube-proxy-token-r017j                   kubernetes.io/service-account-token   3         5d
kubernetes-dashboard-certs               Opaque                                2         2d
kubernetes-dashboard-key-holder          Opaque                                2         4d
kubernetes-dashboard-token-jzx4v         kubernetes.io/service-account-token   3         2d
namespace-controller-token-3j3sn         kubernetes.io/service-account-token   3         5d
node-controller-token-cnjsn              kubernetes.io/service-account-token   3         5d
persistent-volume-binder-token-p1cwr     kubernetes.io/service-account-token   3         5d
pod-garbage-collector-token-rbw2m        kubernetes.io/service-account-token   3         5d
replicaset-controller-token-pt682        kubernetes.io/service-account-token   3         5d
replication-controller-token-s2kb7       kubernetes.io/service-account-token   3         5d
resourcequota-controller-token-xlrrh     kubernetes.io/service-account-token   3         5d
service-account-controller-token-zlcph   kubernetes.io/service-account-token   3         5d
service-controller-token-0cqs6           kubernetes.io/service-account-token   3         5d
statefulset-controller-token-0p29q       kubernetes.io/service-account-token   3         5d
token-cleaner-token-cq9nk                kubernetes.io/service-account-token   3         5d
ttl-controller-token-gndzv               kubernetes.io/service-account-token   3         5d
weave-net-token-w6grc                    kubernetes.io/service-account-token   3         5d
[root@iZwz9f6pgul78p7die5tlzZ dashboard]# kc describe secret attachdetach-controller-token-vkm75 -n kube-system
Name:		attachdetach-controller-token-vkm75
Namespace:	kube-system
Labels:		<none>
Annotations:	kubernetes.io/service-account.name=attachdetach-controller
		kubernetes.io/service-account.uid=691cdfc2-4612-11e8-8dc4-00163e0a39da

Type:	kubernetes.io/service-account-token

Data
====
ca.crt:		1025 bytes
namespace:	11 bytes
token:		eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhdHRhY2hkZXRhY2gtY29udHJvbGxlci10b2tlbi12a203NSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJhdHRhY2hkZXRhY2gtY29udHJvbGxlciIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjY5MWNkZmMyLTQ2MTItMTFlOC04ZGM0LTAwMTYzZTBhMzlkYSIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTphdHRhY2hkZXRhY2gtY29udHJvbGxlciJ9.ZBoh8-i6882nHH_oaEhtUw7L-M-2kGgrV3dRqAFw1bQUbenOYCK_xqQFeW0gTpEmYa_-k2cukfHKgqtyCF2U_hr1mXikc4aAZzJubnszQhNnASzHFihi5AkZfxBBiqYjd8WUaUSms1VbvImvmSg_Ndrw3-SSB0B0b-sWEH4dAwcXx1_hN2V3GBXZjjdFHT51U6ogvwzs-YJ_Uk5GqWIPxPNHhGMFQtQL3vVIHpxumtG6xdoVRuDitMl0gH71gugSgjabNLPMjIHmApbI3BIeH9bX9jO271OdTzNGvKaBOx2xLRJBTvrk4bSyZsSdqrLkWlUVgCTw7VNLKpmgfNTMDQ
```
如上，这里有很多的`secret`存在于系统之中，每个`secret`都对应了一个`token`,但是这些`token`所对应的权限都不相同,所以不一定会符合我们的要求。

这里我们需要创建一个名为admin的ServiceAccount并绑定名为cluster-admin的ClusterRole角色(该角色拥有集群最高权限)，使用下面的yaml文件创建admin用户并赋予他管理员权限，然后可以通过token登陆dashbaord。这种认证方式本质上是通过ServiceAccount的身份认证加上Bearer token请求API server的方式实现。
admin-token.yml
```
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: admin
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
subjects:
- kind: ServiceAccount
  name: admin
  namespace: kube-system
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin
  namespace: kube-system
  labels:
    kubernetes.io/cluster-service: "true"
    addonmanager.kubernetes.io/mode: Reconcile
```
运行下面的命令:
``` bash
$ kubectl create -f admin-token.yml
```
然后我们可以通过如下的命令来获取admin ServiceAccount的token：
```
[root@iZwz9f6pgul78p7die5tlzZ ~]# kc get secret -n kube-system | grep admin
admin-token-whj4t                        kubernetes.io/service-account-token   3         1d
[root@iZwz9f6pgul78p7die5tlzZ ~]# kc describe secret/admin-token-whj4t -n kube-system
Name:		admin-token-whj4t
Namespace:	kube-system
Labels:		<none>
Annotations:	kubernetes.io/service-account.name=admin
		kubernetes.io/service-account.uid=f2e69a6e-4a03-11e8-8dc4-00163e0a39da

Type:	kubernetes.io/service-account-token

Data
====
ca.crt:		1025 bytes
namespace:	11 bytes
token:		eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi10b2tlbi13aGo0dCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJhZG1pbiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6ImYyZTY5YTZlLTRhMDMtMTFlOC04ZGM0LTAwMTYzZTBhMzlkYSIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTphZG1pbiJ9.Nq6N7E7xY09sITcqa0WP3OeNo6Sp_se69q9aH3oM-gWDx7Sp_sbRn1VIBE-op9yG4XRbkoam-KVYhyX7QVATesYCF1fZ0p0ENcctAhmO9aEoQ1GS5I6W3DWrUXgWgaZJSrNHt-czHC_WUB3ilggaDcOEAqPEb3gYqrezyEarclQPNQZfHo3UNWjhjCqmm2vOEweCbyn9o0t8QHTTT_Pp26Bq1ho2B4HqGEeM8RHa175mG18eJQ5aRYuMM70Yp0uNyyMQmXnPTNzX0uHvU9uq-dSxxDRlq5bRg_l5bravtCsr51I-VMU9FyWd3OJWK0z1hrO76X1JrWfsbmzY3rVCSg
```
如上，我们得到了该用户的token,**注意：这里的token是进行base64编码后的结果，而我们需要的是解码之后的结果**,直接获取解码之后的token可以通过下面的命令实现:
``` bash
$ kubectl -n kube-system get secret admin-token-whj4t -o jsonpath={.data.token}|base64 -d
```
这样，我们将解码之后的token复制出来，填入dashboard认证表单中就可以进入dashboard并且获取全集群的最高权限。

# 基于kubeconfig的认证
如何生成kubeconfig文件请参考创建[用户认证授权的kubeconfig文件](https://jimmysong.io/kubernetes-handbook/guide/kubectl-user-authentication-authorization.html)。

注意参考文章中生成的kubeconfig文件中没有token字段，如果我们要使用`kubeconfig`登录dashboard则需要手动添加该字段。

对于访问`dashboard`时候使用的`kubeconfig`文件如`brand.kubeconfig`必须追到`token`字段，否则认证不会通过。而使用`kubectl`命令时的用的`kubeconfig`文件则不需要包含token字段。

kubeconfig的认证可以让拥有该kubeconfig的用户只拥有一个或几个命名空间的操作权限，这相比与上面的token的方式更加的精确和安全。kubeconfig也可以针对`kubectl`:
``` bash
$ cp -f brand.kubeconfig　~/.kube/config
```
这样，该用户就只具有brand.kubeconfig文件所确定的命名空间的操作权限了。