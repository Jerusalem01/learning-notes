## Kubernetes

### 1. 概述

##### [Kubernetes 是什么？](https://kubernetes.io/zh/docs/concepts/overview/what-is-kubernetes/)

Kubernetes 是一个可移植的，可扩展的开源平台，用于管理容器化的工作负载和服务，方便了声明式配置和自动化。它拥有一个庞大且快速增长的生态系统。Kubernetes 的服务，支持和工具广泛可用。

##### [Kubernetes 组件](https://kubernetes.io/zh/docs/concepts/overview/components/)

Kubernetes 集群由代表控制平面的组件和一组称为节点的机器组成。

##### [Kubernetes API](https://kubernetes.io/zh/docs/concepts/overview/kubernetes-api/)

Kubernetes API 使你可以查询和操纵 Kubernetes 中对象的状态。 Kubernetes 控制平面的核心是 API 服务器和它暴露的 HTTP API。 用户、集群的不同部分以及外部组件都通过 API 服务器相互通信。

##### [使用 Kubernetes 对象](https://kubernetes.io/zh/docs/concepts/overview/working-with-objects/)

Kubernetes 对象是 Kubernetes 系统中的持久性实体。Kubernetes 使用这些实体表示您的集群状态。了解 Kubernetes 对象模型以及如何使用这些对象。

### 2. Kubernetes实践

#### 2.1 统一环境配置

```
关闭交换空间   
swapoff -a

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

配置加速器 
vi /etc/docker/daemon.json  
粘贴（采用paste方法）   
{     
  "registry-mirrors": [   
    "https://wzwr26i4.mirror.aliyuncs.com"   
    ]   
}

开机自启动
systemctl enable docker

重启
systemctl restart docker

验证   
docker version

验证   
docker info
```

- **安装必备工具（kubeadm，kubelet，kubectl）**

```
安装系统工具  
apt-get update && apt-get install -y apt-transport-https   

安装 GPG 证书  
curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add -  

写入软件源 
	cat << EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
EOF

安装   
apt-get update && apt-get install -y kubelet kubeadm kubectl

设置 kubelet 自启动，并启动 kubelet   
systemctl enable kubelet && systemctl start kubelet
```

- **同步时间**

```
时区同步-选择亚洲上海
sudo timedatectl set-timezone 'Asia/Shanghai'   

安装 ntpdate  
apt-get install ntpdate

设置系统时间与网络时间同步
ntpdate cn.pool.ntp.org

将系统时间写入硬件时间
hwclock --systohc 

确认时间   
date
```

- **配置 IPVS**

```bash
# 安装系统工具
apt-get install -y ipset ipvsadm

# 配置并加载 IPVS 模块
mkdir -p /etc/sysconfig/modules/
vi /etc/sysconfig/modules/ipvs.modules

# 输入如下内容
#!/bin/bash
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack_ipv4

# 执行脚本，注意：如果重启则需要重新运行该脚本
chmod 755 /etc/sysconfig/modules/ipvs.modules && bash /etc/sysconfig/modules/ipvs.modules && lsmod | grep -e ip_vs -e nf_conntrack_ipv4

# 执行脚本输出如下
ip_vs_sh               16384  0
ip_vs_wrr              16384  0
ip_vs_rr               16384  0
ip_vs                 147456  6 ip_vs_rr,ip_vs_sh,ip_vs_wrr
nf_conntrack_ipv4      16384  3
nf_defrag_ipv4         16384  1 nf_conntrack_ipv4
nf_conntrack          131072  8 xt_conntrack,nf_nat_masquerade_ipv4,nf_conntrack_ipv4,nf_nat,ipt_MASQUERADE,nf_nat_ipv4,nf_conntrack_netlink,ip_vs
libcrc32c              16384  4 nf_conntrack,nf_nat,raid456,ip_vs
```

- **配置内核参数**

```bash
# 配置参数
vi /etc/sysctl.d/k8s.conf

# 输入如下内容
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_nonlocal_bind = 1
net.ipv4.ip_forward = 1
vm.swappiness=0

# 应用参数
sysctl --system

# 应用参数输出如下（找到 Applying /etc/sysctl.d/k8s.conf 开头的日志）
* Applying /etc/sysctl.d/10-console-messages.conf ...
kernel.printk = 4 4 1 7
* Applying /etc/sysctl.d/10-ipv6-privacy.conf ...
* Applying /etc/sysctl.d/10-kernel-hardening.conf ...
kernel.kptr_restrict = 1
* Applying /etc/sysctl.d/10-link-restrictions.conf ...
fs.protected_hardlinks = 1
fs.protected_symlinks = 1
* Applying /etc/sysctl.d/10-lxd-inotify.conf ...
fs.inotify.max_user_instances = 1024
* Applying /etc/sysctl.d/10-magic-sysrq.conf ...
kernel.sysrq = 176
* Applying /etc/sysctl.d/10-network-security.conf ...
net.ipv4.conf.default.rp_filter = 1
net.ipv4.conf.all.rp_filter = 1
net.ipv4.tcp_syncookies = 1
* Applying /etc/sysctl.d/10-ptrace.conf ...
kernel.yama.ptrace_scope = 1
* Applying /etc/sysctl.d/10-zeropage.conf ...
vm.mmap_min_addr = 65536
* Applying /usr/lib/sysctl.d/50-default.conf ...
net.ipv4.conf.all.promote_secondaries = 1
net.core.default_qdisc = fq_codel
* Applying /etc/sysctl.d/99-sysctl.conf ...
* Applying /etc/sysctl.d/k8s.conf ...
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_nonlocal_bind = 1
net.ipv4.ip_forward = 1
vm.swappiness = 0
* Applying /etc/sysctl.conf ...
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

#### 2.2 单一节点配置

- **配置IP**
  
  ````
  vi /etc/netplan/50-cloud-init.yaml
  ````

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

# 让配置生效
netplan apply
```

- **配置主机名**

```
# 修改主机名
hostnamectl set-hostname Kubernetes-master 
# 配置 hosts  
cat >> /etc/hosts << EOF
192.168.141.163 Kubernetes-master
EOF
```

- **重启**
  `reboot`

#### 2.3 安装 HAProxy + Keepalived

##### 2.3.1 概述

Kubernetes Master 节点运行组件如下：

- **kube-apiserver：** 提供了资源操作的唯一入口，并提供认证、授权、访问控制、API 注册和发现等机制
- **kube-scheduler：** 负责资源的调度，按照预定的调度策略将 Pod 调度到相应的机器上
- **kube-controller-manager：** 负责维护集群的状态，比如故障检测、自动扩展、滚动更新等
- **etcd：** CoreOS 基于 Raft 开发的分布式 key-value 存储，可用于服务发现、共享配置以及一致性保障（如数据库选主、分布式锁等）

`kube-scheduler` 和 `kube-controller-manager` 可以以集群模式运行，通过 leader 选举产生一个工作进程，其它进程处于阻塞模式。

**`kube-apiserver` 可以运行多个实例，但对其它组件需要提供统一的访问地址，本章节部署 Kubernetes 高可用集群实际就是利用 HAProxy + Keepalived 配置该组件**

配置的思路就是利用 HAProxy + Keepalived 实现 `kube-apiserver` 虚拟 IP 访问从而实现高可用和负载均衡，拆解如下：

- Keepalived 提供 `kube-apiserver` 对外服务的虚拟 IP（VIP）
- HAProxy 监听 Keepalived VIP
- 运行 Keepalived 和 HAProxy 的节点称为 LB（负载均衡） 节点
- Keepalived 是一主多备运行模式，故至少需要两个 LB 节点
- Keepalived 在运行过程中周期检查本机的 HAProxy 进程状态，如果检测到 HAProxy 进程异常，则触发重新选主的过程，VIP 将飘移到新选出来的主节点，从而实现 VIP 的高可用
- 所有组件（如 kubeclt、apiserver、controller-manager、scheduler 等）都通过 VIP +HAProxy 监听的 6444 端口访问 `kube-apiserver` 服务（**注意：`kube-apiserver` 默认端口为 6443，为了避免冲突我们将 HAProxy 端口设置为 6444，其它组件都是通过该端口统一请求 apiserver**）

![负载均衡架构图](https://www.funtl.com/assets1/20190427104124213.png)

##### 2.3.2 创建 HAProxy 启动脚本

> 该步骤在 `kubernetes-master` 执行

```bash
mkdir -p /usr/local/kubernetes/lb
vi /usr/local/kubernetes/lb/start-haproxy.sh

# 输入内容如下
# !/bin/bash
# 修改为你自己的 Master 地址
MasterIP1=192.168.141.150
MasterIP2=192.168.141.151
MasterIP3=192.168.141.152
# 这是 kube-apiserver 默认端口，不用修改
MasterPort=6443

# 容器将 HAProxy 的 6444 端口暴露出去
docker run -d --restart=always --name HAProxy-K8S -p 6444:6444 \
        -e MasterIP1=$MasterIP1 \
        -e MasterIP2=$MasterIP2 \
        -e MasterIP3=$MasterIP3 \
        -e MasterPort=$MasterPort \
        wise2c/haproxy-k8s

# 设置权限
chmod +x start-haproxy.sh
```

##### 2.3.3 创建 Keepalived 启动脚本

> 该步骤在 `kubernetes-master` 执行

```bash
mkdir -p /usr/local/kubernetes/lb
vi /usr/local/kubernetes/lb/start-keepalived.sh

# 输入内容如下
#!/bin/bash
# 修改为你自己的虚拟 IP 地址
VIRTUAL_IP=192.168.141.200
# 虚拟网卡设备名
INTERFACE=ens33
# 虚拟网卡的子网掩码
NETMASK_BIT=24
# HAProxy 暴露端口，内部指向 kube-apiserver 的 6443 端口
CHECK_PORT=6444
# 路由标识符
RID=10
# 虚拟路由标识符
VRID=160
# IPV4 多播地址，默认 224.0.0.18
MCAST_GROUP=224.0.0.18

docker run -itd --restart=always --name=Keepalived-K8S \
        --net=host --cap-add=NET_ADMIN \
        -e VIRTUAL_IP=$VIRTUAL_IP \
        -e INTERFACE=$INTERFACE \
        -e CHECK_PORT=$CHECK_PORT \
        -e RID=$RID \
        -e VRID=$VRID \
        -e NETMASK_BIT=$NETMASK_BIT \
        -e MCAST_GROUP=$MCAST_GROUP \
        wise2c/keepalived-k8s

# 设置权限
chmod +x start-keepalived.sh
```

##### 2.3.4 复制脚本到其它 Master 地址（主节点不止一个时需要执行）

分别在 `kubernetes-master-02` 和 `kubernetes-master-03` 执行创建工作目录命令

```bash
mkdir -p /usr/local/kubernetes/lb
```

将 `kubernetes-master-01` 中的脚本拷贝至其它 Master

```bash
scp start-haproxy.sh start-keepalived.sh 192.168.141.151:/usr/local/kubernetes/lb
scp start-haproxy.sh start-keepalived.sh 192.168.141.152:/usr/local/kubernetes/lb
```

分别在 3 个 Master 中启动容器（执行脚本）

```bash
sh /usr/local/kubernetes/lb/start-haproxy.sh && sh /usr/local/kubernetes/lb/start-keepalived.sh
```

##### 2.3.5 验证是否成功

- **查看容器**

```bash
docker ps

# 输出如下
CONTAINER ID        IMAGE                   COMMAND                  CREATED             STATUS              PORTS                    NAMES
f50df479ecae        wise2c/keepalived-k8s   "/usr/bin/keepalived…"   About an hour ago   Up About an hour                             Keepalived-K8S
75066a7ed2fb        wise2c/haproxy-k8s      "/docker-entrypoint.…"   About an hour ago   Up About an hour    0.0.0.0:6444->6444/tcp   HAProxy-K8S
```

- **查看网卡绑定的虚拟 IP**

```bash
ip a | grep ens33

# 输出如下
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    inet 192.168.141.151/24 brd 192.168.141.255 scope global ens33
    inet 192.168.141.200/24 scope global secondary ens33
```

> 特别注意：Keepalived 会对 HAProxy 监听的 6444 端口进行检测，如果检测失败即认定本机 HAProxy 进程异常，会将 VIP 漂移到其他节点，所以无论本机 Keepalived 容器异常或 HAProxy 容器异常都会导致 VIP 漂移到其他节点

#### 2.4 部署 Kubernetes 集群

##### 2.3.1 初始化 Master

- **创建工作目录并导出配置文件**

```
# 创建工作目录
mkdir -p /usr/local/kubernetes/cluster

# 导出配置文件到工作目录
kubeadm config print init-defaults --kubeconfig ClusterConfiguration > kubeadm.yml 
```

- **修改配置文件**

```
vi kubeadm.yml

apiVersion: kubeadm.k8s.io/v1beta1
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  # 修改为主节点 IP
  advertiseAddress: 192.168.141.150
  bindPort: 6443
nodeRegistration:
  criSocket: /var/run/dockershim.sock
  name: kubernetes-master
  taints:
  - effect: NoSchedule
    key: node-role.kubernetes.io/master
---
apiServer:
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta1
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
# 配置 Keepalived 地址和 HAProxy 端口
controlPlaneEndpoint: "192.168.141.200:6444"
controllerManager: {}
dns:
  type: CoreDNS
etcd:
  local:
    dataDir: /var/lib/etcd
# 国内不能访问 Google，修改为阿里云
imageRepository: registry.aliyuncs.com/google_containers
kind: ClusterConfiguration
# 修改版本号
kubernetesVersion: v1.14.2
networking:
  dnsDomain: cluster.local
  # 配置成 Calico 的默认网段
  podSubnet: "10.244.0.0/16"
  serviceSubnet: 10.96.0.0/12
scheduler: {}
---
# 开启 IPVS 模式
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
featureGates:
  SupportIPVSProxyMode: true
mode: ipvs
```

- **kubeadm 初始化**

```
# kubeadm 初始化（至少两核）(可提前拉取镜像)
kubeadm init --config=kubeadm.yml --upload-certs | tee kubeadm-init.log
（
kubeadm init --kubernetes-version=1.19.4  \
--apiserver-advertise-address=192.168.75.161   \
--image-repository registry.cn-hangzhou.aliyuncs.com/google_containers  \
--service-cidr=10.96.0.0/16  \
--pod-network-cidr=10.244.0.0/16
）

# 配置 kubectl
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config

# 验证是否成功
kubectl get node 
```

- **安装网络插件 Calico**

```
# 下载 Calico 配置文件并修改
wget https://docs.projectcalico.org/v3.7/manifests/calico.yaml

vi calico.yaml

修改第 611 行，将 192.168.0.0/16 修改为 10.244.0.0/16，可以通过如下命令快速查找
显示行号：:set number
查找字符：/要查找的字符，输入小写 n 下一个匹配项，输入大写 N 上一个匹配项

# 安装 Calico
kubectl apply -f calico.yaml
# 输出如下
configmap/calico-config created
customresourcedefinition.apiextensions.k8s.io/felixconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamblocks.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/blockaffinities.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamhandles.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamconfigs.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgppeers.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgpconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ippools.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/hostendpoints.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/clusterinformations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworksets.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networksets.crd.projectcalico.org created
clusterrole.rbac.authorization.k8s.io/calico-kube-controllers created
clusterrolebinding.rbac.authorization.k8s.io/calico-kube-controllers created
clusterrole.rbac.authorization.k8s.io/calico-node created
clusterrolebinding.rbac.authorization.k8s.io/calico-node created
daemonset.extensions/calico-node created
serviceaccount/calico-node created
deployment.extensions/calico-kube-controllers created
serviceaccount/calico-kube-controllers created

# 验证安装是否成功
watch kubectl get pods --all-namespaces
（需要等待所有状态为 Running，注意时间可能较久，3 - 5 分钟的样子，并再次验证）  
```

##### 2.3.2 加入 Master 节点

从 `kubeadm-init.log` 中获取命令，分别将 `kubernetes-master-02` 和 `kubernetes-master-03` 加入 Master

```bash
# 以下为示例命令
kubeadm join 192.168.141.200:6444 --token abcdef.0123456789abcdef \
  --discovery-token-ca-cert-hash sha256:56d53268517c132ae81c868ce99c44be797148fb2923e59b49d73c99782ff21f \
  --experimental-control-plane --certificate-key c4d1525b6cce4b69c11c18919328c826f92e660e040a46f5159431d5ff0545bd
```

##### 2.3.3 加入 Node 节点

从 `kubeadm-init.log` 中获取命令，分别将 `kubernetes-node-01` 至 `kubernetes-node-03` 加入 Node

```bash
# 以下为示例命令
kubeadm join 192.168.141.200:6444 --token abcdef.0123456789abcdef \
    --discovery-token-ca-cert-hash sha256:56d53268517c132ae81c868ce99c44be797148fb2923e59b49d73c99782ff21f 
```

##### 2.3.4 验证集群状态

- **查看 Node**

```bash
kubectl get nodes -o wide
```

- **查看 Pod**

```bash
kubectl -n kube-system get pod -o wide
```

- **查看 Service**

```bash
kubectl -n kube-system get svc
```

- **验证 IPVS**

查看 kube-proxy 日志，server_others.go:176] Using ipvs Proxier.

```bash
kubectl -n kube-system logs -f <kube-proxy 容器名>
```

- **查看代理规则**

```bash
ipvsadm -ln
```

- **查看生效的配置**

```bash
kubectl -n kube-system get cm kubeadm-config -oyaml
```

- **查看 etcd 集群**

```bash
kubectl -n kube-system exec etcd-kubernetes-master-01 -- etcdctl \
	--endpoints=https://192.168.75.150:2379 \
	--ca-file=/etc/kubernetes/pki/etcd/ca.crt \
	--cert-file=/etc/kubernetes/pki/etcd/server.crt \
	--key-file=/etc/kubernetes/pki/etcd/server.key cluster-health

# 输出如下
member 1dfaf07371bb0cb6 is healthy: got healthy result from https://192.168.141.152:2379
member 2da85730b52fbeb2 is healthy: got healthy result from https://192.168.141.150:2379
member 6a3153eb4faaaffa is healthy: got healthy result from https://192.168.141.151:2379
cluster is healthy
```

##### 2.3.5 验证高可用

> 特别注意：Keepalived 要求至少 2 个备用节点，故想测试高可用至少需要 1 主 2 从模式验证，否则可能出现意想不到的问题

对任意一台 Master 机器执行关机操作

```bash
shutdown -h now
```

在任意一台 Master 节点上查看 Node 状态

```bash
kubectl get node

# 输出如下，除已关机那台状态为 NotReady 其余正常便表示成功
NAME                   STATUS   ROLES    AGE   VERSION
kubernetes-master-01   NotReady master   18m   v1.14.2
kubernetes-master-02   Ready    master   17m   v1.14.2
kubernetes-master-03   Ready    master   16m   v1.14.2
```

查看 VIP 漂移

```bash
ip a |grep ens33

# 输出如下
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    inet 192.168.141.151/24 brd 192.168.141.255 scope global ens33
    inet 192.168.141.200/24 scope global secondary ens33
```

### 3. Kubernetes命令

#### 3.1 Kubectl 与 Docker 命令

##### 概述

Docker 命令和 Kubectl 命令有很多相似的地方，Docker 操作容器，Kubectl 操作 Pod（容器的集合）等

##### 运行容器

- docker：`docker run -d --restart=always -e DOMAIN=cluster --name nginx-app -p 80:80 nginx`
- kubectl：
  - `kubectl run --image=nginx nginx-app --port=80 --env="DOMAIN=cluster"`
  - `kubectl expose deployment nginx-app --port=80 --name=nginx-http`

> **注意：** `kubectl run` 会创建一个 **Deployment** 并且默认会在后台运行，以上面的代码为例它的名称为 **nginx-app**。默认情况 Deployment 并不会将端口暴露出去，所以我们还需要使用 `kubectl expose` 暴露端口以供访问，此时还会创建一个同名的 **Service**

##### 查看已运行的容器

- docker：`docker ps`
- kubectl：
  - `kubectl get pods`
  - `kubectl get deployment`
  - `kubectl get service`
  - `kubectl get all`

##### 交互式进入容器

- docker：`docker exec -it <容器 ID/NAME> /bin/bash`
- kubectl：`kubectl exec -it <容器名> -- /bin/bash`

##### 打印日志

- docker：`docker logs -f <容器 ID/NAME>`
- kubectl：`kubectl logs -f <容器名>`

##### 停止和删除容器

- docker：
  - `docker stop <容器 ID/NAME>`
  - `docker rm <容器 ID/NAME>`
- kubectl：
  - `kubectl delete deployment <Deployment 名称>`
  - `kubectl delete service <Service 名称>`

> **注意：** 不要直接删除 Pod，使用 kubectl 请删除拥有该 Pod 的 Deployment。如果直接删除 Pod，则 Deployment 将会重新创建该 Pod。

##### 查看版本

- docker：`docker version`
- kubectl：`kubectl version`

##### 查看环境信息

- docker：`docker info`
- kubectl：`kubectl cluster-info`

#### 3.2 Kubectl 常用命令

> **小提示：** 所有命令前都可以加上 `watch` 命令来观察状态的实时变化，如：`watch kubectl get pods --all-namespaces`

- **查看组件状态**

```bash
kubectl get cs
```

- **查看环境信息**

```bash
kubectl cluster-info
```

- **查看 Node**

```bash
kubectl get nodes -o wide
```

- **查看集群配置**

```bash
kubectl -n kube-system get cm kubeadm-config -oyaml
```

- **运行容器**

```bash
kubectl run nginx --image=nginx --replicas=2 --port=80
```

- **暴露服务**

```bash
kubectl expose deployment nginx --port=80 --type=LoadBalancer
```

- **查看命名空间**

```bash
kubectl get namespace
```

- **创建命名空间**

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: development
```

- **查看容器**

```bash
kubectl get pods -o wide(默认只查看default命名空间)
kubectl get pods --all-namespaces
kubectl get deployment -o wide
```

- **查看服务**

```bash
kubectl get service -o wide
```

- **查看详情**

```bash
kubectl describe pod <Pod Name>
kubectl describe deployment <Deployment Name>
kubectl describe service <Service Name>
```

- **查看日志**

```bash
kubectl logs -f <Pod Name>
```

- **删除容器和服务**

```bash
kubectl delete deployment <Deployment Name>
kubectl delete service <Service Name>
```

- **配置方式运行**

```bash
kubectl create -f <YAML>
```

- **配置方式删除**

```bash
kubectl delete -f <YAML>
```

- **查看配置**

```bash
kubeadm config view
kubectl config view
```

- **查看 Ingress**

```bash
kubectl get ingress
```

- **查看持久卷**

```bash
kubectl get pv
```

- **查看持久卷消费者**

```bash
kubectl get pvc
```

- **查看 ConfigMap**

```bash
kubectl get cm <ConfigMap Name>
```

- **修改 ConfigMap**

```bash
kubectl edit cm <ConfigMap Name>
```

- **根据命令生成yaml文件**

```
kubectl create deployment tomcat6 --image=tomcat:6.0.53-jre8 --port=80 --target-port=8080 --type=NodePort --dry-run=client -o yaml > tomcat-deployment.yaml
```

- **部署并暴露Service**

```
kubectl apply -f tomcat-deployment.yaml
```

### 4. 部署Ingress

- **根据已有文件进行部署**

```
kubectl apply -f ingress-controller.yaml
```

- **创建规则**

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: web
spec:
  rules:
  - host: tomcat6.tesco.com
    http:
      paths:
        - backend:
           serviceName: tomcat6
           servicePort: 80
```

```
kubectl apply -f ingress-tomcat.yaml
```

### 5. Kubesphere-集群可视化工具

#### 5.1 前置环境

- `Kubernetes`版本： `1.15.x ≤ K8s version ≤ 1.17.x`；
- `Helm`版本： `2.10.0 ≤ Helm Version ＜ 3.0.0`，建议使用 `Helm 2.16.2`（不支持 helm 2.16.0 [#6894](https://github.com/helm/helm/issues/6894)），且已安装了 Tiller，参考 [如何安装与配置 Helm](https://devopscube.com/install-configure-helm-kubernetes/) （预计 3.0 支持 Helm v3）；
- 集群已有默认的存储类型（StorageClass），若还没有准备存储请参考 [安装 OpenEBS 创建 LocalPV 存储类型](https://v2-1.docs.kubesphere.io/docs/zh-CN/appendix/install-openebs) 用作开发测试环境。
- 集群能够访问外网，若无外网请参考 [在 Kubernetes 离线安装 KubeSphere](https://kubesphere.com.cn/docs/installation/install-on-k8s-airgapped/)。

#### 5.2 验证环境

1. 确认现有的 `Kubernetes`版本满足上述的前提条件，可以在执行 `kubectl version`来确认 :

```bash
$ kubectl version | grep Server
Server Version: version.Info{Major:"1", Minor:"13", GitVersion:"v1.13.5", GitCommit:"2166946f41b36dea2c4626f90a77706f426cdea2", GitTreeState:"clean", BuildDate:"2019-03-25T15:19:22Z", GoVersion:"go1.11.5", Compiler:"gc", Platform:"linux/amd64"}
```

> 说明：注意输出结果中的 `Server Version`这行，如果显示 `GitVersion`大于 `v1.13.0`，Kubernetes 的版本是可以安装的。如果低于 `v1.13.0`，可以查看 [Upgrading kubeadm clusters from v1.12 to v1.13](https://v1-13.docs.kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade-1-13/) 先升级下 K8s 版本。

2. 确认已安装 `Helm`，并且 `Helm`的版本至少为 `2.10.0`。在终端执行 `helm version`，得到类似下面的输出：

```bash
$ helm version
Client: &version.Version{SemVer:"v2.13.1", GitCommit:"618447cbf203d147601b4b9bd7f8c37a5d39fbb4", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.13.1", GitCommit:"618447cbf203d147601b4b9bd7f8c37a5d39fbb4", GitTreeState:"clean"}
```

> 说明：
>
> - 如果提示 `helm: command not found`, 表示还未安装 `Helm`。参考这篇 [Install Helm](https://helm.sh/docs/using_helm/#from-the-binary-releases) 安装 `Helm`, 安装完成后执行 `helm init`；
> - 如果 `helm`的版本比较老 (<2.10.0), 需要首先升级，参考 [Upgrading Tiller](https://github.com/helm/helm/blob/master/docs/install.md#upgrading-tiller) 升级。

3. 集群现有的可用内存至少在 `2G`以上，那么执行 `free -g`可以看下可用资源：

```bash
root@kubernetes:~# free -g
              total        used        free      shared  buff/cache   available
Mem:              16          4          10           0           3           2
Swap:             0           0           0
```

4. 集群已有存储类型（StorageClass），执行 `kubectl get sc`看下当前是否设置了默认的 `storageclass`。

```bash
root@kubernetes:~$ kubectl get sc
NAME                      PROVISIONER               AGE
ceph                      kubernetes.io/rbd         3d4h
csi-qingcloud (default)   disk.csi.qingcloud.com    54d
glusterfs                 kubernetes.io/glusterfs   3d4h
```

> 提示：若集群还没有准备存储请参考 [安装 OpenEBS 创建 LocalPV 存储类型](https://v2-1.docs.kubesphere.io/docs/zh-CN/appendix/install-openebs) 用作开发测试环境，生产环境请确保集群配置了稳定的持久化存储。

如果你的 Kubernetes 环境满足以上的要求，那么可以接着执行安装的步骤了。

#### 5.3 最小化安装

- **kubesphere-installer.yaml**

  ```
  ---
  apiVersion: apiextensions.k8s.io/v1beta1
  kind: CustomResourceDefinition
  metadata:
    name: clusterconfigurations.installer.kubesphere.io
  spec:
    group: installer.kubesphere.io
    versions:
    - name: v1alpha1
      served: true
      storage: true
    scope: Namespaced
    names:
      plural: clusterconfigurations
      singular: clusterconfiguration
      kind: ClusterConfiguration
      shortNames:
      - cc
  
  ---
  apiVersion: v1
  kind: Namespace
  metadata:
    name: kubesphere-system
  
  ---
  apiVersion: v1
  kind: ServiceAccount
  metadata:
    name: ks-installer
    namespace: kubesphere-system
  
  ---
  apiVersion: rbac.authorization.k8s.io/v1
  kind: ClusterRole
  metadata:
    name: ks-installer
  rules:
  - apiGroups:
    - ""
    resources:
    - '*'
    verbs:
    - '*'
  - apiGroups:
    - apps
    resources:
    - '*'
    verbs:
    - '*'
  - apiGroups:
    - extensions
    resources:
    - '*'
    verbs:
    - '*'
  - apiGroups:
    - batch
    resources:
    - '*'
    verbs:
    - '*'
  - apiGroups:
    - rbac.authorization.k8s.io
    resources:
    - '*'
    verbs:
    - '*'
  - apiGroups:
    - apiregistration.k8s.io
    resources:
    - '*'
    verbs:
    - '*'
  - apiGroups:
    - apiextensions.k8s.io
    resources:
    - '*'
    verbs:
    - '*'
  - apiGroups:
    - tenant.kubesphere.io
    resources:
    - '*'
    verbs:
    - '*'
  - apiGroups:
    - certificates.k8s.io
    resources:
    - '*'
    verbs:
    - '*'
  - apiGroups:
    - devops.kubesphere.io
    resources:
    - '*'
    verbs:
    - '*'
  - apiGroups:
    - monitoring.coreos.com
    resources:
    - '*'
    verbs:
    - '*'
  - apiGroups:
    - logging.kubesphere.io
    resources:
    - '*'
    verbs:
    - '*'
  - apiGroups:
    - jaegertracing.io
    resources:
    - '*'
    verbs:
    - '*'
  - apiGroups:
    - storage.k8s.io
    resources:
    - '*'
    verbs:
    - '*'
  - apiGroups:
    - admissionregistration.k8s.io
    resources:
    - '*'
    verbs:
    - '*'
  - apiGroups:
    - policy
    resources:
    - '*'
    verbs:
    - '*'
  - apiGroups:
    - autoscaling
    resources:
    - '*'
    verbs:
    - '*'
  - apiGroups:
    - networking.istio.io
    resources:
    - '*'
    verbs:
    - '*'
  - apiGroups:
    - config.istio.io
    resources:
    - '*'
    verbs:
    - '*'
  - apiGroups:
    - iam.kubesphere.io
    resources:
    - '*'
    verbs:
    - '*'
  - apiGroups:
    - notification.kubesphere.io
    resources:
    - '*'
    verbs:
    - '*'
  - apiGroups:
    - auditing.kubesphere.io
    resources:
    - '*'
    verbs:
    - '*'
  - apiGroups:
    - events.kubesphere.io
    resources:
    - '*'
    verbs:
    - '*'
  - apiGroups:
    - core.kubefed.io
    resources:
    - '*'
    verbs:
    - '*'
  - apiGroups:
    - installer.kubesphere.io
    resources:
    - '*'
    verbs:
    - '*'
  - apiGroups:
    - storage.kubesphere.io
    resources:
    - '*'
    verbs:
    - '*'
  - apiGroups:
    - security.istio.io
    resources:
    - '*'
    verbs:
    - '*'
  - apiGroups:
    - monitoring.kiali.io
    resources:
    - '*'
    verbs:
    - '*'
  - apiGroups:
    - networking.k8s.io
    resources:
    - '*'
    verbs:
    - '*'
  ---
  kind: ClusterRoleBinding
  apiVersion: rbac.authorization.k8s.io/v1
  metadata:
    name: ks-installer
  subjects:
  - kind: ServiceAccount
    name: ks-installer
    namespace: kubesphere-system
  roleRef:
    kind: ClusterRole
    name: ks-installer
    apiGroup: rbac.authorization.k8s.io
  
  ---
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: ks-installer
    namespace: kubesphere-system
    labels:
      app: ks-install
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: ks-install
    template:
      metadata:
        labels:
          app: ks-install
      spec:
        serviceAccountName: ks-installer
        containers:
        - name: installer
          image: kubespheredev/ks-installer:latest
          imagePullPolicy: "Always"
          resources:
            limits:
              cpu: "1"
              memory: 1Gi
            requests:
              cpu: 20m
              memory: 100Mi
          volumeMounts:
          - mountPath: /etc/localtime
            name: host-time
        volumes:
        - hostPath:
            path: /etc/localtime
            type: ""
          name: host-time
  ```

- **cluster-configuration.yaml**

  ```
  ---
  apiVersion: installer.kubesphere.io/v1alpha1
  kind: ClusterConfiguration
  metadata:
    name: ks-installer
    namespace: kubesphere-system
    labels:
      version: v3.0.0
  spec:
    persistence:
      storageClass: ""        # If there is not a default StorageClass in your cluster, you need to specify an existing StorageClass here.
    authentication:
      jwtSecret: ""           # Keep the jwtSecret consistent with the host cluster. Retrive the jwtSecret by executing "kubectl -n kubesphere-system get cm kubesphere-config -o yaml | grep -v "apiVersion" | grep jwtSecret" on the host cluster.
    etcd:
      monitoring: false       # Whether to enable etcd monitoring dashboard installation. You have to create a secret for etcd before you enable it.
      endpointIps: localhost  # etcd cluster EndpointIps, it can be a bunch of IPs here.
      port: 2379              # etcd port
      tlsEnable: true
    common:
      mysqlVolumeSize: 20Gi # MySQL PVC size.
      minioVolumeSize: 20Gi # Minio PVC size.
      etcdVolumeSize: 20Gi  # etcd PVC size.
      openldapVolumeSize: 2Gi   # openldap PVC size.
      redisVolumSize: 2Gi # Redis PVC size.
      monitoring:
        endpoint: http://prometheus-operated.kubesphere-monitoring-system.svc:9090 # Prometheus endpoint to get metrics data
      es:   # Storage backend for logging, events and auditing.
        # elasticsearchMasterReplicas: 1   # total number of master nodes, it's not allowed to use even number
        # elasticsearchDataReplicas: 1     # total number of data nodes.
        elasticsearchMasterVolumeSize: 4Gi   # Volume size of Elasticsearch master nodes.
        elasticsearchDataVolumeSize: 20Gi    # Volume size of Elasticsearch data nodes.
        logMaxAge: 7                     # Log retention time in built-in Elasticsearch, it is 7 days by default.
        elkPrefix: logstash              # The string making up index names. The index name will be formatted as ks-<elk_prefix>-log.
    console:
      enableMultiLogin: true  # enable/disable multiple sing on, it allows an account can be used by different users at the same time.
      port: 30880
    alerting:                # (CPU: 0.3 Core, Memory: 300 MiB) Whether to install KubeSphere alerting system. It enables Users to customize alerting policies to send messages to receivers in time with different time intervals and alerting levels to choose from.
      enabled: false
    auditing:                # Whether to install KubeSphere audit log system. It provides a security-relevant chronological set of records，recording the sequence of activities happened in platform, initiated by different tenants.
      enabled: false
    devops:                  # (CPU: 0.47 Core, Memory: 8.6 G) Whether to install KubeSphere DevOps System. It provides out-of-box CI/CD system based on Jenkins, and automated workflow tools including Source-to-Image & Binary-to-Image.
      enabled: false
      jenkinsMemoryLim: 2Gi      # Jenkins memory limit.
      jenkinsMemoryReq: 1500Mi   # Jenkins memory request.
      jenkinsVolumeSize: 8Gi     # Jenkins volume size.
      jenkinsJavaOpts_Xms: 512m  # The following three fields are JVM parameters.
      jenkinsJavaOpts_Xmx: 512m
      jenkinsJavaOpts_MaxRAM: 2g
    events:                  # Whether to install KubeSphere events system. It provides a graphical web console for Kubernetes Events exporting, filtering and alerting in multi-tenant Kubernetes clusters.
      enabled: false
      ruler:
        enabled: true
        replicas: 2
    logging:                 # (CPU: 57 m, Memory: 2.76 G) Whether to install KubeSphere logging system. Flexible logging functions are provided for log query, collection and management in a unified console. Additional log collectors can be added, such as Elasticsearch, Kafka and Fluentd.
      enabled: false
      logsidecarReplicas: 2
    metrics_server:                    # (CPU: 56 m, Memory: 44.35 MiB) Whether to install metrics-server. IT enables HPA (Horizontal Pod Autoscaler).
      enabled: false
    monitoring:
      # prometheusReplicas: 1            # Prometheus replicas are responsible for monitoring different segments of data source and provide high availability as well.
      prometheusMemoryRequest: 400Mi   # Prometheus request memory.
      prometheusVolumeSize: 20Gi       # Prometheus PVC size.
      # alertmanagerReplicas: 1          # AlertManager Replicas.
    multicluster:
      clusterRole: none  # host | member | none  # You can install a solo cluster, or specify it as the role of host or member cluster.
    networkpolicy:       # Network policies allow network isolation within the same cluster, which means firewalls can be set up between certain instances (Pods).
      # Make sure that the CNI network plugin used by the cluster supports NetworkPolicy. There are a number of CNI network plugins that support NetworkPolicy, including Calico, Cilium, Kube-router, Romana and Weave Net.
      enabled: false
    notification:        # Email Notification support for the legacy alerting system, should be enabled/disabled together with the above alerting option.
      enabled: false
    openpitrix:          # (2 Core, 3.6 G) Whether to install KubeSphere Application Store. It provides an application store for Helm-based applications, and offer application lifecycle management.
      enabled: false
    servicemesh:         # (0.3 Core, 300 MiB) Whether to install KubeSphere Service Mesh (Istio-based). It provides fine-grained traffic management, observability and tracing, and offer visualization for traffic topology.
      enabled: false
  ```

- **部署**

  ```
  kubectl apply -f kubesphere-installer.yaml
  kubectl apply -f cluster-configuration.yaml
  ```

  该过程需要大量时间。

- **验证与访问**

  1. 查看滚动刷新的安装日志，请耐心等待安装成功。

  ```bash
  $ kubectl logs -n kubesphere-system $(kubectl get pod -n kubesphere-system -l app=ks-install -o jsonpath='{.items[0].metadata.name}') -f
  ```

  > 说明：安装过程中若遇到问题，也可以通过以上日志命令来排查问题。

  2. 通过 `kubectl get pod --all-namespaces`查看 KubeSphere 相关 namespace 下所有 Pod 状态是否为 Running。
  3. 使用 `IP:30880`访问 KubeSphere UI 界面，默认的集群管理员账号为 `admin/P@88w0rd`。

- **如何重启安装**

若安装过程中遇到问题，当您解决问题后，可以通过重启 ks-installer 的 Pod 来重启安装任务，将 ks-installer 的 Pod 删除即可让其自动重启：

```text
$ kubectl delete pod ks-installer-xxxxxx-xxxxx -n kubesphere-system
```

- **开启 etcd 监控**

完整安装成功后，KubeSphere 还支持在控制台开启 etcd 的监控面板，请参考 [ks-installer GitHub](https://github.com/kubesphere/ks-installer/tree/master) 创建 etcd 所需要的。

- **成功效果**

![](E:\程序人生\乐购商城数据\图片\集群.png)

#### 5.4 [安装可插拔功能组件](https://v2-1.docs.kubesphere.io/docs/zh-CN/installation/pluggable-components/)

- **KubeSphere DevOps 系统**

  KubeSphere 针对容器与 Kubernetes 的应用场景，基于 Jenkins 提供了一站式 DevOps 系统，包括丰富的 CI/CD 流水线构建与插件管理功能，还提供 Binary-to-Image（B2I）、Source-to-Image（S2I），为流水线、S2I、B2I 提供代码依赖缓存支持，以及代码质量管理与流水线日志等功能。

  内置的 DevOps 系统将应用的开发和自动发布与容器平台进行了很好的结合，还支持对接第三方的私有镜像仓库和代码仓库形成完善的私有场景下的 CI/CD，提供了端到端的用户体验。

  可参考如下文档进一步了解 KubeSphere DevOps 系统的功能：

> - [Binary-to-Image](https://v2-1.docs.kubesphere.io/docs/zh-CN/quick-start/b2i-war)：将 WAR、JAR、Binary 这一类的制品快速打包成 Docker 镜像，并发布到镜像仓库中，最终将服务自动发布至 Kubernetes；
> - [Source-to-Image](https://v2-1.docs.kubesphere.io/docs/zh-CN/quick-start/source-to-image)：无需写 Dockerfile，仅输入源代码地址即可自动打包成可运行程序到 Docker 镜像的工具，方便构建镜像发布至镜像仓库和 Kubernetes；
> - [图形化构建流水线](https://v2-1.docs.kubesphere.io/docs/zh-CN/quick-start/jenkinsfile-out-of-scm)：通过图形化编辑的界面构建流水线，无需写 Jenkinsfile，交互更友好；
> - [基于 Jenkinsfile 构建流水线](https://v2-1.docs.kubesphere.io/docs/zh-CN/quick-start/devops-online)：基于项目仓库中已有的 Jenkinsfile 快速构建流水线；
> - [基于 GitLab + Harbor 构建流水线](https://v2-1.docs.kubesphere.io/docs/zh-CN/harbor-gitlab-devops-offline)：支持对接第三方的镜像仓库和代码仓库；

**安装前如何开启安装 DevOps 系统**

注意：开启可选功能组件之前，请先参考 [可插拔功能组件列表](https://v2-1.docs.kubesphere.io/docs/zh-CN/installation/intro/#可插拔功能组件列表)，确认集群的可用 CPU 与内存空间是否充足，开启安装前可能需要提前扩容集群或机器配置，否则可能会因为资源不足而导致的机器崩溃或其它问题。

> 注意，本篇文档仅适用于 Linux Installer 安装的环境，若 KubeSphere 部署在 Kubernetes 之上，需提前创建 CA 证书，请参考 [ks-installer](https://github.com/kubesphere/ks-installer)。

安装前，在 installer 目录下编辑 `conf/common.yaml`文件，然后参考如下开启。

> 提示：KubeSphere 支持对接外置的 SonarQube，若您已有外置的 SonarQube，建议在以下参数中配置对接，可减少 KubeSphere 集群的资源消耗。

```yaml
#DevOps Configuration
devops_enabled: true # 是否安装内置的 DevOps 系统（支持流水线、 S2i 和 B2i 等功能），若机器配置充裕建议安装
jenkins_memory_lim: 8Gi # Jenkins 内存限制，默认 8 Gi
jenkins_memory_req: 4Gi # Jenkins 内存请求，默认 4 Gi
jenkins_volume_size: 8Gi # Jenkins 存储卷大小，默认 8 Gi
jenkinsJavaOpts_Xms: 3g # 以下三项为 jvm 启动参数
jenkinsJavaOpts_Xmx: 6g
jenkinsJavaOpts_MaxRAM: 8g
sonarqube_enabled: true # 是否安装内置的 SonarQube （代码静态分析工具）
#sonar_server_url: SHOULD_BE_REPLACED # 安装支持对接外部已有的 SonarQube，此处填写 SonarQube 服务的地址
#sonar_server_token: SHOULD_BE_REPLACED  # 此处填写 SonarQube 的 Token
```

**安装后如何开启安装 DevOps 系统**

通过修改 ks-installer 的 configmap 可以选装组件，执行以下命令（kubectl 命令需要以 root 用户执行）。

```bash
$ kubectl edit cc -n kubesphere-system ks-installer
```

**参考如下修改 ConfigMap**

```yaml
devops:
      enabled: True
      jenkinsMemoryLim: 2Gi
      jenkinsMemoryReq: 1500Mi
      jenkinsVolumeSize: 8Gi
      jenkinsJavaOpts_Xms: 512m
      jenkinsJavaOpts_Xmx: 512m
      jenkinsJavaOpts_MaxRAM: 2g
      sonarqube:
        enabled: True
```

> 按功能需求编辑配置文件之后，退出等待生效即可，如长时间未生效请使用如下命令查看相关日志:

```
kubectl logs -n kubesphere-system $(kubectl get pod -n kubesphere-system -l app=ks-install -o jsonpath='{.items[0].metadata.name}') -f
```

### 6. KubeSphere快速入门

#### 6.1 多租户管理快速入门

##### 6.1.1 概念

目前，平台的资源一共有三个层级，包括 **集群 (Cluster)、 企业空间 (Workspace)、 项目 (Project) 和 DevOps Project (Dev 工程)**，层级关系如下图所示，即一个集群中可以创建多个企业空间，而每个企业空间，可以创建多个项目和 DevOps工程，而集群、企业空间、项目和 DevOps工程中，默认有多个不同的内置角色。

![img](https://pek3b.qingstor.com/kubesphere-docs/png/20191026004813.png)

##### 6.1.2 基本使用

**第一步：创建角色和账号**

平台中的 cluster-admin 角色可以为其他用户创建账号并分配平台角色，平台内置了集群层级的以下三个常用的角色，同时支持自定义新的角色。

| 内置角色           | 描述                                                         |
| :----------------- | :----------------------------------------------------------- |
| cluster-admin      | 集群管理员，可以管理集群中所有的资源。                       |
| workspaces-manager | 集群中企业空间管理员，仅可创建、删除企业空间，维护企业空间中的成员列表。 |
| cluster-regular    | 集群中的普通用户，在被邀请加入企业空间之前没有任何资源操作权限。 |

本示例首先新建一个角色 (users-manager)，为该角色授予账号管理和角色管理的权限，然后新建一个账号并给这个账号授予 users-manager 角色。

| 账号名       | 集群角色      | 职责                 |
| :----------- | :------------ | :------------------- |
| user-manager | users-manager | 管理集群的账户和角色 |

通过下图您可以更清楚地了解本示例的逻辑： ![img](https://hello-world-20181219.pek3b.qingstor.com/KS2.1/%E5%A4%9A%E7%A7%9F%E6%88%B7%E7%AE%A1%E7%90%86%E5%BF%AB%E9%80%9F%E5%85%A5%E9%97%A8.png)

**第二步：创建企业空间**

企业空间 (workspace) 是 KubeSphere 实现多租户模式的基础，是用户管理项目、DevOps 工程和企业成员的基本单位。

**第三步：创建项目**

创建工作负载、服务和 CI/CD 流水线等资源之前，需要预先创建项目和 DevOps 工程。

**设置外网访问**

在创建应用路由之前，需要先启用外网访问入口，即网关。这一步是创建对应的应用路由控制器，负责接收项目外部进入的流量，并将请求转发到对应的后端服务。

选择 「项目设置」 → 「外网访问」，点击 「设置网关」。

![img](https://pek3b.qingstor.com/kubesphere-docs/png/20191027154926.png)

在弹窗中，默认 NodePort 即可，然后点击 「保存」。

![img](https://pek3b.qingstor.com/kubesphere-docs/png/20190507222321.png)

> 提示：若希望开启 LoadBalancer 类型暴露集群内部服务，则需要安装云厂商支持 LoadBalancer 插件，例如 [QingCloud 负载均衡器插件](https://v2-1.docs.kubesphere.io/docs/zh-CN/installation/qingcloud-lb)；若 KubeSphere 部署在阿里云或其它云平台，则需要手动安装其支持的 LB 插件；若 KubeSphere 部署在物理机环境，可安装配置 [Porter - 面向物理环境的 K8s LB 插件](https://github.com/kubesphere/porter)。

当前可以看到网关地址、http/https 端口号都已经开启。

![img](https://pek3b.qingstor.com/kubesphere-docs/png/20191027155527.png)

**第四步：创建 DevOps 工程（可选）**

> 说明：DevOps 功能组件作为可选安装项，提供 CI/CD 流水线、B2i/S2i 等丰富的企业级功能，若还未安装可参考 [自定义组件安装（安装后）](https://v2-1.docs.kubesphere.io/docs/zh-CN/installation/components) 开启 DevOps 组件安装，然后参考以下步骤。

点击 **工作台**，在当前企业空间下，点击 **创建**，在弹窗中选择 **创建一个 DevOps 工程**

![创建 DevOps 工程](https://pek3b.qingstor.com/kubesphere-docs/png/20190428131518.png)

输入 DevOps 工程的名称和描述信息。点击 **创建**，注意创建一个 DevOps 有一个初始化环境的过程需要几秒钟。

> 说明：DevOps 工程管理的详细说明请参考 [管理 DevOps 工程](https://v2-1.docs.kubesphere.io/docs/zh-CN/devops/devops-project)。

点击 DevOps 工程列表进入该工程的详情页。

![img](https://pek3b.qingstor.com/kubesphere-docs/png/20191027160905.png)

#### 6.2 创建 Wordpress 应用并发布至 Kubernetes

##### 6.2.1 WordPress 简介

WordPress 是使用 PHP 开发的博客平台，用户可以在支持 PHP 和 MySQL 数据库的环境中架设属于自己的网站。本文以创建一个 [Wordpress 应用](https://v2-1.docs.kubesphere.io/docs/zh-CN/quick-start/wordpress-deployment/www.wordpress.com/‎) 为例，以创建 KubeSphere 应用的形式将 Wordpress 的组件（MySQL 和 Wordpress）创建后发布至 Kubernetes 中，并在集群外访问 Wordpress 服务。

一个完整的 Wordpress 应用会包括以下 Kubernetes 对象，其中 MySQL 作为后端数据库，Wordpress 本身作为前端提供浏览器访问。

![img](https://pek3b.qingstor.com/kubesphere-docs/png/20191027200444.png)

### 7. DevOps（开发+维护）

#### 基于Spring Boot项目构建流水线

Jenkinsfile in SCM 意为将 Jenkinsfile 文件本身作为源代码管理 (Source Control Management) 的一部分，根据该文件内的流水线配置信息快速构建工程内的 CI/CD 功能模块，比如阶段 (Stage)，步骤 (Step) 和任务 (Job)。因此，在代码仓库中应包含 Jenkinsfile。

#### 7.1 前提条件

- 开启安装了 DevOps 功能组件，参考 [安装 DevOps 系统](https://v2-1.docs.kubesphere.io/docs/zh-CN/installation/install-devops)；
- 本示例以 GitHub 和 DockerHub 为例，参考前确保已创建了 [GitHub](https://github.com/) 和 [DockerHub](http://www.dockerhub.com/) 账号；
- 已创建了企业空间和 DevOps 工程并且创建了项目普通用户 project-regular 的账号，若还未创建请参考 [多租户管理快速入门](https://v2-1.docs.kubesphere.io/docs/zh-CN/quick-start/admin-quick-start)；
- 使用项目管理员 `project-admin`邀请项目普通用户 `project-regular`加入 DevOps 工程并授予 `maintainer`角色，若还未邀请请参考 [多租户管理快速入门 - 邀请成员](https://v2-1.docs.kubesphere.io/docs/zh-CN/quick-start/admin-quick-start/#邀请成员)。
- 参考 [配置 ci 节点](https://v2-1.docs.kubesphere.io/docs/zh-CN/system-settings/edit-system-settings/#如何配置-ci-节点进行构建) 为流水线选择执行构建的节点。

#### 7.2 流水线概览

下面的流程图简单说明了流水线完整的工作过程：

![img](https://pek3b.qingstor.com/kubesphere-docs/png/20190512155453.png)

> 流程说明：
>
> - **阶段一. Checkout SCM**: 拉取 GitHub 仓库代码
> - **阶段二. Unit test**: 单元测试，如果测试通过了才继续下面的任务
> - **阶段三. SonarQube analysis**：sonarQube 代码质量检测
> - **阶段四. Build & push snapshot image**: 根据行为策略中所选择分支来构建镜像，并将 tag 为 `SNAPSHOT-$BRANCH_NAME-$BUILD_NUMBER`推送至 Harbor (其中 `$BUILD_NUMBER`为 pipeline 活动列表的运行序号)。
> - **阶段五. Push latest image**: 将 master 分支打上 tag 为 latest，并推送至 DockerHub。
> - **阶段六. Deploy to dev**: 将 master 分支部署到 Dev 环境，此阶段需要审核。
> - **阶段七. Push with tag**: 生成 tag 并 release 到 GitHub，并推送到 DockerHub。
> - **阶段八. Deploy to production**: 将发布的 tag 部署到 Production 环境。

#### 7.3 [操作示例](https://v2-1.docs.kubesphere.io/docs/zh-CN/quick-start/devops-online/)

### 8. CI/CD/CD

![](E:\程序人生\个人学习笔记\学习笔记图床\QQ图片20201201105136.png)



**1、持续集成(Continuous Integration)**
持续集成是指软件个人研发的部分向软件整体部分交付，频繁进行集成以便更快地发其中的错误。“持续集成”源自于极限编程(XP)，是XP最初的12种实践之一。
	CI 需要具备：

- 全面的自动化测试。这是实践持续集成&持续部署的基础，同时，选择合适自动化测试工具也极其重要；
- 灵活的基础设施。容器，虚拟机的存在让开发人员和QA 人员不必再大费折；
- 版本控制工具。如Git， CVS， SVN等；
- 自动化的构建和软件发布流程的工具[如 Jenkins, flow.ci
- 反馈机制。如构建/测试的失败，可以快速地反馈到相关负责人，以尽快解达到一个更稳定的版本。

**2、持续交付(Continuous Delivery)**
持续交付在持续集成的基础上，将集成后的代码部署到更贴近真实运行环境的「类生产环境」(production-like environments)中。持续交付优先于整个产品生命周期的软件部署，建立在高水平自动化持续集成之上。
灰度发布。
持续交付和持续集成的优点非常相似：

- 快速发布。能够应对业务需求，并更快地实现软件价值。
- 编码->测试->上线->交付的频繁迭代周期缩短，同时获得迅速反馈；
- 高质量的软件发布标准。整个交付过程标准化、可重复、可靠，
- 整个交付过程进度可视化，方便团队人员了解项目成熟度；
- 更先进的团队协作方式。从需求分析、产品的用户体验到交互设计、开发、测试、运维等角色密切协作，相比于传统的瀑布式软件团队，更少浪费。

**3、持续部署(Continuous Deployment)**
持续部署是指当交付的代码通过评审之后，自动部署到生产环境中。持续部署是持续交付的最高阶段。这意味着，所有通过了一系列的自动化测试的改动都将自动部署到生产环境。它也可以被称为“Continuous Release"。

```
开发人员提交代码，持续集成服务器获取代码，执行单元测试，根据测
试结果决定是否部署到预演环境，如果成功部署到预演环境，进行整体
验收测试，如果测试通过，自动部署到产品环境，全程自动化高效运转。
```

持续部署主要好处是，可以相对独立地部署新的功能，并能快速地收集真实用户的反馈。