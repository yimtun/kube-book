# host

192.168.3.103



# sys init



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



#  install docker


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



# install kubelet

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
systemctl enable kubelet
```



# config kubelet



```
/usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --cgroup-driver=systemd --hostname-override=192.168.3.101 --network-plugin=cni --pod-infra-container-image=registry.aliyuncs.com/google_containers/pause:3.2
```



```
mkdir  /var/lib/kubelet/
```



```
scp 192.168.3.101:/var/lib/kubelet/config.yaml  /var/lib/kubelet/
```





```
mkdir /etc/kubernetes/pki/ -p
```

```
scp 192.168.3.101:/etc/kubernetes/pki/ca.crt  /etc/kubernetes/pki/
```



```
scp -r 192.168.3.101:/etc/cni  /etc/
```





# create   bootstrap-kubelet.conf



192.168.3.101 上创建



```
kubectl  -n kube-system  delete secrets  bootstrap-token-398c7c
```







##### ok

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
  auth-extra-groups: system:bootstrappers:node01
EOF
```



```
kubectl create -f bootstrap-token-398c7c.yaml
```





######  op1    ok   默认就是system:bootstrappers  组

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
kubectl create -f  bootstrap-token-398c7c.yaml
```



```
kubectl  -n kube-system  delete secrets  bootstrap-token-398c7c
```



token ok

```
kubectl -n kube-system create secret generic bootstrap-token-398c7c \
--type 'bootstrap.kubernetes.io/token' \
--from-literal description="cluster bootstrap token" \
--from-literal token-id=398c7c \
--from-literal token-secret=b18a35f794625922 \
--from-literal usage-bootstrap-authentication=true \
--from-literal usage-bootstrap-signing=true
```





-----------------



```
rm -rf /opt/bootstrap-kubelet.conf
```



```
kubectl config set-cluster kubernetes \
      --certificate-authority=/etc/kubernetes/pki/ca.crt \
      --embed-certs=true \
      --server=https://192.168.3.101:6443 \
      --kubeconfig=/opt/bootstrap-kubelet.conf
```



```
kubectl config set-credentials tls-bootstrap-token-user \
      --token=398c7c.b18a35f794625922 \
      --kubeconfig=/opt/bootstrap-kubelet.conf
```



```
kubectl config set-context tls-bootstrap-token-user@kubernetes \
      --cluster=kubernetes \
      --user=tls-bootstrap-token-user \
      --kubeconfig=/opt/bootstrap-kubelet.conf
```



```
kubectl config use-context tls-bootstrap-token-user@kubernetes  --kubeconfig=/opt/bootstrap-kubelet.conf
```







```
kubectl create clusterrolebinding kubelet-bootstrap --clusterrole=system:node-bootstrapper --group=system:bootstrappers
```



```
kubectl delete  clusterrolebinding kubelet-bootstrap
```



### op2



```
kubectl config set-cluster kubernetes \
      --certificate-authority=/etc/kubernetes/pki/ca.crt \
      --embed-certs=true \
      --server=https://192.168.3.101:6443 \
      --kubeconfig=/opt/bootstrap-kubelet.conf
```



```
kubectl config set-credentials kubelet-bootstrap \
      --token=398c7c.b18a35f794625922 \
      --kubeconfig=/opt/bootstrap-kubelet.conf
```



```
kubectl config set-context kubelet-bootstrap@kubernetes \
      --cluster=kubernetes \
      --user=kubelet-bootstrap \
      --kubeconfig=/opt/bootstrap-kubelet.conf
```



```
kubectl config use-context kubelet-bootstrap@kubernetes  --kubeconfig=/opt/bootstrap-kubelet.conf
```





### op3



```
export BOOTSTRAP_TOKEN=$(kubeadm token create \
      --description kubelet-bootstrap-token \
      --groups system:bootstrappers:node01 \
      --kubeconfig ~/.kube/config)
```



```
echo $BOOTSTRAP_TOKEN
```

```
rm -rf /opt/bootstrap-kubelet.conf
```



```
kubectl config set-cluster kubernetes \
      --certificate-authority=/etc/kubernetes/pki/ca.crt \
      --embed-certs=true \
      --server=https://192.168.3.101:6443 \
      --kubeconfig=/opt/bootstrap-kubelet.conf
```



```
kubectl config set-credentials kubelet-bootstrap \
      --token=${BOOTSTRAP_TOKEN} \
      --kubeconfig=/opt/bootstrap-kubelet.conf
```



```
kubectl config set-context default \
      --cluster=kubernetes \
      --user=kubelet-bootstrap \
      --kubeconfig=/opt/bootstrap-kubelet.conf
```



```
kubectl config use-context default --kubeconfig=/opt/bootstrap-kubelet.conf
```









```
kubectl  delete node 192.168.3.103
kubectl  get  csr -w
```



# run kubelet

192.168.3.103



```
scp 192.168.3.101:/opt/bootstrap-kubelet.conf  /etc/kubernetes/
```

```
scp 192.168.3.101:/etc/kubernetes/pki/ca.crt  /etc/kubernetes/pki/
```



```
/usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --cgroup-driver=systemd --hostname-override=192.168.3.103 --network-plugin=cni --pod-infra-container-image=registry.aliyuncs.com/google_containers/pause:3.2
```



```
rm -rf /var/lib/kubelet/pki/*
rm -rf /etc/kubernetes/kubelet.conf
```



```
/usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --cgroup-driver=systemd --hostname-override=192.168.3.103 --network-plugin=cni --pod-infra-container-image=registry.aliyuncs.com/google_containers/pause:3.2  --kubelet-cgroups=/systemd/system.slice
```



```
kubectl get csr
```



```
kubectl  certificate  approve  csr-tsffv
```





# kubeadm add node



```
cat /usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf 
```



```
# Note: This dropin only works with kubeadm and kubelet v1.11+
[Service]
Environment="KUBELET_KUBECONFIG_ARGS=--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf"
Environment="KUBELET_CONFIG_ARGS=--config=/var/lib/kubelet/config.yaml"
# This is a file that "kubeadm init" and "kubeadm join" generates at runtime, populating the KUBELET_KUBEADM_ARGS variable dynamically
EnvironmentFile=-/var/lib/kubelet/kubeadm-flags.env
# This is a file that the user can use for overrides of the kubelet args as a last resort. Preferably, the user should use
# the .NodeRegistration.KubeletExtraArgs object in the configuration files instead. KUBELET_EXTRA_ARGS should be sourced from this file.
EnvironmentFile=-/etc/sysconfig/kubelet
ExecStart=
ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS
```

----------------



```
cat /var/lib/kubelet/kubeadm-flags.env
```



```
KUBELET_KUBEADM_ARGS="--cgroup-driver=systemd --hostname-override=192.168.3.103 --network-plugin=cni --pod-infra-container-image=registry.aliyuncs.com/google_containers/pause:3.2"
```



-------------------



```
cat /var/lib/kubelet/config.yaml
```



```
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
- 10.1.0.10
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
```

----------------





```
cat /usr/lib/systemd/system/kubelet.service
```



```
[Unit]
Description=kubelet: The Kubernetes Node Agent
Documentation=https://kubernetes.io/docs/

[Service]
ExecStart=/usr/bin/kubelet
Restart=always
StartLimitInterval=0
RestartSec=10

[Install]
WantedBy=multi-user.target
```

---------------



#  config file

```
mkdir -p /usr/lib/systemd/system/kubelet.service.d/
```

```
scp 192.168.3.101:/usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf  /usr/lib/systemd/system/kubelet.service.d/
```

---------------------

```
mkdir /var/lib/kubelet/
```

```
cat  > /var/lib/kubelet/kubeadm-flags.env << EOF
KUBELET_KUBEADM_ARGS="--cgroup-driver=systemd --hostname-override=192.168.3.103 --network-plugin=cni --pod-infra-container-image=registry.aliyuncs.com/google_containers/pause:3.2"
EOF
```



```
scp 192.168.3.101:/var/lib/kubelet/config.yaml  /var/lib/kubelet/
```



```
mkdir  -p /etc/kubernetes/pki/
```

