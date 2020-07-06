apiserver

```
- --enable-aggregator-routing=true
```


controller-manager

```
- --experimental-cluster-signing-duration=17520h0m0s
```


修改后 kubelet  会自动重启 api-server static pod



```
vim /etc/kubernetes/manifests/kube-controller-manager.yaml
```

修改后自动生效

