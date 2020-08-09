[TOC]



# 1 查看当前kubelet 运行参数

192.168.3.103  是使用kubeadm  方法 加入集群的一个node 节点  观察到他的kubelet 运行参数如下

```
ps -ef | grep kubelet
```



```
/usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --cgroup-driver=systemd --hostname-override=192.168.3.103 --network-plugin=cni --pod-infra-container-image=registry.aliyuncs.com/google_containers/pause:3.2
```



初次启动时，--kubeconfig参数指定的文件不必存在，--bootstrap-kubeconfig参数指定文件需要事先创建好







# 2 新建机器用于测试手动将node 加入集群





192.168.3.104



```
/root/create_vm.sh 
前置环境检查
存储池 kvmimage 中存在vol 模板文件 CentOS7.6.1810.x86_64.qcow2 检查通过继续运行
请输入新建虚拟机名称: 192.168.3.104
Vol 192.168.3.104.qcow2 cloned from CentOS7.6.1810.x86_64.qcow2

请输入新建虚拟机磁盘大小单位G: 40
开始创建虚拟磁盘
Size of volume '192.168.3.104.qcow2' successfully changed to 40G

请输入新建虚拟机内存大小单位M   : 4096
请输入新建虚拟机cpu核心数 : 2
192.168.3.104 deinfe 成功 开始设置ip
请输入为虚拟机配置的ip:  192.168.3.104
请输入为虚拟机配置的掩码: 255.255.255.0
请输入为虚拟机配置的网关: 192.168.3.1
NAME=eth0
DEVICE=eth0
ONBOOT=yes
TYPE=Ethernet
BOOTPROTO=none
IPADDR=192.168.3.104
NETMASK=255.255.255.0
GATEWAY=192.168.3.1
DNS1=223.5.5.5
请确认您的网卡配置，按y键继续?[y/n]: y
开始配置192.168.3.104 的网卡，请稍后.......
正在启动虚拟机请稍后.....
虚拟机启动成功，登录赋值如下命令即可  ssh 192.168.3.104
```



###  sys init



```
sed -i 's/enforcing/disabled/g'  /etc/selinux/config
setenforce 0
sed -i 's/#UseDNS yes/UseDN	S no/g'   /etc/ssh/sshd_config
systemctl   restart sshd
grep DNS               /etc/ssh/sshd_config
grep SELINUX=disabled  /etc/selinux/config 
systemctl  disable firewalld  NetworkManager
systemctl  stop    firewalld    NetworkManager
mv /etc/yum.repos.d/CentOS-Base.repo      /etc/yum.repos.d/CentOS-Base.repo.backup
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
curl  -o /etc/yum.repos.d/epel.repo       http://mirrors.aliyun.com/repo/epel-7.repo
```



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



###   install docker


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



```
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.2
```



###  install kubelet

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
yumdownloader --destdir  /opt/rpms/    kubernetes-cni-0.7.5-0
```

```
yumdownloader --destdir  /opt/rpms/    kubelet-1.18.3
```



```
yum -y install  /opt/rpms/*
```



```
systemctl enable kubelet
```



准备kubelet 配置文件



```
mkdir /var/lib/kubelet/
```



```
cat  > /var/lib/kubelet/config.yaml  << EOF
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    cacheTTL: 0s
    enabled: true
  x509:
    clientCAFile: /etc/kubernetes/pki/ca.crt
authorization:
  mode: Webhook
  webhook:
    cacheAuthorizedTTL: 0s
    cacheUnauthorizedTTL: 0s
clusterDNS:
- 10.96.0.10
clusterDomain: cluster.local
cpuManagerReconcilePeriod: 0s
evictionPressureTransitionPeriod: 0s
fileCheckFrequency: 0s
healthzBindAddress: 127.0.0.1
healthzPort: 10248
httpCheckFrequency: 0s
imageMinimumGCAge: 0s
kind: KubeletConfiguration
nodeStatusReportFrequency: 0s
nodeStatusUpdateFrequency: 0s
rotateCertificates: true
runtimeRequestTimeout: 0s
staticPodPath: /etc/kubernetes/manifests
streamingConnectionIdleTimeout: 0s
syncFrequency: 0s
volumeStatsAggPeriod: 0s
EOF
```



拷贝ca证书 

```
mkdir  /etc/kubernetes/pki/
```

```
scp  192.168.3.101:/etc/kubernetes/pki/ca.crt /etc/kubernetes/pki/
```







# 3 准备一个符合  bootstrap token格式的字符串



先生成一个符合格式的token

```
openssl rand -hex 3
```

```
398c7c
```

--------------

```
openssl rand -hex 8
```

```
b18a35f794625922
```

-------------

```
398c7c.b18a35f794625922
```



其实也可以使用kubeadm   本节尽量避免使用kubeadm 这个工具  

```
kubeadm token generate
```



#  4 存储 bootstrap token 的secret要求的格式





> https://v1-18.docs.kubernetes.io/docs/reference/access-authn-authz/bootstrap-tokens/



> https://github.com/kubernetes/community/blob/master/contributors/design-proposals/cluster-lifecycle/bootstrap-discovery.md#new-bootstrap-token-secrets



> https://v1-18.docs.kubernetes.io/docs/reference/access-authn-authz/bootstrap-tokens/#bootstrap-token-secret-format









bootstrap token 存储在secret 中

对secret的格式要求如下



1 secret 的名字必须是 bootstrap-token-tokenid的形式     当前环境token id 就是398c7c

2 该secret 必须在 名字为kube-system 的 命名空间中

3 secret 的 type  必须是 bootstrap.kubernetes.io/token

4 Extra groups to authenticate the token as. Must start with "system:bootstrappers:"

如果使用扩展认证组   组的名字必须以  system:bootstrappers:  开头





Kubelet 组件使用 bootstrap-token 发起的请求

用户名被识别为  system:bootstrap:token-id

用户组被识别为  system:bootstrappers











# 5 bootstrap-token  的作用



> https://v1-18.docs.kubernetes.io/docs/reference/command-line-tools-reference/kubelet-tls-bootstrapping/#bootstrap-initialization





```
引导程序初始化
在引导程序初始化过程中，会发生以下情况：

kubelet开始
kubelet认为，它并没有有一个kubeconfig文件
kubelet搜索并找到bootstrap-kubeconfig文件
kubelet读取其引导程序文件，检索API服务器的URL和有限的使用“令牌”
kubelet连接到API服务器，使用令牌进行身份验证
kubelet现在具有有限的凭据来创建和检索证书签名请求（CSR）
kubelet为自己创建一个CSR，并将signerName设置为 kubernetes.io/kube-apiserver-client-kubelet
通过以下两种方式之一批准CSR：
如果已配置，则kube-controller-manager会自动批准CSR
如果进行了配置，则外部流程（可能是一个人）会使用Kubernetes API或通过以下方式批准CSR kubectl
为kubelet创建了证书
证书已颁发给kubelet
kubelet检索证书
kubelet kubeconfig使用密钥和签名证书创建一个正确的名称
kubelet开始正常运行
可选：如果已配置，则kubelet在证书即将到期时会自动请求更新证书
根据配置，自动或手动批准和颁发续签的证书。
本文档的其余部分描述了配置TLS引导的必要步骤及其局限性。
```









#  6 创建一个用于存放bootstrap token 的secret示例



398c7c.b18a35f794625922

不指定 扩展认证组



创建方法1   使用yaml 文件创建

```
cat  >  bootstrap-token-398c7c.yaml << EOF
apiVersion: v1
kind: Secret
metadata:
  name: bootstrap-token-398c7c
  namespace: kube-system
type: bootstrap.kubernetes.io/token
stringData:
  token-id: '398c7c'
  token-secret: b18a35f794625922
  usage-bootstrap-authentication: "true"
  usage-bootstrap-signing: "true"
EOF
```



默认就是system:bootstrappers 组    用户是system:bootstrap:398c7c



```
kubectl  -n kube-system  delete secrets  bootstrap-token-398c7c
kubectl create -f  bootstrap-token-398c7c.yaml
```

```
kubectl delete secret  bootstrap-token-398c7c  -n kube-system
```



创建方法2  命令行创建



```
kubectl -n kube-system create secret generic bootstrap-token-398c7c \
--type 'bootstrap.kubernetes.io/token' \
--from-literal description="cluster bootstrap token" \
--from-literal token-id=398c7c \
--from-literal token-secret=b18a35f794625922 \
--from-literal usage-bootstrap-authentication=true \
--from-literal usage-bootstrap-signing=true
```



```
kubectl delete secret  bootstrap-token-398c7c  -n kube-system
```





#  7 创建bootstrap-kubeconfig



在  192.168.3.101 上创建



```
rm -rf /opt/bootstrap-kubelet.conf
```



```
kubectl config set-cluster kubernetes \
      --certificate-authority=/etc/kubernetes/pki/ca.crt \
      --embed-certs=true \
      --server=https://192.168.3.101:6443 \
      --kubeconfig=/opt/bootstrap-kubelet.conf
      
      
kubectl config set-credentials kubelet-bootstrap \
      --token=398c7c.b18a35f794625922 \
      --kubeconfig=/opt/bootstrap-kubelet.conf
      


kubectl config set-context kubelet-bootstrap@kubernetes \
      --cluster=kubernetes \
      --user=kubelet-bootstrap \
      --kubeconfig=/opt/bootstrap-kubelet.conf
      
      
kubectl config use-context kubelet-bootstrap@kubernetes \
--kubeconfig=/opt/bootstrap-kubelet.conf

```





# 8 使bootstrap token 具有创建csr 的权限





创建  clusterrolebinding 





> https://v1-18.docs.kubernetes.io/docs/reference/command-line-tools-reference/kubelet-tls-bootstrapping/#authorize-kubelet-to-create-csr



这个 名字为   system:node-bootstrapper  的clusterrole  具有创建csr 的权限



```
kubectl create clusterrolebinding kubelet-bootstrap --clusterrole=system:node-bootstrapper --group=system:bootstrappers
```

```
kubectl  delete clusterrolebinding kubelet-bootstrap
```



创建一个 clusterrolebinding  名字是 kubelet-bootstrap  

 clusterrole  是system:node-bootstrapper

subject是一个组  组名是 system:bootstrappers  这个组名就是 bootstrap token 被识别的组名



这个操作时间就是将代表 bootstrap token 的组 绑定到名字为 system:node-bootstrapper 的clusterrole











# 9 不为bootstrap token 赋予任何权限 测试添加node



192.168.3.101 





```
cat  >  bootstrap-token-398c7c.yaml << EOF
apiVersion: v1
kind: Secret
metadata:
  name: bootstrap-token-398c7c
  namespace: kube-system
type: bootstrap.kubernetes.io/token
stringData:
  token-id: '398c7c'
  token-secret: b18a35f794625922
  usage-bootstrap-authentication: "true"
  usage-bootstrap-signing: "true"
EOF
```

```
kubectl  -n kube-system  delete  secrets  bootstrap-token-398c7c
```



```
kubectl create -f  bootstrap-token-398c7c.yaml
```





```
kubectl delete  clusterrolebinding kubelet-bootstrap
```



如果  该节点已经加入集群 就先删除这个节点

```
kubectl drain 192.168.3.104 --ignore-daemonsets
kubectl delete node 192.168.3.104
```


```
kubectl  get csr -w
```



```
kubectl get node -w
```

--------------------



192.168.3.104   



```
scp 192.168.3.101:/opt/bootstrap-kubelet.conf   /etc/kubernetes
```

如果之前作为node 节点加入过集群 可以按如下清理环境

```
rm -rf /var/lib/kubelet/pki/*
rm -rf /etc/kubernetes/kubelet.conf
```

```
reboot
```

----------------





```
/usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --cgroup-driver=systemd --hostname-override=192.168.3.104 --network-plugin=cni --pod-infra-container-image=registry.aliyuncs.com/google_containers/pause:3.2  --kubelet-cgroups=/systemd/system.slice
```



```
E0706 11:05:52.984888   22687 reflector.go:178] k8s.io/kubernetes/pkg/kubelet/kubelet.go:526: Failed to list *v1.Node: nodes "192.168.3.104" is forbidden: User "system:anonymous" cannot list resource "nodes" in API group "" at the cluster scope
```












