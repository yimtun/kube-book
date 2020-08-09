



# 192.168.3.101



当前环境下 192.168.3.101 作为master 节点



创建虚拟机 

```
/root/create_vm.sh
```



sys init



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



kernel 



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





install docker



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



install  kubernetes

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







completion



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





```
kubectl  desc
```





```
poweroff
```





在kvm宿主机上做如下操作



```
cp /kvm/image/192.168.3.101.qcow2 /kvm/image/192.168.3.101.qcow2-bak
```




# 192.168.3.104





当前环境下 192.168.3.104 作为node 节点 

此环境为手动添加节点而非使用kubeadm 加入节点

所以仅仅需要安装kubelet 和 cni



创建虚拟机 

```
/root/create_vm.sh
```



sys init



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



kernel 



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





install docker



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



install  kubernetes

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
yum -y install  /opt/rpms/*
```





```
poweroff
```











在kvm宿主机上做如下操作



```
cp /kvm/image/192.168.3.104.qcow2 /kvm/image/192.168.3.104.qcow2-bak
```




# 192.168.3.103



当前环境下 192.168.3.103 作为node 节点 由kubeadm 添加到集群



创建虚拟机 

```
/root/create_vm.sh
```



sys init



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



kernel 



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





install docker



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



install  kubernetes

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
poweroff
```



在kvm宿主机上做如下操作

```
cp /kvm/image/192.168.3.103.qcow2 /kvm/image/192.168.3.103.qcow2-bak
```


