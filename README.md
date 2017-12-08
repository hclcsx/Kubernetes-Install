# Kubernetes-Install
使用 kubeadm 安装Kubernetes1.8.1安装过程（ plus 采坑历程 ）。安装采用了两台ubuntu16.04的虚拟机，一台作为masterode主机、一台作为workernode
## 工具版本
* Docker：17.09-ce
* Kubeadm：1.8.4
* Ubuntu：16.04
* Kubernetes: 1.8.1

## 安装 Docker
需要在所有节点安装docker
```
apt-get update
apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") \
   $(lsb_release -cs) \
   stable"
apt-get update && apt-get install -y docker-ce=$(apt-cache madison docker-ce | grep 17.09 | head -1 | awk '{print $3}')
```

## 安装 Kubedm Kubelet Kubectl
* kubeadm：the command to bootstrap the cluster
* kubelet：the component that runs on all of the machines in your cluster and does things like starting pods and containers
* kubectl：the command line util to talk to your cluster
```
apt-get update && apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update
apt-get install -y kubelet kubeadm kubectl
```
* 由于在安装的时候可能会链接不上，所以上传了kubeadm、kubelet、kubectl的安装包，可以直接下载安装

## 初始化主节点
