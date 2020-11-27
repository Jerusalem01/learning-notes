## Kubernetes

#### 1.创建镜像虚拟机并进行统一资源配置

```
关闭交换空间   
sudo swapoff -a
避免开机启动交换空间
vi /etc/fstab 
注释swap所在行
关闭防火墙 
ufw disable
配置DNS   
vi /etc/systemd/resolved.conf   
修改 DNS=114.114.114.114
```

- **安装Docker**

```
更新软件源  
sudo apt-get update 

安装所需依赖   
sudo apt-get -y install apt-transport-https ca-certificates curl software-properties-common

安装 GPG 证书   
curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add - 

新增软件源信息  
sudo add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"

再次更新软件源   
sudo apt-get -y update 

安装 Docker CE 版   
sudo apt-get -y install docker-ce

验证   
docker version   

配置加速器 
vi /etc/docker/daemon.json  
粘贴（采用paste方法）   
{     
  "registry-mirrors": [   
    "https://wzwr26i4.mirror.aliyuncs.com"   
    ]   
}

重启
systemctl restart docker

验证   
docker info
```

- **安装必备工具**

```
安装系统工具  
apt-get update && apt-get install -y apt-transport-https   

安装 GPG 证书  
curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add -  

写入软件源 
   cat << EOF > /etc/apt/sources.list.d/kubernetes.list`  
> deb https://mirrors.aliyun.com/kubernetes/apt/kubernetes-xenial main`  
> EOF

安装   
apt-get update && apt-get install -y kubelet kubeadm kubectl

设置 kubelet 自启动，并启动 kubelet   
systemctl enable kubelet && systemctl start kubelet
```

- **同步时间**

```
时区同步   
dpkg-reconfigure txdata   
选择亚洲上海

时间同步   
apt-get install ntpdate  
ntpdate cn.pool.ntp.org   
hwclock --systohc 

确认时间   
date
```

- **修改cloud.cfg**

```
vi /etc/cloud/cloud.cfg    
修改 preserve_hostname: true
```

- **重启**
  `reboot`

- **关机**
  `shutdown -h now`

#### 2.基于镜像克隆虚拟机并进行单一资源配置

- **配置IP**
  `vi /etc/netplan/50-cloud-init.yaml`

```
添加   
network:   
   ethernets:  
         ens33:   
          addresses: [192.168.137.138/24]   
          gateway4: 192.168.137.1   
          nameservers:   
            addresses: [192.168.137.1]  
   version: 2   
netplan apply
```

- **配置主机名**

```
# 修改主机名
hostnamectl set-hostname kubernetes-master 
# 配置 hosts  
   cat >> /etc/hosts << EOF`  
> 192.169.137.138 kubernetes-master   
> EOF  
```

- **重启**
  `reboot`

#### 3.第一个kubernetes容器

- **创建工作目录并导出配置文件**

```
# 创建工作目录
mkdir -p /usr/local/kubernetes/cluster

# 导出配置文件到工作目录
kubeadm config print init-defaults --kubeconfig ClusterConfiguration > kubeadm.yml 
vi kubeadm.yml
```

- **修改配置yml文件**

```
修改为主节点IP  
advertiseAddress: 192.168.137.138 

国内不能访问 Google，修改为阿里云  
imageRepository:registry.aliyuncs.com/google_containers  

修改版本号   
kubernetesVersion: v1.14.1  
（注意版本号，防止阻塞）  

添加配置 Calico 的默认网段   
podSubnet: "192.168.0.0/16"   

开启 IPVS 模式   
apiVersion: kubeproxy.config.k8s.io/v1alpha1
```

- **查看所需镜像列表**
  `kubeadm config images list --config kubeadm.yml`

- **拉取镜像**
  `kubeadm config images pull --config kubeadm.yml`

- **安装 kubernetes 主节点**

```
kubeadm init --config=kubeadm.yml --experimental-upload-certs | tee kubeadm-init.log
```

- **配置 kubectl**

```
# kubeadm 初始化
kubeadm init --config=kubeadm.yml --experimental-upload-certs | tee kubeadm-init.log

# 配置 kubectl
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config(非root用户执行)

# 验证是否成功
kubectl get node  
```

- **安装从节点**
  复制粘贴

- **安装网络插件 Calico**

```
# 安装 Calico
kubectl apply -f https://docs.projectcalico.org/v3.7/manifests/calico.yaml

# 验证安装是否成功
watch kubectl get pods --all-namespaces
（需要等待所有状态为 Running，注意时间可能较久，3 - 5 分钟的样子，并再次验证）  
```

- **检查组件运行状态**
  `kubectl get cs`

- **检查 Master 状态**
  `kubectl cluster-info`

- **检查 Nodes 状态**
  `kubectl get nodes`

- **运行实例**
  `kubectl run nginx --image=nginx --replicas=2 --port=80`
  （使用 kubectl 命令创建两个监听 80 端口的 Nginx Pod（Kubernetes 运行容器的最小单元））

- **查看全部 Pods 的状态**
  `kubectl get pods`

- **查看已部署的服务**
  `kubectl get deployment`

- **映射服务，让用户可以访问**
  `kubectl expose deployment nginx --port=80 --type=LoadBalancer`

- **查看已发布的服务**
  `kubectl get services`

- **查看服务详情**
  `kubectl describe service nginx`

- **验证是否成功**
  通过浏览器访问 Master 服务器
  `http://192.168.141.130:31738/`
  (此时 Kubernetes 会以负载均衡的方式访问部署的 Nginx 服务，能够正常看到 Nginx 的欢迎页即表示成功。容器实际部署在其它 Node 节点上，通过访问 Node 节点的 IP:Port 也是可以的)

- **停止服务**
  `kubectl delete deployment nginx`
  `kubectl delete service nginx`

- **通过资源配置运行容器**
  `cd /usr/local/kubernetes/`
  `mkdir service`
  `cd service/`
  `vi nginx.yml`

```
粘贴  
v1.16.0 之前 

apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx-app
spec:
  replicas: 2
  template:
    metadata:
      labels:
        name: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-http
spec:
  ports:
    - port: 80
      targetPort: 80
      # 可以指定 NodePort 端口，默认范围是：30000-32767
      # nodePort: 30080
  type: LoadBalancer
  selector:
    name: nginx
    
v1.16.0 之后 

# API 版本号：由 extensions/v1beta1 修改为 apps/v1
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-app
spec:
  # 增加了选择器配置
  selector:
    matchLabels:
      app: nginx
  replicas: 2
  template:
    metadata:
      labels:
        # 设置标签由 name 修改为 app
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-http
spec:
  ports:
    - port: 80
      targetPort: 80
  type: LoadBalancer
  selector:
    # 标签选择器由 name 修改为 app
    app: nginx
部署   
kubectl create -f nginx.yml 
```

- **验证是否生效**

```
查看 Pod 列表
kubectl get pods

# 输出如下
NAME                         READY   STATUS    RESTARTS   AGE
nginx-app-64bb598779-2pplx   1/1     Running   0          25m
nginx-app-64bb598779-824lc   1/1     Running   0          25m

#查看 Deployment 列表
kubectl get deployment

# 输出如下
NAME        READY   UP-TO-DATE   AVAILABLE   AGE
nginx-app   2/2     2            2           25m

#查看 Service 列表
kubectl get service

# 输出如下
NAME         TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP      10.96.0.1      <none>        443/TCP        20h
nginx-http    LoadBalancer   10.98.49.142   <pending>     80:31631/TCP   14m

#查看 Service 详情
kubectl describe service nginx-app 

# 输出如下
Name:                     nginx-http
Namespace:                default
Labels:                   <none>
Annotations:              <none>
Selector:                 name=nginx
Type:                     LoadBalancer
IP:                       10.98.49.142
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  31631/TCP
Endpoints:                10.244.141.205:80,10.244.2.4:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
通过浏览器访问   
 http://192.168.141.150:31631/
 出现 Nginx 欢迎页即表示成功 
# 删除
kubectl delete -f nginx-service.yml   
```

