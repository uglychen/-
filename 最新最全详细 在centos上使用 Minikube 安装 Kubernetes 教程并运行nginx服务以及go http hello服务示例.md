**最新最全详细 在centos上使用 Minikube 安装 Kubernetes 教程，在安装完k8s单机集群后并运行一个nginx服务 和一个go 的http hello server 服务**

**本文已更新到 Minikube v1.11.0/Kubernetes v1.18+**

**什么是Minikube**？  

- [ ] Minikube 是一种可以让您在本地轻松运行 Kubernetes 的工具。Minikube 在笔记本电脑上的虚拟机（VM）中运行单节点 Kubernetes 集群，供那些希望尝试 Kubernetes 或进行日常开发的用户使用

  参考官方文档：
https://kubernetes.io/zh/docs/tasks/tools/install-minikube/

pis：
- 官方的在国内不怎么好使，使用都是google源，大部分被墙了，下面安装都是使用国内的源进行安装。
- 安装k8s对机器硬件系统有一定要求，2G  2核  20G好像是最低要求，小于这个配置会提示你安装不了k8s集群。具体看官方配置要求。  
参考：https://www.jianshu.com/p/ae6260bd5596

**注意: 本文安装教程是基于centos系统的。安装之前可以把yum源更换为国内阿里源，然后可以yum date一下**  
参考：https://blog.csdn.net/sinat_33384251/article/details/91404617

# 一、安装安装 kubectl
kubectl是Kubernetes集群的命令行工具,用来操作集群的。  
在 Kubernetes 上使用 Kubernetes 命令行工具 kubectl 部署和管理应用程序。使用 kubectl，您可以检查集群资源；创建、删除和更新组件；查看您的新集群；并启动实例应用程序。  
官方参考：https://kubernetes.io/zh/docs/tasks/tools/install-kubectl/

国内安装快速安装kubectl  
参考：https://www.jianshu.com/p/b58c85436f0a  

配置k8s的kubelet yum源
```
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```
yum安装kubectl：

```
# 安装
yum install -y kubectl kubelet kubeadm
# 开机启动
systemctl enable kubelet
# 启动
systemctl start kubelet
```
查看版本 kubectl version
我这安装的是1.18.3版本
```
[root@localhost k8s]# kubectl version
Client Version: version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.3", GitCommit:"2e7996e3e2712684bc73f0dec0200d64eec7fe40", GitTreeState:"clean", BuildDate:"2020-05-20T12:52:00Z", GoVersion:"go1.13.9", Compiler:"gc", Platform:"linux/amd64"}
The connection to the server localhost:8080 was refused - did you specify the right host or port?
```

# 二、先把docker安装上吧
在安装Minikube 之前把docker安装好吧，如果docker版本比较则最好更新好最新的版本免得后面要安装出现问题。  
参考：https://www.runoob.com/docker/centos-docker-install.html


```
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
```
更改docker镜像源
参考：https://www.jianshu.com/p/4002f93c87c4

```
vim /etc/docker/daemon.json #没有则创建daemon.json文件
```

```
{
    "registry-mirrors":["你个人的加速器地址"]
}
#加速地址自行搜索阿里云docker配置
```
我的:
```
root@localhost k8s]# vim /etc/docker/daemon.json
[root@localhost k8s]# cat /etc/docker/daemon.json
{
    "registry-mirrors":["https://qdsf52uj.mirror.aliyuncs.com"]
}

```
重启docker
```
systemctl daemon-reload
systemctl restart docker
```

# 三、安装 Minikube
minikube
阿里云发布的minikube   
github地址：https://github.com/AliyunContainerService/minikube

```
curl -Lo minikube https://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/releases/v1.11.0/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
```

# 启动单机集群
我使用--driver=none模式
```
minikube start --driver=none
```

第一次需要一点时间下载，耐心等待吧.....
```
[root@localhost k8s]# minikube start --driver=none
😄  minikube v1.11.0 on Centos 7.8.2003
✨  Using the none driver based on user configuration
👍  Starting control plane node minikube in cluster minikube
🤹  Running on localhost (CPUs=2, Memory=1819MB, Disk=17394MB) ...
ℹ️  OS release is CentOS Linux 7 (Core)
🐳  Preparing Kubernetes v1.18.3 on Docker 19.03.11 ...
    > kubeadm.sha256: 65 B / 65 B [--------------------------] 100.00% ? p/s 0s
    > kubelet.sha256: 65 B / 65 B [--------------------------] 100.00% ? p/s 0s
    > kubectl.sha256: 65 B / 65 B [--------------------------] 100.00% ? p/s 0s
    > kubeadm: 37.97 MiB / 37.97 MiB [------------] 100.00% 1011.17 KiB p/s 38s
    > kubectl: 41.99 MiB / 41.99 MiB [-------------] 100.00% 894.57 KiB p/s 48s
    > kubelet: 10.71 MiB / 108.04 MiB [>______] 9.91% 26.82 KiB p/s ETA 1h1m56s
```
出现错误：

```
/proc/sys/net/bridge/bridge-nf-call-iptables contents are not set to 1
```

解决方法：

```
[root@localhost k8s]# swapoff -a
[root@localhost k8s]#  echo 1 > /proc/sys/net/bridge/bridge-nf-call-iptables
```
看到下面信息就表示成功启动k8s单机集群了

```
[root@localhost k8s]# minikube start --driver=none
😄  minikube v1.11.0 on Centos 7.8.2003
✨  Using the none driver based on user configuration
👍  Starting control plane node minikube in cluster minikube
🤹  Running on localhost (CPUs=2, Memory=1819MB, Disk=17394MB) ...
ℹ️  OS release is CentOS Linux 7 (Core)
🐳  Preparing Kubernetes v1.18.3 on Docker 19.03.11 ...

🤹  Configuring local host environment ...

❗  The 'none' driver is designed for experts who need to integrate with an existing VM
💡  Most users should use the newer 'docker' driver instead, which does not require root!
📘  For more information, see: https://minikube.sigs.k8s.io/docs/reference/drivers/none/

❗  kubectl and minikube configuration will be stored in /root
❗  To use kubectl or minikube commands as your own user, you may need to relocate them. For example, to overwrite your own settings, run:

    ▪ sudo mv /root/.kube /root/.minikube $HOME
    ▪ sudo chown -R $USER $HOME/.kube $HOME/.minikube

💡  This can also be done automatically by setting the env var CHANGE_MINIKUBE_NONE_USER=true
🔎  Verifying Kubernetes components...
🌟  Enabled addons: default-storageclass, storage-provisioner
🏄  Done! kubectl is now configured to use "minikube"

```

# 打开Kubernetes控制台

```
minikube dashboard
```
如果出现错误多执行几次minikube dashboard： 因为可能是容器没完全拉下来

kubectl get pod --all-namespaces 查看是否全部拉下没有  READY 状态 1/1表完成 我下面0/1表示没拉下。执行minikube dashboard直到都完全拉下来
```
kubectl get pod --all-namespaces
NAMESPACE              NAME                                            READY   STATUS             RESTARTS   AGE
kube-system            coredns-546565776c-lb46j                        0/1     Running            0          17m
kube-system            coredns-546565776c-sxfkw                        0/1     Running            0          17m
kube-system            etcd-localhost.localdomain                      1/1     Running            0          17m
kube-system            kube-apiserver-localhost.localdomain            1/1     Running            0          17m
kube-system            kube-controller-manager-localhost.localdomain   1/1     Running            0          17m
kube-system            kube-proxy-467j8                                1/1     Running            0          17m
kube-system            kube-scheduler-localhost.localdomain            1/1     Running            0          17m
kube-system            storage-provisioner                             1/1     Running            0          17m
kubernetes-dashboard   dashboard-metrics-scraper-84bfdf55ff-hdjhn      1/1     Running            0          13m
kubernetes-dashboard   kubernetes-dashboard-696dbcc666-tzzls           0/1     CrashLoopBackOff   7          13m
```
再次kubectl get pod --all-namespaces

```
[root@localhost ~]# kubectl get pod --all-namespaces
NAMESPACE              NAME                                            READY   STATUS             RESTARTS   AGE
kube-system            coredns-546565776c-lb46j                        1/1     Running            0          25m
kube-system            coredns-546565776c-sxfkw                        1/1     Running            0          25m
kube-system            etcd-localhost.localdomain                      1/1     Running            0          25m
kube-system            kube-apiserver-localhost.localdomain            1/1     Running            0          25m
kube-system            kube-controller-manager-localhost.localdomain   1/1     Running            0          25m
kube-system            kube-proxy-467j8                                1/1     Running            0          25m
kube-system            kube-scheduler-localhost.localdomain            1/1     Running            0          25m
kube-system            storage-provisioner                             1/1     Running            0          25m
kubernetes-dashboard   dashboard-metrics-scraper-84bfdf55ff-hdjhn      1/1     Running            0          21m
kubernetes-dashboard   kubernetes-dashboard-696dbcc666-tzzls           0/1     CrashLoopBackOff   8          21m
```
出现下面的信息表示完全启动dashboard

```
 minikube dashboard
🤔  Verifying dashboard health ...
🚀  Launching proxy ...
🤔  Verifying proxy health ...



http://127.0.0.1:39798/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/
```
**通过apiserver访问**

对外暴露地址：  

nohup kubectl proxy  --port=8088 --address='192.168.1.128'  --accept-hosts='^.*'   >/dev/null 2>&1 &  
然后就可以在浏览器上访问了:

http://192.168.1.128:8088/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/

- 完成安装之后，我们就可以发布，更新，回滚自己的程序。这个时候我们就需要了解Pod、ReplicaSet、Deployment、Service相关的概念。

# 使用k8s运行服务
## 1、运行nginx服务

创建第一个Deployment

- 了解到只需要创建好Deployment就会自动完成ReplicaSet和pod。下面就创建一个Nginx的Deployment。

nginx-deployment.yaml文件内容

```
apiVersion: apps/v1 # for versions before 1.16.0 use extensions/v1beta1
kind: Deployment
metadata:
  name: nginx-deployment # deployment的名称
spec:
  selector:
    matchLabels:
      app: nginx # 选择器必须匹配 template中的metadata.labels
  replicas: 3 # tells deployment to run 3 pods matching the template
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.16.0
        ports:
        - containerPort: 80
```

创建deployment  

kubectl create -f ./nginx-deployment.yaml --record
```
[root@localhost k8s]# kubectl create -f ./nginx-deployment.yaml --record
deployment.apps/nginx-deployment created
```
kubectl get deployments 查看

```
[root@localhost k8s]# kubectl get deployments
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           62s
```
- NAME 列出群集中的部署名称。
- DESIRED 显示应用程序的所需副本数，您在创建部署时定义这些副本。这是理想的状态。
- CURRENT 显示当前正在运行的副本数量。
- UP-TO-DATE 显示已更新以实现所需状态的副本数。
- AVAILABLE 显示用户可以使用的应用程序副本数。
- AGE 显示应用程序运行的时间。

**我们为刚才的nginx-deployment创建服务对象:**  

Service  

一种抽象的方式暴露在一组运行的应用程序Pods作为网络服务。

使用Kubernetes，您无需修改​​应用程序即可使用不熟悉的服务发现机制。Kubernetes为Pods提供了自己的IP地址和一组Pod的单个DNS名称，并且可以在它们之间进行负载均衡。

**nginx-service.yaml 文件内容**

```
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  type: NodePort # 为POD开放端口
  ports:
  - protocol: TCP
    port: 80
    nodePort: 30000 # 30000流量转到80端口
```
**执行kubectl apply -f nginx-service.yaml来创建Service**


```
[root@localhost k8s]# kubectl apply -f nginx-service.yaml
service/nginx-service created

```
**如果你使用的是minikube, 你通过minikube service nginx-service --url来获取访问入口**


```
[root@localhost k8s]# minikube service nginx-service --url
http://192.168.1.128:30000
```

可以通过kubctl名来设置访问的url  

kubectl port-forward nginx-deployment-7d9d7464fb-mrc4h 30001:80

```
[root@localhost k8s]# kubectl port-forward nginx-deployment-9664d7db6-jfwvv  30001:80
Forwarding from 127.0.0.1:30001 -> 80
Forwarding from [::1]:30001 -> 80
```
- [ ] **ubectl port-forward需要的是pod名称, 你可以通过kubectl get pods得到名称.**

```
[root@localhost k8s]# kubectl get pod
NAME                               READY   STATUS    RESTARTS   AGE
nginx-deployment-9664d7db6-jfwvv   1/1     Running   0          10m
nginx-deployment-9664d7db6-p9zhh   1/1     Running   0          10m
nginx-deployment-9664d7db6-wgxg7   1/1     Running   0          10m
```
## 2、运行自己的go http web服务
- 创建自己的web服务
- 创建服务的docker镜像，需要些dockerfile文件
- 部署服务到k8s

### 创建自己的web服务
server.go 文件
```
package main

import (
   "net/http"
   "log"
   "io"
)

func hello(w http.ResponseWriter,r *http.Request)  {
   io.WriteString(w,"hello, kubetnetes!\n")
}

func main() {
   http.HandleFunc("/hello",hello)
   err := http.ListenAndServe(":7070",nil)
   if err != nil {
      log.Panic(err)
   }
}
```
编译之后可以运行起来。

[root@localhost src]# go build -o server server.go   
[root@localhost src]# ./server  
新开一个窗口进行curl，可以看到输出了hello, kubetnetes!

[root@localhost ~]# curl 192.168.42.131:7070/hello
hello, kubetnetes!

### 创建服务docker镜像
编写Dockerfile


```
# Use the official Golang image to create a build artifact.
# https://hub.docker.com/_/golang
FROM golang:latest as builder

# Copy local code to the container image.
WORKDIR /root/k8s/godockerfile
COPY . .

RUN GOOS=linux GOARCH=amd64 CGO_ENABLED=0 go build -o app http_server.go

# Use a Docker multi-stage build to create a lean production image.
# https://docs.docker.com/develop/develop-images/multistage-build/#use-multi-stage-builds
FROM alpine:latest

COPY --from=builder /root/k8s/godockerfile .

ENV PORT 7070

# Run the web service on container startup.
CMD ["./app"]
```
打包镜像： **docker build -t http_server.go:v1 .**

```
[root@localhost dockerfile]# docker build -t server:v1 .
Sending build context to Docker daemon  3.584kB
Step 1/8 : FROM golang:latest as builder
 ---> 5fbd6463d24b
Step 2/8 : WORKDIR /root/k8s/godockerfile
 ---> Using cache
 ---> e493573ef164
Step 3/8 : COPY . .
 ---> 885e68693538
Step 4/8 : RUN GOOS=linux GOARCH=amd64 CGO_ENABLED=0 go build -o app http_server.go
 ---> Running in 24a7027bdee3
Removing intermediate container 24a7027bdee3
 ---> c58ebb1735c1
Step 5/8 : FROM alpine:latest
latest: Pulling from library/alpine
df20fa9351a1: Pull complete 
Digest: sha256:185518070891758909c9f839cf4ca393ee977ac378609f700f60a771a2dfe321
Status: Downloaded newer image for alpine:latest
 ---> a24bb4013296
Step 6/8 : COPY --from=builder /root/k8s/godockerfile .
 ---> fc2c74b4c794
Step 7/8 : ENV PORT 7070
 ---> Running in 3b0dbbf02eea
Removing intermediate container 3b0dbbf02eea
 ---> c13bb216b91f
Step 8/8 : CMD ["./app"]
 ---> Running in a74c69d28545
Removing intermediate container a74c69d28545
 ---> 76bf3e25da25
Successfully built 76bf3e25da25
Successfully tagged server:v1

```
### 部署server到k8s
docker iamges 查看镜像名

```
[root@localhost dockerfile]# docker images
REPOSITORY                                                                    TAG                 IMAGE ID            CREATED             SIZE
server                                                                        v1                  76bf3e25da25        3 minutes ago       13MB
```

使用 yaml 文件

```
touch hello-kubernetes.yaml
```


```
apiVersion: v1
kind: Service
metadata:
  name: hello-kubernetes
spec:
  selector:
    app: hello-kubernetes
  type: NodePort # 为POD开放端口
  ports:
  - protocol: TCP
    port: 7070
    nodePort: 30001 # 30000流量转到80端口
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-kubernetes
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello-kubernetes
  template:
    metadata:
      labels:
        app: hello-kubernetes
    spec:
      containers:
      - name: hello-kubernetes
        image: server:v1
        imagePullPolicy: Never
        ports:
        - containerPort: 7070
```
###  执行部署

kubectl apply -f hello-kubernetes.yaml


```
root@localhost dockerfile]# kubectl apply -f hello-kubernetes.yaml
service/hello-kubernetes created
deployment.apps/hello-kubernetes created
```
kubectl get po 查看一下

```
[root@localhost dockerfile]# kubectl get svc
NAME               TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
hello-kubernetes   NodePort    10.110.213.111   <none>        7070:30001/TCP   55s
```
kubectl get svc 查看一下

```
[root@localhost dockerfile]# kubectl get svc
NAME               TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
hello-kubernetes   NodePort    10.110.213.111   <none>        7070:30001/TCP   94s
```
**查看一下url**

```
[root@localhost dockerfile]# minikube service hello-kubernetes  --url
http://192.168.1.128:30001
```
curl或者在浏览器中访问:

```
[root@localhost dockerfile]# curl http://192.168.1.128:30001/hello
hello, kubetnetes!
```


### 删除部署

```
$ kubectl delete -f hello-kubernetes.yaml
```









参考文章：  

- [Minikube安装成功Kubernetes,一次过！](https://www.cnblogs.com/sanshengshui/p/11228985.html)
- https://www.cnblogs.com/0pandas0/p/12002386.html
- https://yq.aliyun.com/articles/221687
- https://www.cnblogs.com/0pandas0/p/12002386.html
- https://blog.csdn.net/weixin_45536736/article/details/104087487
- https://loocode.com/post/10174