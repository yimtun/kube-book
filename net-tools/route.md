

# 查看所有路由表



```
ip rule list
```



```
0:	from all lookup local 
32766:	from all lookup main 
32767:	from all lookup default 
```



# 查看某一个路由表

```
ip route show table local
```



```
local 10.96.0.1 dev kube-ipvs0 proto kernel scope host src 10.96.0.1 
broadcast 10.96.0.1 dev kube-ipvs0 proto kernel scope link src 10.96.0.1 
local 10.96.0.10 dev kube-ipvs0 proto kernel scope host src 10.96.0.10 
broadcast 10.96.0.10 dev kube-ipvs0 proto kernel scope link src 10.96.0.10 
local 10.96.0.100 dev kube-ipvs0 proto kernel scope host src 10.96.0.100 
broadcast 10.96.0.100 dev kube-ipvs0 proto kernel scope link src 10.96.0.100 
local 10.244.0.0 dev flannel.1 proto kernel scope host src 10.244.0.0 
broadcast 10.244.0.0 dev cni0 proto kernel scope link src 10.244.0.1 
local 10.244.0.1 dev cni0 proto kernel scope host src 10.244.0.1 
broadcast 10.244.0.255 dev cni0 proto kernel scope link src 10.244.0.1 
broadcast 127.0.0.0 dev lo proto kernel scope link src 127.0.0.1 
local 127.0.0.0/8 dev lo proto kernel scope host src 127.0.0.1 
local 127.0.0.1 dev lo proto kernel scope host src 127.0.0.1 
broadcast 127.255.255.255 dev lo proto kernel scope link src 127.0.0.1 
broadcast 172.17.0.0 dev docker0 proto kernel scope link src 172.17.0.1 
local 172.17.0.1 dev docker0 proto kernel scope host src 172.17.0.1 
broadcast 172.17.255.255 dev docker0 proto kernel scope link src 172.17.0.1 
broadcast 192.168.3.0 dev eth0 proto kernel scope link src 192.168.3.101 
local 192.168.3.101 dev eth0 proto kernel scope host src 192.168.3.101 
broadcast 192.168.3.255 dev eth0 proto kernel scope link src 192.168.3.101 
```

