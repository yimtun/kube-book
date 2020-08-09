
# 查看当前kubelet 客户端证书


```
openssl x509 -in /var/lib/kubelet/pki/kubelet-client-current.pem   -noout -text  |grep Not
```



----------------

```
openssl x509  -noout -subject -issuer  -in  /var/lib/kubelet/pki/kubelet-client-current.pem
```



```
subject= /O=system:nodes/CN=system:node:192.168.3.104
issuer= /CN=kubernetes
```



# 设置kubelet证书有效期 



192.168.3.101



```
vim /etc/kubernetes/manifests/kube-controller-manager.yaml
```



启动参数处添加一行

```
- --experimental-cluster-signing-duration=0h8m0s
```





#  删除干扰因素

删除允许节点更新证书的clusterrolebinding





> https://v1-18.docs.kubernetes.io/docs/reference/command-line-tools-reference/kubelet-tls-bootstrapping/#approval







```
kubectl get clusterrolebinding -o custom-columns='Name:metadata.name,RoleRef:roleRef.name,Subjects:subjects'  | grep 'system:nodes'
```



```
kubectl get clusterrolebinding  kubeadm:node-autoapprove-certificate-rotation -o yaml
```







```
kubectl delete  clusterrolebinding  kubeadm:node-autoapprove-certificate-rotation
```



# 重新将节点加入集群 造成kubelet 客户端证书过期的情况



### 192.168.3.101







```
kubectl  -n kube-system  delete  secrets  bootstrap-token-398c7c
```



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
kubectl delete  clusterrolebinding  kubelet-bootstrap
```





```
kubectl create clusterrolebinding kubelet-bootstrap --clusterrole=system:node-bootstrapper --group=system:bootstrappers	
```



```
kubectl delete  clusterrolebinding  auto-approve-csrs-for-group
```



```
kubectl create clusterrolebinding auto-approve-csrs-for-group \
--clusterrole=system:certificates.k8s.io:certificatesigningrequests:nodeclient \
--group=system:bootstrappers
```









清除环境 重新添加node



```
kubectl drain 192.168.3.104 --ignore-daemonsets
kubectl delete node 192.168.3.104
```


```
kubectl  delete csr --all
```


```
kubectl get csr -w
```

```
kubectl get  node  -w
```



### 192.168.3.104



```
systemctl  stop kubelet
```



```
scp 192.168.3.101:/opt/bootstrap-kubelet.conf  /etc/kubernetes/
```



```
systemctl disable kubelet
```



```
rm -rf /var/lib/kubelet/pki/*
rm -rf /etc/kubernetes/kubelet.conf
rm -rf /etc/cni/net.d/10-flannel.conflist
```

```
reboot
```



```
systemctl start kubelet
```



----------------

```
openssl x509 -in /var/lib/kubelet/pki/kubelet-client-current.pem   -noout -text  |grep Not
```











------------------------



#  允许kubelete更新客户端证书 的测试



To enable the kubelet to renew its own client certificate



193.192.168.3.101 



```
kubectl create clusterrolebinding auto-approve-renewals-for-nodes \
--clusterrole=system:certificates.k8s.io:certificatesigningrequests:selfnodeclient \
--group=system:nodes
```





```
rm -rf /etc/kubernetes/kubelet.conf
```



```
systemctl  restart kubelet
```



```
rm -rf /etc/kubernetes/bootstrap-kubelet.conf
```









#  复原







```
kubectl create clusterrolebinding kubeadm:node-autoapprove-certificate-rotation \
--clusterrole=system:certificates.k8s.io:certificatesigningrequests:selfnodeclient \
--group=system:nodes
```



删除之前对此文件的修改

```
vim /etc/kubernetes/manifests/kube-controller-manager.yaml
```

