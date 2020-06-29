# flanneld 运行的节点必须是node 节点也即是必须运行了kubelet





# 获取可执行程序

```
docker run  --name flannel -it quay.io/coreos/flannel:v0.12.0-amd64  /bin/bash
```



```
mkdir /opt/bin/
```



```
docker cp flannel:/opt/bin/flanneld   /opt/bin/
docker cp flannel:/opt/bin/mk-docker-opts.sh  /opt/bin/
```



```
docker stop flannel
docker rm   flannel
```

```
echo 'export PATH=$PATH:/opt/bin/'  >> /etc/profile
source /etc/profile
```

```
which flanneld
which  mk-docker-opts.sh
```





# 删除flannel 

```
kubectl  delete -f /opt/kube-config/flannel/flannel-v0.12.0/
```



# 创建ClusterRole  和 ClusterRoleBinding

```
cat > /opt/flannel-rbac.yaml << EOF
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: flannel
rules:
  - apiGroups: ['extensions']
    resources: ['podsecuritypolicies']
    verbs: ['use']
    resourceNames: ['psp.flannel.unprivileged']
  - apiGroups:
      - ""
    resources:
      - pods
    verbs:
      - get
  - apiGroups:
      - ""
    resources:
      - nodes
    verbs:
      - list
      - watch
  - apiGroups:
      - ""
    resources:
      - nodes/status
    verbs:
      - patch
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: flannel
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: flannel
subjects:
- kind: User
  name: flannel
---
EOF
```



```
kubectl  create  -f /opt/flannel-rbac.yaml
```



和 ds 方式的区别

绑定的是 User 类型  名字是 flannel 



# 创建 flannel  使用的kubeconfig文件

192.168.3.101  上执行



```
mkdir /cert/k8s -p
cd /cert/k8s
```



这里的cn    flannel 同上面的  User  相对应

```
openssl genrsa -out flannel.key 2048
openssl req -new -key flannel.key  -subj "/CN=flannel" -out flannel.csr
```



```
cat > flannel-csr.conf  << EOF
[ req ]
default_bits = 2048
prompt = no
default_md = sha256
distinguished_name = dn
[ dn ]
[ v3_ext ]
basicConstraints = CA:FALSE
keyUsage = critical, digitalSignature, keyEncipherment
extendedKeyUsage = clientAuth
EOF
```



```
openssl x509 -req -in flannel.csr  \
-CA  /etc/kubernetes/pki/ca.crt  -CAkey /etc/kubernetes/pki/ca.key  \
-CAcreateserial -out flannel.crt -days 10000 \
-extensions v3_ext -extfile flannel-csr.conf
```



---------------

```
kubectl config set-cluster default \
      --certificate-authority=/etc/kubernetes/pki/ca.crt \
      --embed-certs=true \
      --server=https://192.168.3.101:6443 \
      --kubeconfig=/cert/k8s/flannel-kubeconfig.conf
```

```
kubectl config set-credentials  default  \
  --client-certificate=/cert/k8s/flannel.crt \
  --client-key=/cert/k8s/flannel.key \
  --embed-certs=true \
  --kubeconfig=/cert/k8s/flannel-kubeconfig.conf
```

```
kubectl config set-context  default  \
  --cluster=default \
  --user=default \
  --kubeconfig=/cert/k8s/flannel-kubeconfig.conf
```

```
kubectl config use-context default   --kubeconfig=/cert/k8s/flannel-kubeconfig.conf
```



```
cp /cert/k8s/flannel-kubeconfig.conf  /opt/
```





# 配置文件写入



```
cat >  /etc/cni/net.d/10-flannel.conflist  << EOF
{
  "name": "cbr0",
  "plugins": [
    {
      "type": "flannel",
      "delegate": {
        "hairpinMode": true,
        "isDefaultGateway": true
      }
    },
    {
      "type": "portmap",
      "capabilities": {
        "portMappings": true
      }
    }
  ]
}
EOF
```





--------------

```
mkdir /etc/kube-flannel
```



```
cat > /etc/kube-flannel/net-conf.json << EOF
{
  "Network": "10.244.0.0/16",
  "Backend": {
    "Type": "vxlan"
  }
}
EOF
```



------------



/etc/cni/net.d/10-flannel.conflist    kubelet 使用此文件

```
cat  > /etc/cni/net.d/10-flannel.conflist  << EOF
{
  "name": "cbr0",
  "cniVersion": "0.3.1",
  "plugins": [
    {
      "type": "flannel",
      "delegate": {
        "hairpinMode": true,
        "isDefaultGateway": true
      }
    },
    {
      "type": "portmap",
      "capabilities": {
        "portMappings": true
      }
    }
  ]
}
EOF
```



# 运行



```
export NODE_NAME=192.168.3.101
```

```
/opt/bin/flanneld -kubeconfig-file=/opt/flannel-kubeconfig.conf -ip-masq=1 -kube-subnet-mgr=1 
```



```
/opt/bin/flanneld -kubeconfig-file=/opt/flannel-kubeconfig.conf -ip-masq=1 -kube-subnet-mgr=1 -public-ip=192.168.3.101 -net-config-path=/etc/kube-flannel/net-conf.json
```



#  systemd



```
cat  >  /usr/lib/systemd/system/flanneld.service  << EOF
[Unit]
After=network.target
After=network-online.target
Wants=network-online.target

[Service]
Environment=NODE_NAME=192.168.3.101
ExecStart=/opt/bin/flanneld -kubeconfig-file /opt/flannel-kubeconfig.conf -ip-masq=1 -kube-subnet-mgr=1 -public-ip=192.168.3.101 -net-config-path=/etc/kube-flannel/net-conf.json
Restart=always
StartLimitInterval=0
RestartSec=10

[Install]
WantedBy=multi-user.target
EOF
```



----------



```
cat /usr/lib/systemd/system/flanneld.service
```



```
[Unit]
After=network.target
After=network-online.target
Wants=network-online.target

[Service]
Environment=NODE_NAME=192.168.3.101
ExecStart=/opt/bin/flanneld -kubeconfig-file /opt/flannel-kubeconfig.conf -ip-masq=1 -kube-subnet-mgr=1 -public-ip=192.168.3.101 -net-config-path=/etc/kube-flannel/net-conf.json
Restart=always
StartLimitInterval=0
RestartSec=10

[Install]
WantedBy=multi-user.target
```



#  重启

```
reboot
```

-------



```
ip a
```



```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:18:8f:58:39 brd ff:ff:ff:ff:ff:ff
    inet 192.168.3.101/24 brd 192.168.3.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::5054:18ff:fe8f:5839/64 scope link 
       valid_lft forever preferred_lft forever
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 02:42:55:ff:f0:76 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
4: dummy0: <BROADCAST,NOARP> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 9e:df:09:f1:c2:b2 brd ff:ff:ff:ff:ff:ff
5: kube-ipvs0: <BROADCAST,NOARP> mtu 1500 qdisc noop state DOWN group default 
    link/ether 66:7d:da:8e:57:0a brd ff:ff:ff:ff:ff:ff
```

------------



```
systemctl  start flanneld
systemctl  status  flanneld
```



------------



```
ip a
```



```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:18:8f:58:39 brd ff:ff:ff:ff:ff:ff
    inet 192.168.3.101/24 brd 192.168.3.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::5054:18ff:fe8f:5839/64 scope link 
       valid_lft forever preferred_lft forever
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 02:42:55:ff:f0:76 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
4: dummy0: <BROADCAST,NOARP> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 9e:df:09:f1:c2:b2 brd ff:ff:ff:ff:ff:ff
5: kube-ipvs0: <BROADCAST,NOARP> mtu 1500 qdisc noop state DOWN group default 
    link/ether 66:7d:da:8e:57:0a brd ff:ff:ff:ff:ff:ff
    inet 10.96.0.10/32 brd 10.96.0.10 scope global kube-ipvs0
       valid_lft forever preferred_lft forever
    inet 10.96.0.1/32 brd 10.96.0.1 scope global kube-ipvs0
       valid_lft forever preferred_lft forever
6: flannel.1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UNKNOWN group default 
    link/ether be:b0:f9:fc:a7:d3 brd ff:ff:ff:ff:ff:ff
    inet 10.244.0.0/32 scope global flannel.1
       valid_lft forever preferred_lft forever
    inet6 fe80::bcb0:f9ff:fefc:a7d3/64 scope link 
       valid_lft forever preferred_lft forever
7: cni0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP group default qlen 1000
    link/ether 2e:df:8f:50:a5:7f brd ff:ff:ff:ff:ff:ff
    inet 10.244.0.1/24 scope global cni0
       valid_lft forever preferred_lft forever
    inet6 fe80::2cdf:8fff:fe50:a57f/64 scope link 
       valid_lft forever preferred_lft forever
8: veth94701074@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue master cni0 state UP group default 
    link/ether ce:b7:57:03:a6:3a brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::ccb7:57ff:fe03:a63a/64 scope link 
       valid_lft forever preferred_lft forever
9: veth836998fb@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue master cni0 state UP group default 
    link/ether de:07:8c:71:e5:e9 brd ff:ff:ff:ff:ff:ff link-netnsid 1
    inet6 fe80::dc07:8cff:fe71:e5e9/64 scope link 
       valid_lft forever preferred_lft forever
```



---------------

```
systemctl  enable flanneld
```



```
reboot
```

