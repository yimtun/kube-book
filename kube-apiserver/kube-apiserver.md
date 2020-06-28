
# 获取程序

```
docker run -it --name=apiserver registry.aliyuncs.com/google_containers/kube-apiserver:v1.18.3 /bin/sh
```



```
docker cp apiserver:/usr/local/bin/kube-apiserver /opt
```

```
docker stop apiserver
docker rm apiserver
```



```
cp /opt/kube-apiserver  /usr/local/bin/
```



# 查看启动参数



```
ps -ef | grep kube-apiserver
```



```
kube-apiserver --advertise-address=192.168.3.101 --allow-privileged=true --authorization-mode=Node,RBAC --client-ca-file=/etc/kubernetes/pki/ca.crt --enable-admission-plugins=NodeRestriction --enable-bootstrap-token-auth=true --etcd-cafile=/etc/etcd/certs/etcd-ca.crt --etcd-certfile=/etc/etcd/certs/etcd-client.crt --etcd-keyfile=/etc/etcd/certs/etcd-client.key --etcd-servers=https://192.168.3.102:2379 --insecure-port=0 --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key --requestheader-allowed-names=front-proxy-client --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User --secure-port=6443 --service-account-key-file=/etc/kubernetes/pki/sa.pub --service-cluster-ip-range=10.96.0.0/12 --tls-cert-file=/etc/kubernetes/pki/apiserver.crt --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
```



```
cat /etc/kubernetes/manifests/kube-apiserver.yaml
```

```
    - kube-apiserver
    - --advertise-address=192.168.3.101
    - --allow-privileged=true
    - --authorization-mode=Node,RBAC
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --enable-admission-plugins=NodeRestriction
    - --enable-bootstrap-token-auth=true
    - --etcd-cafile=/etc/etcd/certs/etcd-ca.crt
    - --etcd-certfile=/etc/etcd/certs/etcd-client.crt
    - --etcd-keyfile=/etc/etcd/certs/etcd-client.key
    - --etcd-servers=https://192.168.3.102:2379
    - --insecure-port=0
    - --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt
    - --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key
    - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
    - --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt
    - --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key
    - --requestheader-allowed-names=front-proxy-client
    - --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt
    - --requestheader-extra-headers-prefix=X-Remote-Extra-
    - --requestheader-group-headers=X-Remote-Group
    - --requestheader-username-headers=X-Remote-User
    - --secure-port=6443
    - --service-account-key-file=/etc/kubernetes/pki/sa.pub
    - --service-cluster-ip-range=10.96.0.0/12
    - --tls-cert-file=/etc/kubernetes/pki/apiserver.crt
    - --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
```





```
cat > /usr/lib/systemd/system/kube-apiserver.service << EOF
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/GoogleCloudPlatform/kubernetes
After=network.target

[Service]
ExecStart=/usr/local/bin/kube-apiserver \\
--advertise-address=192.168.3.101 \\
--allow-privileged=true \\
--authorization-mode=Node,RBAC \\
--client-ca-file=/etc/kubernetes/pki/ca.crt \\
--enable-admission-plugins=NodeRestriction \\
--enable-bootstrap-token-auth=true \\
--etcd-cafile=/etc/etcd/certs/etcd-ca.crt \\
--etcd-certfile=/etc/etcd/certs/etcd-client.crt \\
--etcd-keyfile=/etc/etcd/certs/etcd-client.key \\
--etcd-servers=https://192.168.3.102:2379 \\
--insecure-port=0 \\
--kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt \\
--kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key \\
--kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname \\
--proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt \\
--proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key \\
--requestheader-allowed-names=front-proxy-client \\
--requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt \\
--requestheader-extra-headers-prefix=X-Remote-Extra- \\
--requestheader-group-headers=X-Remote-Group \\
--requestheader-username-headers=X-Remote-User \\
--secure-port=6443 \\
--service-account-key-file=/etc/kubernetes/pki/sa.pub \\
--service-cluster-ip-range=10.96.0.0/12 \\
--tls-cert-file=/etc/kubernetes/pki/apiserver.crt \\
--tls-private-key-file=/etc/kubernetes/pki/apiserver.key
Restart=on-failure
RestartSec=5
Type=notify
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
EOF
```



# 删除apiserver static pod 

```
mv /etc/kubernetes/manifests/kube-apiserver.yaml  /opt/
```


```
systemctl  start kube-apiserver
systemctl  status   kube-apiserver
```

```
systemctl  enable   kube-apiserver
```



# test



```
reboot
```



```
kubectl  get componentstatuses 
```



```
NAME                 STATUS    MESSAGE             ERROR
scheduler            Healthy   ok                  
controller-manager   Healthy   ok                  
etcd-0               Healthy   {"health":"true"}   
```

