# Kubernetes安装教程（Kubeadm）
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
* 相关组件的介绍
  * kubeadm：the command to bootstrap the cluster
  * kubelet：the component that runs on all of the machines in your cluster and does things like starting pods and containers
  * kubectl：the command line util to talk to your cluster
* 在所有节点执行以下脚本进行安装
```
apt-get update && apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update
apt-get install -y kubelet kubeadm kubectl
```
* 由于没有科学上网，所以在执行到sudo apt-get update的时候失败了，报错
```
could not fetch from http://apt.kubernetes.io/
```
* 如果上述步骤无法成功下载，可以直接在packages中下载kubeadm、kubelet、kubectl的安装包，执行sudo dpkg -i安装。


## 拉取相关镜像（非常关键的一步）
理论上来说在成功下载好了上述的组件之后，可以直接跳过本步骤直接进行主节点的初始化，但是由于在进行初始化的时候会默认从google_containers中拉取kubernetes所需要的相关镜像（非常慢，很可能最终导致初始化失败）。我已经把相关的 images push到了我的dockerhub当中，可以直接执行以下脚本进行镜像的拉取（images的tag很重要，以下的代码仅在相对应的kuberadm、kuberntes版本中适用）
```
images=(
   kube-proxy-amd64:v1.8.4
   kube-discovery-amd64:1.0
   kubedns-amd64:1.9
   kube-scheduler-amd64:v1.8.4
   kube-controller-manager-amd64:v1.8.4
   kube-apiserver-amd64:v1.8.4
   etcd-amd64:3.0.17
   kube-dnsmasq-amd64:1.4
   pause-amd64:3.0
   kubernetes-dashboard-amd64:v1.6.3
   k8s-dns-sidecar-amd64:1.14.5
   k8s-dns-dnsmasq-nanny-amd64:1.14.5
   k8s-dns-kube-dns-amd64:1.14.5
   exechealthz-amd64:1.2)
for imageName in ${images[@]} ; do
  docker pull huangchuo/$imageName
  docker tag huangchuo/$imageName gcr.io/google_containers/$imageName
  docker rmi huangchuo/$imageName
done
```
## 初始化主节点
```
sudo systemctl enable kubelet
sudo systemctl start kubelet
sudo kubeadm init --pod-network-cidr=192.168.0.0/16 --kubernetes-version v1.8.4
```
主节点配置好后会生成一段token，使用这个token让其他节点加入这个集群。
