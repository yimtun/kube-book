

> https://v1-18.docs.kubernetes.io/docs/reference/command-line-tools-reference/kubelet-tls-bootstrapping/#bootstrap-tokens





# 1 事先生成bootstrapper token的方法



方法1

```
openssl rand -hex 3
usvieq
```



```
openssl rand -hex 8
2zg86228emymgcd4
```



方法2    在一个装有 kubeadm 的机器上生成

```
kubeadm token generate
```





下面配置文件里的 token  即为   usvieq.2zg86228emymgcd4





# 2 指定 bootstrapper token 的kubeadm  配置文件



相比外置etcd 的配置文件 添加了如下部分   在本节文档的3 中  仅仅指定了 token 的值  目的让 kubeadm 配置token的其他参数为默认参数

```
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: usvieq.2zg86228emymgcd4
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
```







```
cat > /opt/kube-1.18.3-config.yaml << EOF
apiVersion: kubeadm.k8s.io/v1beta2
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 0.0.0.0
  bindPort: 6443
nodeRegistration:
  name: 192.168.3.101
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: usvieq.2zg86228emymgcd4
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
---
apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
kubernetesVersion: v1.18.3
imageRepository: registry.aliyuncs.com/google_containers
controlPlaneEndpoint: 192.168.3.101:6443
controllerManager: {}
scheduler: {}
etcd:
    external:
        endpoints:
        - https://192.168.3.102:2379
        caFile: /etc/etcd/certs/etcd-ca.crt
        certFile: /etc/etcd/certs/etcd-client.crt
        keyFile: /etc/etcd/certs/etcd-client.key
networking:
  serviceSubnet: 10.96.0.0/12
  podSubnet: 10.244.0.0/16
---
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
mode: ipvs
EOF
```









-----------------





# 3 指定 bootstrapTokens部署k8s

清理环境



```
virsh  undefine 192.168.3.101
virsh  destroy  192.168.3.101
rm -rf /kvm/image/192.168.3.101.qcow2
systemctl restart libvirtd
```



使用 外置etcd

使用自己创建的证书

 

192.168.3.101



###  创建虚拟机



```
/root/create_vm.sh
```



```
/root/create_vm.sh 
前置环境检查
存储池 kvmimage 中存在vol 模板文件 CentOS7.6.1810.x86_64.qcow2 检查通过继续运行
请输入新建虚拟机名称: 192.168.3.101
Vol 192.168.3.101.qcow2 cloned from CentOS7.6.1810.x86_64.qcow2

请输入新建虚拟机磁盘大小单位G: 40
开始创建虚拟磁盘
Size of volume '192.168.3.101.qcow2' successfully changed to 40G

请输入新建虚拟机内存大小单位M   : 4096
请输入新建虚拟机cpu核心数 : 2
192.168.3.101 deinfe 成功 开始设置ip
请输入为虚拟机配置的ip:  192.168.3.101
请输入为虚拟机配置的掩码: 255.255.255.0
请输入为虚拟机配置的网关: 192.168.3.1
NAME=eth0
DEVICE=eth0
ONBOOT=yes
TYPE=Ethernet
BOOTPROTO=none
IPADDR=192.168.3.101
NETMASK=255.255.255.0
GATEWAY=192.168.3.1
DNS1=223.5.5.5
请确认您的网卡配置，按y键继续?[y/n]: y
开始配置192.168.3.101 的网卡，请稍后.......
正在启动虚拟机请稍后.....
虚拟机启动成功，登录赋值如下命令即可  ssh 192.168.3.101
```





---------------------------



###  系统初始化

关闭selinux

```
sed -i 's/enforcing/disabled/g'  /etc/selinux/config
setenforce 0
sed -i 's/#UseDNS yes/UseDNS no/g'   /etc/ssh/sshd_config
systemctl   restart sshd
grep DNS               /etc/ssh/sshd_config
grep SELINUX=disabled  /etc/selinux/config 
systemctl  disable firewalld  NetworkManager
systemctl  stop    firewalld    NetworkManager
mv /etc/yum.repos.d/CentOS-Base.repo      /etc/yum.repos.d/CentOS-Base.repo.backup
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
curl  -o /etc/yum.repos.d/epel.repo       http://mirrors.aliyun.com/repo/epel-7.repo
```



时间相关

```
timedatectl set-timezone Asia/Shanghai
yum -y install ntpdate
ntpdate ntp1.aliyun.com
timedatectl set-local-rtc 0
systemctl restart rsyslog
systemctl restart crond
yum -y install ntp
```



```
cat >   /etc/ntp.conf  << EOF
driftfile /var/lib/ntp/drift
restrict default nomodify notrap nopeer noquery
restrict 127.0.0.1 
restrict ::1
server ntp1.aliyun.com iburst  iburst
logfile /var/log/ntp.log
includefile /etc/ntp/crypto/pw
keys /etc/ntp/keys
disable monitor
EOF
```



```
systemctl  enable  --now ntpd
ntpq -p
```


###  配置必要的内核参数

```
cat <<EOF | tee /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
```



```
modprobe br_netfilter
sysctl -p /etc/sysctl.d/k8s.conf
```





###  docker 安装与配置

```
yum -y install yum-utils
```



```
yum-config-manager --add-repo \
https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```



```
mkdir  /etc/docker
```



```
cat > /etc/docker/daemon.json  << EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "registry-mirrors": ["https://2xdz2l32.mirror.aliyuncs.com"]
}
EOF
```





```
yum -y install docker-ce-18.06.1.ce-3.el7
systemctl  enable --now docker
```





###  k8s 安装基础组件




```
cat > /etc/yum.repos.d/kubernetes.repo <<  EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=0
EOF
```





```
mkdir /opt/rpms/
```



```
yumdownloader --destdir  /opt/rpms/   kubernetes-cni-0.7.5-0
```



```
yumdownloader --destdir  /opt/rpms/    kubelet-1.18.3
```



```
yumdownloader --destdir  /opt/rpms/    cri-tools-1.13.0
```



```
yumdownloader --destdir  /opt/rpms/    kubectl-1.18.3
```



```
yumdownloader --destdir  /opt/rpms/    kubeadm-1.18.3
```



```
yum -y install  /opt/rpms/*
```



```
systemctl enable kubelet.service
```





```
kubectl  version
```

```
kubeadm  version
```

```
kubelet  --version
```



```
crictl   --version
```



###  kubeclt 命令补全



```
yum   -y install bash-completion
```



```
source /usr/share/bash-completion/bash_completion
```



```
echo 'source <(kubectl completion bash)'     >>  /root/.bashrc
```



```
source  /root/.bashrc
```



测试  复制下面内容到终端按 tab 测试是否自动补全

```
kubectl  desc
```









### 配置集群参数文件



```
cat > /opt/kube-1.18.3-config.yaml << EOF
apiVersion: kubeadm.k8s.io/v1beta2
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 0.0.0.0
  bindPort: 6443
nodeRegistration:
  name: 192.168.3.101
bootstrapTokens:
  - token: usvieq.2zg86228emymgcd4
---
apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
kubernetesVersion: v1.18.3
imageRepository: registry.aliyuncs.com/google_containers
controlPlaneEndpoint: 192.168.3.101:6443
controllerManager: {}
scheduler: {}
etcd:
    external:
        endpoints:
        - https://192.168.3.102:2379
        caFile: /etc/etcd/certs/etcd-ca.crt
        certFile: /etc/etcd/certs/etcd-client.crt
        keyFile: /etc/etcd/certs/etcd-client.key
networking:
  serviceSubnet: 10.96.0.0/12
  podSubnet: 10.244.0.0/16
---
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
mode: ipvs
EOF
```



###  设置etcd 证书



拷贝etcd客户端证书 

```
mkdir -p /etc/etcd/certs/
```



```
scp  192.168.3.102:/etcd-certs/{etcd-ca.crt,etcd-client.crt,etcd-client.key}    /etc/etcd/certs/
```



拷贝etcd客户端工具 etcdctl  方便调试

```
scp  192.168.3.102:/usr/local/bin/etcdctl    /usr/local/bin/
```



在 api server 上使用命令行客户端 工具 etcdctl  进行连接测试

```
ETCDCTL_API=3 etcdctl \
--endpoints=https://192.168.3.102:2379 \
--cacert=/etc/etcd/certs/etcd-ca.crt \
--cert=/etc/etcd/certs/etcd-client.crt \
--key=/etc/etcd/certs/etcd-client.key endpoint health
```



```
https://192.168.3.102:2379 is healthy: successfully committed proposal: took = 41.89946ms
```





查看etcd 数据



```
ETCDCTL_API=3 etcdctl \
--endpoints=https://192.168.3.102:2379 \
--cacert=/etc/etcd/certs/etcd-ca.crt \
--cert=/etc/etcd/certs/etcd-client.crt \
--key=/etc/etcd/certs/etcd-client.key  \
get / --prefix --keys-only
```





删除所有数据   重新装的时候  etcd  不必重新安装  清除数据即可

安装前如果查看当前etcd 有数据使用下面命令删除

```
ETCDCTL_API=3 etcdctl \
--endpoints=https://192.168.3.102:2379 \
--cacert=/etc/etcd/certs/etcd-ca.crt \
--cert=/etc/etcd/certs/etcd-client.crt \
--key=/etc/etcd/certs/etcd-client.key  \
del / --prefix
```





###  部署证书



上传  my-certs.zip 到虚拟机 /root/   下



```
yum -y install unzip
```



```
unzip  /root/my-certs.zip   -d /
```



```
mkdir -p /etc/kubernetes/pki/
```



```
cp /my-certs/{apiserver.crt,apiserver.key,apiserver-kubelet-client.crt,apiserver-kubelet-client.key}    /etc/kubernetes/pki/
cp /my-certs/{ca.crt,ca.key,front-proxy-ca.crt,front-proxy-ca.key}  /etc/kubernetes/pki/
cp /my-certs/{front-proxy-client.crt,front-proxy-client.key,sa.key,sa.pub} /etc/kubernetes/pki/
```



-------------

```
ls /etc/kubernetes/pki/
```



```
apiserver.crt  apiserver-kubelet-client.crt  ca.crt  front-proxy-ca.crt  front-proxy-client.crt  sa.key
apiserver.key  apiserver-kubelet-client.key  ca.key  front-proxy-ca.key  front-proxy-client.key  sa.pub
```



###  kubeadm 安装k8s



```
kubeadm config images  list --config  /opt/kube-1.18.3-config.yaml
```



```
kubeadm config images  pull  --config  /opt/kube-1.18.3-config.yaml
```



------------

```
kubeadm  init --config  /opt/kube-1.18.3-config.yaml
```





```
W0628 13:37:19.528661   13283 configset.go:202] WARNING: kubeadm cannot validate component configs for API groups [kubelet.config.k8s.io kubeproxy.config.k8s.io]
[init] Using Kubernetes version: v1.18.3
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Using existing ca certificate authority
[certs] Using existing apiserver certificate and key on disk
[certs] Using existing apiserver-kubelet-client certificate and key on disk
[certs] Using existing front-proxy-ca certificate authority
[certs] Using existing front-proxy-client certificate and key on disk
[certs] External etcd mode: Skipping etcd/ca certificate authority generation
[certs] External etcd mode: Skipping etcd/server certificate generation
[certs] External etcd mode: Skipping etcd/peer certificate generation
[certs] External etcd mode: Skipping etcd/healthcheck-client certificate generation
[certs] External etcd mode: Skipping apiserver-etcd-client certificate generation
[certs] Using the existing "sa" key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
W0628 13:37:21.173667   13283 manifests.go:225] the default kube-apiserver authorization-mode is "Node,RBAC"; using "Node,RBAC"
[control-plane] Creating static Pod manifest for "kube-scheduler"
W0628 13:37:21.174480   13283 manifests.go:225] the default kube-apiserver authorization-mode is "Node,RBAC"; using "Node,RBAC"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 22.502296 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.18" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node 192.168.3.101 as control-plane by adding the label "node-role.kubernetes.io/master=''"
[mark-control-plane] Marking the node 192.168.3.101 as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[bootstrap-token] Using token: usvieq.2zg86228emymgcd4
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of control-plane nodes by copying certificate authorities
and service account keys on each node and then running the following as root:

  kubeadm join 192.168.3.101:6443 --token usvieq.2zg86228emymgcd4 \
    --discovery-token-ca-cert-hash sha256:f15e6eefb83447b09aed5e0d34b061ba2240426aec1649366153add3ca46a787 \
    --control-plane 

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.3.101:6443 --token usvieq.2zg86228emymgcd4 \
    --discovery-token-ca-cert-hash sha256:f15e6eefb83447b09aed5e0d34b061ba2240426aec1649366153add3ca46a787 
```



注意 --discovery-token-ca-cert-hash  输出会不同

----------------------





查看etcd 是否被写入数据

```
ETCDCTL_API=3 etcdctl \
--endpoints=https://192.168.3.102:2379 \
--cacert=/etc/etcd/certs/etcd-ca.crt \
--cert=/etc/etcd/certs/etcd-client.crt \
--key=/etc/etcd/certs/etcd-client.key  \
get / --prefix --keys-only
```

```
mkdir  /root/.kube
cp /etc/kubernetes/admin.conf  /root/.kube/config
```







```
kubectl  get node
```





```
kubectl  get pod -A
```



```
NAMESPACE     NAME                                    READY   STATUS    RESTARTS   AGE
kube-system   coredns-7ff77c879f-66ts8                0/1     Pending   0          54s
kube-system   coredns-7ff77c879f-dpv97                0/1     Pending   0          54s
kube-system   kube-apiserver-192.168.3.101            1/1     Running   0          63s
kube-system   kube-controller-manager-192.168.3.101   1/1     Running   0          63s
kube-system   kube-proxy-qn66v                        1/1     Running   1          54s
kube-system   kube-scheduler-192.168.3.101            1/1     Running   0          63s
```





网络两个选项选择一个安装即可

### 网络选项1 

### 安装  calico



```
docker pull  calico/cni:v3.14.0
```

```
docker pull  calico/node:v3.14.0
```

```
docker pull calico/pod2daemon-flexvol:v3.14.0
```

```
docker pull calico/kube-controllers:v3.14.0
```



```
yum -y install git
git clone https://github.com/yimtun/kube-config.git  /opt/kube-config
```

```
kubectl  apply  -f /opt/kube-config/calico/calico-v3.14.0/calico.yaml
```



或者直接使用官网 yaml

```
kubectl apply -f https://docs.projectcalico.org/v3.14/manifests/calico.yaml
```



```
kubectl  delete  -f /opt/kube-config/calico/calico-v3.14.0/calico.yaml
```





### 网络选项2 

### 安装flannel



```
docker pull quay.io/coreos/flannel:v0.12.0-amd64
```



> https://raw.githubusercontent.com/coreos/flannel/v0.12.0/Documentation/kube-flannel.yml



```
yum -y install git
git clone https://github.com/yimtun/kube-config.git  /opt/kube-config
```



```
kubectl  apply  -f /opt/kube-config/flannel/flannel-v0.12.0/flannel-v0.12.0-amd64.yaml
```



------------------

```
kubectl  get node
```



```
NAME            STATUS   ROLES    AGE     VERSION
192.168.3.101   Ready    master   4m31s   v1.18.3
```

-----------------





```
kubectl  get pod -A
```



```
NAMESPACE     NAME                                    READY   STATUS    RESTARTS   AGE
kube-system   coredns-7ff77c879f-66ts8                1/1     Running   0          2m19s
kube-system   coredns-7ff77c879f-dpv97                1/1     Running   0          2m19s
kube-system   kube-apiserver-192.168.3.101            1/1     Running   0          2m28s
kube-system   kube-controller-manager-192.168.3.101   1/1     Running   0          2m28s
kube-system   kube-flannel-ds-amd64-6g628             1/1     Running   0          27s
kube-system   kube-proxy-qn66v                        1/1     Running   1          2m19s
kube-system   kube-scheduler-192.168.3.101            1/1     Running   0          2m28s
```



--------------



### install metrics-server



```
yum -y install git
git clone https://github.com/yimtun/metrics-yaml.git   /opt/metrics-yaml
```





```
docker pull yimtune/metrics-server-amd64:v0.3.6
docker tag  yimtune/metrics-server-amd64:v0.3.6  k8s.gcr.io/metrics-server-amd64:v0.3.6
```



```
kubectl  create -f /opt/metrics-yaml/1.8+/
```



```
kubectl  get pod -A
```



```
NAMESPACE     NAME                                    READY   STATUS    RESTARTS   AGE
kube-system   coredns-7ff77c879f-66ts8                1/1     Running   0          3m23s
kube-system   coredns-7ff77c879f-dpv97                1/1     Running   0          3m23s
kube-system   kube-apiserver-192.168.3.101            1/1     Running   0          3m32s
kube-system   kube-controller-manager-192.168.3.101   1/1     Running   0          3m32s
kube-system   kube-flannel-ds-amd64-6g628             1/1     Running   0          91s
kube-system   kube-proxy-qn66v                        1/1     Running   1          3m23s
kube-system   kube-scheduler-192.168.3.101            1/1     Running   0          3m32s
kube-system   metrics-server-8c4f4d7c4-6bclh          0/1     Pending   0          4s
```





```
kubectl  get event -n kube-system
```

```
8s          Warning   FailedScheduling    Pod          0/1 nodes are available: 1 node(s) had taints that the pod didn't tolerate.
34s         Normal    SuccessfulCreate    ReplicaSet   Created pod: metrics-server-5df586cb87-mvxqk
34s         Normal    ScalingReplicaSet   Deployment   Scaled up replica set metrics-server-5df586cb87 to 1
```





```
kubectl taint node   192.168.3.101   node-role.kubernetes.io/master:NoSchedule-
```





```
kubectl  get event -n kube-system
```



```
2s          Normal    Scheduled           Pod          Successfully assigned kube-system/metrics-server-5df586cb87-mvxqk to 172.16.99.100
0s          Normal    Pulled              Pod          Container image "k8s.gcr.io/metrics-server-amd64:v0.3.6" already present on machine
0s          Normal    Created             Pod          Created container
```





```
kubectl  top node
```



```
NAME            CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
192.168.3.101   133m         6%     1481Mi          40%    
```





```
kubectl  top  pod -n kube-system
```



```
NAME                                    CPU(cores)   MEMORY(bytes)   
coredns-7ff77c879f-8hvrm                2m           9Mi             
coredns-7ff77c879f-bfd68                2m           10Mi            
kube-apiserver-192.168.3.101            23m          369Mi           
kube-controller-manager-192.168.3.101   7m           38Mi            
kube-flannel-ds-amd64-5lkzz             2m           8Mi             
kube-proxy-7nzdc                        1m           12Mi            
kube-scheduler-192.168.3.101            3m           15Mi            
metrics-server-8c4f4d7c4-zrvs2          1m           11Mi      
```





----------------------

###  undo

最好重新部署虚拟机  网络资源 kubeadm reset 清理的不干净



```
kubeadm  reset -f
rm -rf /etc/kubernetes
rm -rf /var/lib/kubelet
rm -rf /root/.kube
rm -rf /etc/cni/net.d/*
```



```
ETCDCTL_API=3 etcdctl \
--endpoints=https://192.168.3.102:2379 \
--cacert=/etc/etcd/certs/etcd-ca.crt \
--cert=/etc/etcd/certs/etcd-client.crt \
--key=/etc/etcd/certs/etcd-client.key  \
del / --prefix
```



```
reboot
```

