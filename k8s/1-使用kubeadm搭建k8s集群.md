# 环境准备

准备ecs 两台， 系统 ubuntu

## 下载docker

- step 1: 安装必要的一些系统工具
``` bash
sudo apt-get update
sudo apt-get -y install apt-transport-https ca-certificates curl software-properties-common 
```
- step 2: 安装GPG证书
``` bash
curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
```
- Step 3: 写入软件源信息
``` bash
sudo add-apt-repository "deb [arch=amd64] https://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
```
- Step 4: 查找对应的docker 版本信息
``` bash
sudo apt-cache madsion docker-ce
```
- Step 5: 下载对应版本docker-ce
``` bash
sudo apt-get install docker-ce=*******
```
- Step 6: 下载完成后将当前用户名加入docker组 
```bash
sudo groupadd docker
sudo gpasswd -a ${USER} docker
sudo systemctl restart docker
sudo newgrp - docker
```
- Step 7: docker version 查看是否安装完成， docker images 能否正常运行
``` bash
docker version
docker images
docker ps
```

## 下载kubeadm, kubelet, kubectl (以kubeadm=1.13.1 为准)
- Step1: 替换下载源 
``` bash
sudo vim /etc/apt/source.list.d/kubernetes.list
sudo apt-get install kubeadm=1.13.1-00 kubectl=1.13.1-00 kubelet=1.13.1-00
```
 - Step2: 下载依赖项 
 ``` bash
 sudo apt-get install kubernetes-cni=0.6.0-00
 ```
 - Step3: 下载kubeadm， kubelte， kubectl。 
 ``` bash
 sudo apt-get install kubeadm=1.13.1-00 kubectl=1.13.1-00 kubelet=1.13.1-00
```
- Step4: 国内拉取kubeadm 所需要的镜像
``` bash
# 查看kubeadm 所需要的镜像名称
kubeadm config images list
>> k8s.gcr.io/kube-apiserver:v1.13.1
>> k8s.gcr.io/kube-controller-manager:v1.13.1
>> k8s.gcr.io/kube-scheduler:v1.13.1
>> k8s.gcr.io/kube-proxy:v1.13.1
>> k8s.gcr.io/pause:3.1
>> k8s.gcr.io/etcd:3.2.24
>> k8s.gcr.io/coredns:1.2.6
# 去docker.io 中搜索仓库地址 传送门: https://hub.docker.com/u/mirrorgooglecontainers/
docker pull docker.io/mirrorgooglecontainers/kube-apiserver:v1.13.1
# 替换tag 重复下面动作， 直到所有的镜像全部下载完成
docker tag docker.io/mirrorgooglecontainer/kube-apiserver:v1.13.1 k8s.gcr.io/kube-apiserver:v1.13.1
```
- Step5: 初始化k8s
``` bash
sudo kubeadm init --kubernetes-version=1.13.1 --pod-network-cidr=10.244.0.0/16 --service-cidr=172.10.0.0/16

# 记下来 后面的输出 kubeadm join *****， node 节点加入集群使用
# 创建使用的 网络组件（这里使用 flannel）
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
# 先删除master 节点上的污点， 因为master 不允许容器调度， coredns 需要调度到master 节点上面
kubectl get node # 查看节点
kubectl describe node ***** # 查看节点详情 taint 为污点
kubectl taint node ***** key- 
```
# node 节点
- Step1：安装 kubeadm， docker 参考master 节点安装步骤 
- Step2：加入集群
```bash
kubeadm join *****
```
- Step3: master 节点查看
``` bash
kubectl get node
```