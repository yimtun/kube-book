###  kubeconfig

> https://v1-18.docs.kubernetes.io/zh/docs/concepts/configuration/organize-cluster-access-kubeconfig/



api server 对外提供服务

作为api server 的客户端 比如 kubelet    kubectl   kube-controller-manager   kube-schduler  kube-proxy

等组件 都可以使用 kubeconfig 文件作为 访问apiserver  的凭证



使用kubeadm 安装后 会自动创建一个kubeconfig 文件  之前文档中执行的如下命令就是 复制一个 kubeconfig 文件 给 kubectl 使用 这个文件是kubeadm创建好的

```
mkdir  /root/.kube
cp /etc/kubernetes/admin.conf  /root/.kube/config
```



 



kubeconfig  中的context 





#### 	 以kubectl  使用kubeconfig 文件为例 说明 



 kubectl   使用kubeconfig默认路径就是  /root/.kube/config 如果存在这个文件

执行kubectl 的时候不必手动只当这个文件



##### 1 先查看  current-context     如果有值  就使用这个 current-context  中指定的 context





##### 2   这里显示的  current-context   的值为    kubernetes-admin@kubernetes   这代表一个context 的名字

```
current-context: kubernetes-admin@kubernetes
```



##### 3  寻找 名字为kubernetes-admin@kubernetes  的context

它的内容如下



```
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
```



这个 context 将一个 名字为 kubernetes 的k8s  cluster 和一个名字为 kubernetes-admin 的user相关联起来



##### 4  查看 kubeconfig  中名字为  kubernetes 的cluster信息



```
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUMvVENDQWVXZ0F3SUJBZ0lKQUsxMXBMZ3JidDdvTUEwR0NTcUdTSWIzRFFFQkN3VUFNQlV4RXpBUkJnTlYKQkFNTUNtdDFZbVZ5Ym1WMFpYTXdIaGNOTWpBd05qSXlNVEExTmpNNVdoY05ORGN4TVRBNE1UQTFOak01V2pBVgpNUk13RVFZRFZRUUREQXByZFdKbGNtNWxkR1Z6TUlJQklqQU5CZ2txaGtpRzl3MEJBUUVGQUFPQ0FROEFNSUlCCkNnS0NBUUVBeFI4WlcxZzlBVUZ1cVNzVmhIRVVtNlliUzN3ajliWnU4SFM4bGZHeGZ5OU1kUStzcW9JOW1GckgKc3BTem1QU3pybEs4c1BuRW9yMzA0VnhZanJZTXJsS3hhYTQzZS8zN3JPRThBNmlGeDBUNmd3OEMwQXpmbTV0SQpOUmY1elIwSmR2ZDVqUWY0V2xtNzNWelpyNzcwOVpqVlplU0tZYU9HbEtuNk5kNVdHWDkxeW9ob3FaL2RMakZ3CnFycFFNMkhobGMvbHFwNFQrSS83L3EyVG1CanBKMDJFV2RxNzdra25XbUhPWG5yc0d5TVhLZkp1VG5zUWlOcTIKNy9rUHcwd1FxQW9VeDk4RHE2TVV0SnFnSERMQ1JEem4wd2lkNXdhVUVlVkp1YlR6Rk9IMmRFUHRJQ2taN2t6WApSS1Q0Y2VxbDVtL01QbEdPT0tFMTFraU1rMGVIQlFJREFRQUJvMUF3VGpBZEJnTlZIUTRFRmdRVWplYjhRcXF6Cm9zRDh3RXZGbWZOWnIzUEMvRU13SHdZRFZSMGpCQmd3Rm9BVWplYjhRcXF6b3NEOHdFdkZtZk5acjNQQy9FTXcKREFZRFZSMFRCQVV3QXdFQi96QU5CZ2txaGtpRzl3MEJBUXNGQUFPQ0FRRUFydDVybUhRTUdwVmNjV280MUM4NgpzYTVRSTlSckNGRDE5YThLTVFGbTVtTGFCYXdPaWtFUDJ2N0VaTUNVdlVla1l4ZGx0OElzNGNOVVJrcDlvN0FaClVPTzZOaEhCNFdubjhWeWpKODZGczkxUnVaT0NPWW9OTG9LTTJ2aWV1MDdPcHErZUtVdnY5MGVrZzYya3BrOVoKSDllUGx4azV1Z3hyaTZKSVo3ZlVNalJoWUdXZnd6YmJOUUpZWXJ6VmFhd21sMUpTWXJlZjFsL3JVT2RVN1ZUMQpHTXF4ZDc5WEQvRDlIU05ocVNJQThNZ1Q0SThHSEc1Mng5blVSN1QzSTA2akdLTFpFMGh3VUJNUUtxWkx4c0dWCk1XVXRlL1Z0ZmZ1cHpDdHhpdFRxTDhZcVhVUXFnekprMSs4UGlvUURlRlA2bStSMFdvRmthK3BOczhPTkNVeHkKWGc9PQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
    server: https://192.168.3.101:6443
  name: kubernetes
```



就是 kube-apiserver 的地址端口 和 ca 证书  的内容

base64 解码



```
echo 'LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUMvVENDQWVXZ0F3SUJBZ0lKQUsxMXBMZ3JidDdvTUEwR0NTcUdTSWIzRFFFQkN3VUFNQlV4RXpBUkJnTlYKQkFNTUNtdDFZbVZ5Ym1WMFpYTXdIaGNOTWpBd05qSXlNVEExTmpNNVdoY05ORGN4TVRBNE1UQTFOak01V2pBVgpNUk13RVFZRFZRUUREQXByZFdKbGNtNWxkR1Z6TUlJQklqQU5CZ2txaGtpRzl3MEJBUUVGQUFPQ0FROEFNSUlCCkNnS0NBUUVBeFI4WlcxZzlBVUZ1cVNzVmhIRVVtNlliUzN3ajliWnU4SFM4bGZHeGZ5OU1kUStzcW9JOW1GckgKc3BTem1QU3pybEs4c1BuRW9yMzA0VnhZanJZTXJsS3hhYTQzZS8zN3JPRThBNmlGeDBUNmd3OEMwQXpmbTV0SQpOUmY1elIwSmR2ZDVqUWY0V2xtNzNWelpyNzcwOVpqVlplU0tZYU9HbEtuNk5kNVdHWDkxeW9ob3FaL2RMakZ3CnFycFFNMkhobGMvbHFwNFQrSS83L3EyVG1CanBKMDJFV2RxNzdra25XbUhPWG5yc0d5TVhLZkp1VG5zUWlOcTIKNy9rUHcwd1FxQW9VeDk4RHE2TVV0SnFnSERMQ1JEem4wd2lkNXdhVUVlVkp1YlR6Rk9IMmRFUHRJQ2taN2t6WApSS1Q0Y2VxbDVtL01QbEdPT0tFMTFraU1rMGVIQlFJREFRQUJvMUF3VGpBZEJnTlZIUTRFRmdRVWplYjhRcXF6Cm9zRDh3RXZGbWZOWnIzUEMvRU13SHdZRFZSMGpCQmd3Rm9BVWplYjhRcXF6b3NEOHdFdkZtZk5acjNQQy9FTXcKREFZRFZSMFRCQVV3QXdFQi96QU5CZ2txaGtpRzl3MEJBUXNGQUFPQ0FRRUFydDVybUhRTUdwVmNjV280MUM4NgpzYTVRSTlSckNGRDE5YThLTVFGbTVtTGFCYXdPaWtFUDJ2N0VaTUNVdlVla1l4ZGx0OElzNGNOVVJrcDlvN0FaClVPTzZOaEhCNFdubjhWeWpKODZGczkxUnVaT0NPWW9OTG9LTTJ2aWV1MDdPcHErZUtVdnY5MGVrZzYya3BrOVoKSDllUGx4azV1Z3hyaTZKSVo3ZlVNalJoWUdXZnd6YmJOUUpZWXJ6VmFhd21sMUpTWXJlZjFsL3JVT2RVN1ZUMQpHTXF4ZDc5WEQvRDlIU05ocVNJQThNZ1Q0SThHSEc1Mng5blVSN1QzSTA2akdLTFpFMGh3VUJNUUtxWkx4c0dWCk1XVXRlL1Z0ZmZ1cHpDdHhpdFRxTDhZcVhVUXFnekprMSs4UGlvUURlRlA2bStSMFdvRmthK3BOczhPTkNVeHkKWGc9PQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==' | base64 -d > /opt/test.crt
```



使用diff 查看是不是同一个证书

```
diff /etc/kubernetes/pki/ca.crt  /opt/test.crt
```



##### 5  查看 kubeconfig  中名字为  kubernetes-admin 的user的信息

```
users:
- name: kubernetes-admin
  user:
    client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURFekNDQWZ1Z0F3SUJBZ0lJYmlkVTF3R25DVmd3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXd3S2EzVmlaWEp1WlhSbGN6QWVGdzB5TURBMk1qSXhNRFUyTXpsYUZ3MHlNVEEzTURNd01qQXlORGRhTURReApGekFWQmdOVkJBb1REbk41YzNSbGJUcHRZWE4wWlhKek1Sa3dGd1lEVlFRREV4QnJkV0psY201bGRHVnpMV0ZrCmJXbHVNSUlCSWpBTkJna3Foa2lHOXcwQkFRRUZBQU9DQVE4QU1JSUJDZ0tDQVFFQXhicGVlZEVKY01vZFlrU1AKNWVRYzVCRjM0cmg0M0lxdGpXZlpQcEQwR0YwaDV2dFY1Rk0zV29hSjlqWExzVmV1T21RN2NzeG9GUlBuMUZTZgpGQnJJM1p4RmM5bXVqNzFDNGJ6OWR1UW82Y2JrNFdqOTdCU3JTb3VsbXQvU0E2OVJPK0h6b2Q1RnpZT01tNkovCnRya1pvV2FwZXRrOGRmQU52MzR0a2pNaTBpQmRmaXArdVVvMGlEbER0ZHYyUlFGN2owUW9qSC9wZHdxNzJSMjEKN2Nobmt3ZkRuM1JacXFWcmk2RjUyNjMzbkxOU1h6amlZVWgreXNkN1Y3Wk55aXNXV0F1b1BTYUZIK2ZrNTRyMwpLcm5PZTkrWUtOck12UzR2aWY5empiQ2loU2pDSEc2NVpJNi90VWxHR1M0bG8xRzVNVEE0Q2QzaXNxcm5Pbm5MCnJQNzh5d0lEQVFBQm8wZ3dSakFPQmdOVkhROEJBZjhFQkFNQ0JhQXdFd1lEVlIwbEJBd3dDZ1lJS3dZQkJRVUgKQXdJd0h3WURWUjBqQkJnd0ZvQVVqZWI4UXFxem9zRDh3RXZGbWZOWnIzUEMvRU13RFFZSktvWklodmNOQVFFTApCUUFEZ2dFQkFEUFZjOGY5bDNXWittR08wU0xzbkV6MXFPSEMzamZNYll5SkVSRnlIdmFGYjJraS9ZMXMreTdHCktnbGVMQ0JPN29pWXRDSW1zS1VNZk55WHo1cUZVdFJVTEQ0RS9YbVBJVmpyM2Y3VnVwaUZOMXlxZlQra3lCQmQKcXduSEdzOFJFN0lVcFFJOEFVTWlFNm91OHBVUVF6NzBydlVnVnZ4d051bVFKRnQ3YkRpVVRoaW9rbkVlMDNGTApxTUNuNzBueUtaRXdRWlRaYmJTY3MzZlRHbzFVMHUrYzBLQ2c2MHBWUkpqNmlPa2ZGOWNVQTZBek5vdUdwd3NyCkdnTGFoK1ltWjYwRzhMM1JTeHZ3bmFQdU5BNmxGazUyazRGS0dqVlEvVTgwY0w3WmZDOXc5WDgvSEVvcmpJSCsKeGRkTUt6bE9zcitISGk1MkoyOHlNb3RmOHg3ci84Zz0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
    client-key-data: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFcEFJQkFBS0NBUUVBeGJwZWVkRUpjTW9kWWtTUDVlUWM1QkYzNHJoNDNJcXRqV2ZaUHBEMEdGMGg1dnRWCjVGTTNXb2FKOWpYTHNWZXVPbVE3Y3N4b0ZSUG4xRlNmRkJySTNaeEZjOW11ajcxQzRiejlkdVFvNmNiazRXajkKN0JTclNvdWxtdC9TQTY5Uk8rSHpvZDVGellPTW02Si90cmtab1dhcGV0azhkZkFOdjM0dGtqTWkwaUJkZmlwKwp1VW8waURsRHRkdjJSUUY3ajBRb2pIL3Bkd3E3MlIyMTdjaG5rd2ZEbjNSWnFxVnJpNkY1MjYzM25MTlNYemppCllVaCt5c2Q3VjdaTnlpc1dXQXVvUFNhRkgrZms1NHIzS3JuT2U5K1lLTnJNdlM0dmlmOXpqYkNpaFNqQ0hHNjUKWkk2L3RVbEdHUzRsbzFHNU1UQTRDZDNpc3Fybk9ubkxyUDc4eXdJREFRQUJBb0lCQVFDRHB0ZUkzSG9nc3pKbApYNmxBTkdaWUpKbGlSOW1SWG5TNEZsRTdxMkFiYU1kTitFTDBSOFF2YmkwbDFpUE43TWVBOFlQenA4NFZXcStkClhNcWVwRWJoNTA4SEdBVjJoMW1rM0NVWHFFcmxmUnlnU1R2b21NcUVVLzdyNCtMOXVSbXBlWVN5WGtDejJjY2gKU1UwbjZJNzhQRkxVRFJpSW5sRkpFMFpjZGRmVmQ3TTZDOHUrdGYxaklRcHBOVm03bytoWUgrVlFUQ25rejYwZgozdHpGK2dPOEJIT2dySVl5YzVEOUNleGx6SUM4d3NIMmlQSXNETjI3NnBDRVZuZzZJejV4eVFDTzFSREFhdlcwCmVjb294Q084dDlBOUZka1N0WW15TVd0VDhnemRoWUpYZGQ2ZSs0K090ZjNiclVQMXFkSlNtRWpMVjN3VVFsN3kKeGlvWTQwNmhBb0dCQU8ydFF0UitvUTlicGRmSjJwQXBoODdKTnE4alNOK0FYRGdZRzFtTTV0TDB5c09wZlJCRApQdXd6Y2pXUHBRUVVvMWU1bDhmcG1hd0JGZmIraXRCemdBOXJ6dTNYTVlFSm5NT0ZBcHNNM2FLd2xGMzdIa3IxCk5QL2E2R0FvTzhZUkNkV3VVVm9LRTVCWk1tMHFpMkdLaEdIeGFjZms4REYwZXVrZXBkZkRnL2hYQW9HQkFOVDQKcjkrc0JOcG04bm9QS1FSaXNBWTBwb3BJVkVvTHZ1Z1oxTlRVTkZhUFQvQ1BXREFpMStTQnJqRE1OS2U1NHJJTwpjRS9IUThIUkZXKzBOQ3RGZUUvT0NlS1pGUFR4TjYvNURWVEJxekZaeVlzMFYzNHYvSllxVWQveTdod1RKRXhiCll3Nk5VakpycVlNT29GUGNkVmNWR050MTNUTFVGRlJ2d3lzNm51YXRBb0dBT2k1K3ZKdmUrMjU0ODVFVE10VW4KektRUEFlS0dWVWdMeXlPRGxuRmFrK3Vlc3pVTFMyN2F1V0dDcEwvc0trcVBEY3Q5NzA4czhpRTE2a2UzWFgzWQpyRzI4c3haSnBRZmdXekIxU2RWbGNBei8xTjNETmFBL0FCN3JZWmFYdzAycWRhZDlmS2dZeis0MTNPbGNRMTF3Ck9MV3JLbWJOc1oyTTlRSXVvTm5ZdFhNQ2dZQnl4OGs1K21Mdk5wYXVsQ2NlRnZZWmtoekQ0SEdWS3Jsc0xDZloKd0xpb2dqcXFRd2RiZ2h2cktyMHZ6WTcvYXA2MEtqWDd1VUJhWUE1MmtwK2ZScVN1RmpTYnJMZHZ2K1dzY01UdgpqaVZ1eHA1cDZQN1NvcGcyY242SC9VeTVVdE80VjNTT3JqbkR0T1M2SHBMb1A2UDZHQU82bTg1b2k0YWRiMUszCnBMTnBUUUtCZ1FEZGhVT1Bub2hUZC9NYW5BOG9WS1hvTy9weG9pZThEQmdmdEt0ZGhodHFrMXpKL21mYTRVVHoKRjhqWjZKT1hGaDhUNmdjZEFyK3I5TnBPRVdLQXJHNTl0NzVrMUhOOHQ5MmZqa2Vabzg5d1BHQ0l6OEpOTCtqMgpiUisvVEkzY3pySkZLb0xYNlROS0lkb1VuTzFnaGpWNjZzMlN2bjdacUxjSmdtdWIwaUlaTGc9PQotLS0tLUVORCBSU0EgUFJJVkFURSBLRVktLS0tLQo=
```



存储用户认证信息     这里式客户端证书 和一个私钥文件    

也可以存储token 作为认证的凭证



查看这个在kubeconfig 文件中  user  名字为kubernetes-admin 的  证书信息

```
echo 'LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURFekNDQWZ1Z0F3SUJBZ0lJYmlkVTF3R25DVmd3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXd3S2EzVmlaWEp1WlhSbGN6QWVGdzB5TURBMk1qSXhNRFUyTXpsYUZ3MHlNVEEzTURNd01qQXlORGRhTURReApGekFWQmdOVkJBb1REbk41YzNSbGJUcHRZWE4wWlhKek1Sa3dGd1lEVlFRREV4QnJkV0psY201bGRHVnpMV0ZrCmJXbHVNSUlCSWpBTkJna3Foa2lHOXcwQkFRRUZBQU9DQVE4QU1JSUJDZ0tDQVFFQXhicGVlZEVKY01vZFlrU1AKNWVRYzVCRjM0cmg0M0lxdGpXZlpQcEQwR0YwaDV2dFY1Rk0zV29hSjlqWExzVmV1T21RN2NzeG9GUlBuMUZTZgpGQnJJM1p4RmM5bXVqNzFDNGJ6OWR1UW82Y2JrNFdqOTdCU3JTb3VsbXQvU0E2OVJPK0h6b2Q1RnpZT01tNkovCnRya1pvV2FwZXRrOGRmQU52MzR0a2pNaTBpQmRmaXArdVVvMGlEbER0ZHYyUlFGN2owUW9qSC9wZHdxNzJSMjEKN2Nobmt3ZkRuM1JacXFWcmk2RjUyNjMzbkxOU1h6amlZVWgreXNkN1Y3Wk55aXNXV0F1b1BTYUZIK2ZrNTRyMwpLcm5PZTkrWUtOck12UzR2aWY5empiQ2loU2pDSEc2NVpJNi90VWxHR1M0bG8xRzVNVEE0Q2QzaXNxcm5Pbm5MCnJQNzh5d0lEQVFBQm8wZ3dSakFPQmdOVkhROEJBZjhFQkFNQ0JhQXdFd1lEVlIwbEJBd3dDZ1lJS3dZQkJRVUgKQXdJd0h3WURWUjBqQkJnd0ZvQVVqZWI4UXFxem9zRDh3RXZGbWZOWnIzUEMvRU13RFFZSktvWklodmNOQVFFTApCUUFEZ2dFQkFEUFZjOGY5bDNXWittR08wU0xzbkV6MXFPSEMzamZNYll5SkVSRnlIdmFGYjJraS9ZMXMreTdHCktnbGVMQ0JPN29pWXRDSW1zS1VNZk55WHo1cUZVdFJVTEQ0RS9YbVBJVmpyM2Y3VnVwaUZOMXlxZlQra3lCQmQKcXduSEdzOFJFN0lVcFFJOEFVTWlFNm91OHBVUVF6NzBydlVnVnZ4d051bVFKRnQ3YkRpVVRoaW9rbkVlMDNGTApxTUNuNzBueUtaRXdRWlRaYmJTY3MzZlRHbzFVMHUrYzBLQ2c2MHBWUkpqNmlPa2ZGOWNVQTZBek5vdUdwd3NyCkdnTGFoK1ltWjYwRzhMM1JTeHZ3bmFQdU5BNmxGazUyazRGS0dqVlEvVTgwY0w3WmZDOXc5WDgvSEVvcmpJSCsKeGRkTUt6bE9zcitISGk1MkoyOHlNb3RmOHg3ci84Zz0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=' | base64 -d > /opt/client.crt
```





```
openssl x509  -noout -subject -issuer  -in  /opt/client.crt 
```



```
subject= /O=system:masters/CN=kubernetes-admin
issuer= /CN=kubernetes
```









-------------

##### 6 可以看见 一个kubeconfig 文件可以有多个 cluster  和user

```
clusters:
users:
```



不同的user和 不同的cluster 可以任意组合出不同的context

使用kubeconfig 的文件的程序比如kubectl  也就可以随意切换不同的context  

这样就可以访问不同的cluster  ，而且可以用不同的用户访问不同的cluter 

当然也可以用不同的用户访问同一个cluster





如果不额外指定 context 就会使用   current-context 里指向的 context



##### 7 注意

kubeconfig 文件中所有的 name  字段的值 都仅仅式一个引用而已 并没有实际任何意义

context 的name 为了标识 不同的  cluster  和 user  的组合

cluster   的name 为了标识不同的 k8s 集群

user 中的name  为了标识不同的 凭证信息





##### 8 查看 kubeconfig文件中用户名为kubernetes-admin的user在集群名为kubernetes的k8s 集群中的权限



```
openssl x509  -noout -subject   -in  /opt/client.crt 
```



O 被k8s 识别为组   CN 被识别为用户

```
subject= /O=system:masters/CN=kubernetes-admin
```







查找哪些 clusterrolebinding中   有名字为      kubernetes-admin   的subject   无论什么类型的subject

```
kubectl get clusterrolebinding -o custom-columns='Name:metadata.name,RoleRef:roleRef.name,Subjects:subjects' | grep 'kubernetes-admin'
```





查找哪些 rolebinding中   有名字为        kubernetes-admin   的subject   无论什么类型的subject

```
kubectl  get rolebinding -A -o custom-columns='Name:metadata.name,RoleRef:roleRef.name,Subjects:subjects' | grep 'kubernetes-admin'
```

---------------



查找哪些 clusterrolebinding 中 有名字为     system:masters   的subject   无论什么类型的subject



```
kubectl get clusterrolebinding -o custom-columns='Name:metadata.name,RoleRef:roleRef.name,Subjects:subjects' | grep 'system:masters'
```





```
kubectl get clusterrolebinding cluster-admin  -o yaml
```



```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  creationTimestamp: "2020-07-03T02:03:09Z"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  managedFields:
  - apiVersion: rbac.authorization.k8s.io/v1
    fieldsType: FieldsV1
    fieldsV1:
      f:metadata:
        f:annotations:
          .: {}
          f:rbac.authorization.kubernetes.io/autoupdate: {}
        f:labels:
          .: {}
          f:kubernetes.io/bootstrapping: {}
      f:roleRef:
        f:apiGroup: {}
        f:kind: {}
        f:name: {}
      f:subjects: {}
    manager: kube-apiserver
    operation: Update
    time: "2020-07-03T02:03:09Z"
  name: cluster-admin
  resourceVersion: "626308"
  selfLink: /apis/rbac.authorization.k8s.io/v1/clusterrolebindings/cluster-admin
  uid: 9b5802d1-029c-40e6-85f9-ca0ed7a05869
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: Group
  name: system:masters
```



```
cluster-admin
```



--------------

```
kubectl  get clusterrole cluster-admin  -o yaml
```





```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  creationTimestamp: "2020-07-03T02:03:09Z"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  managedFields:
  - apiVersion: rbac.authorization.k8s.io/v1
    fieldsType: FieldsV1
    fieldsV1:
      f:metadata:
        f:annotations:
          .: {}
          f:rbac.authorization.kubernetes.io/autoupdate: {}
        f:labels:
          .: {}
          f:kubernetes.io/bootstrapping: {}
      f:rules: {}
    manager: kube-apiserver
    operation: Update
    time: "2020-07-03T02:03:09Z"
  name: cluster-admin
  resourceVersion: "626252"
  selfLink: /apis/rbac.authorization.k8s.io/v1/clusterroles/cluster-admin
  uid: 1c77f798-6968-4c40-a73c-654de6ec71ef
rules:
- apiGroups:
  - '*'
  resources:
  - '*'
  verbs:
  - '*'
- nonResourceURLs:
  - '*'
  verbs:
  - '*'
```







查找哪些 rolebinding 中 有名字为     system:masters   的subject   无论什么类型的subject



```
kubectl  get rolebinding -A -o custom-columns='Name:metadata.name,RoleRef:roleRef.name,Subjects:subjects' | grep 'system:masters'
```





kubeconfig  文件里的用户并不是 k8s  中的用户  ，仅仅是为了 定位 凭证而存在的

凭证可以是 token  也可以是 证书

  





###  kubectl

k8s  的 命令行 客户端工具





### RBAC

基于角色的访问控制，是k8s 常用的一种权限管理的方法



```
RBAC基于 角色 的 权限 访问 控制Role- Based Access Control， 
```



```
role            角色             区分命名空间
clusterrole     集群角色          不分命名空间  
无论是 role  还是clusterrole 都可看作是一组权限的集合
```



```
rolebinding             角色绑定          区分命名空间
clusterrolebinding      集群角色绑定       不分命名空间
```



```
subject    主体    可以是        1 用户   2  组    3 服务账户
```



```
rolebinding  将role同 subject绑定    subject 中的用户,组,或者服务账户就拥有了role里定义的权限

clusterrolebinding  将clusterrole同subject绑定  subject中的用户,组,或者服务账户就拥有了  clusterrole里定义的权限
```











### BootstrapToken

当 Kubernetes 集群 中有 非常 多的 节点 时， 
手动 为 每个 节点 配置 TLS 认证 比较 烦琐， 
为此 Kubernetes 提供 了 BootstrapToken 认证， 
其 也 被称为 引导 Token。 为token 授予一定的权限，  认证 通过，
可以 自动 为 节点 颁发 客户端证书， 这是 一种 引导 Token 的 机制。 





```
kubectl api-resources --sort-by=name
```



```
NAME                              SHORTNAMES   APIGROUP                       NAMESPACED   KIND
apiservices                                    apiregistration.k8s.io         false        APIService
bindings                                                                      true         Binding
certificatesigningrequests        csr          certificates.k8s.io            false        CertificateSigningRequest
clusterrolebindings                            rbac.authorization.k8s.io      false        ClusterRoleBinding
clusterroles                                   rbac.authorization.k8s.io      false        ClusterRole
componentstatuses                 cs                                          false        ComponentStatus
configmaps                        cm                                          true         ConfigMap
controllerrevisions                            apps                           true         ControllerRevision
cronjobs                          cj           batch                          true         CronJob
csidrivers                                     storage.k8s.io                 false        CSIDriver
csinodes                                       storage.k8s.io                 false        CSINode
customresourcedefinitions         crd,crds     apiextensions.k8s.io           false        CustomResourceDefinition
daemonsets                        ds           apps                           true         DaemonSet
deployments                       deploy       apps                           true         Deployment
endpoints                         ep                                          true         Endpoints
endpointslices                                 discovery.k8s.io               true         EndpointSlice
events                            ev                                          true         Event
events                            ev           events.k8s.io                  true         Event
horizontalpodautoscalers          hpa          autoscaling                    true         HorizontalPodAutoscaler
ingressclasses                                 networking.k8s.io              false        IngressClass
ingresses                         ing          extensions                     true         Ingress
ingresses                         ing          networking.k8s.io              true         Ingress
jobs                                           batch                          true         Job
leases                                         coordination.k8s.io            true         Lease
limitranges                       limits                                      true         LimitRange
localsubjectaccessreviews                      authorization.k8s.io           true         LocalSubjectAccessReview
mutatingwebhookconfigurations                  admissionregistration.k8s.io   false        MutatingWebhookConfiguration
namespaces                        ns                                          false        Namespace
networkpolicies                   netpol       networking.k8s.io              true         NetworkPolicy
nodes                             no                                          false        Node
nodes                                          metrics.k8s.io                 false        NodeMetrics
persistentvolumeclaims            pvc                                         true         PersistentVolumeClaim
persistentvolumes                 pv                                          false        PersistentVolume
poddisruptionbudgets              pdb          policy                         true         PodDisruptionBudget
pods                              po                                          true         Pod
pods                                           metrics.k8s.io                 true         PodMetrics
podsecuritypolicies               psp          policy                         false        PodSecurityPolicy
podtemplates                                                                  true         PodTemplate
priorityclasses                   pc           scheduling.k8s.io              false        PriorityClass
replicasets                       rs           apps                           true         ReplicaSet
replicationcontrollers            rc                                          true         ReplicationController
resourcequotas                    quota                                       true         ResourceQuota
rolebindings                                   rbac.authorization.k8s.io      true         RoleBinding
roles                                          rbac.authorization.k8s.io      true         Role
runtimeclasses                                 node.k8s.io                    false        RuntimeClass
secrets                                                                       true         Secret
selfsubjectaccessreviews                       authorization.k8s.io           false        SelfSubjectAccessReview
selfsubjectrulesreviews                        authorization.k8s.io           false        SelfSubjectRulesReview
serviceaccounts                   sa                                          true         ServiceAccount
services                          svc                                         true         Service
statefulsets                      sts          apps                           true         StatefulSet
storageclasses                    sc           storage.k8s.io                 false        StorageClass
subjectaccessreviews                           authorization.k8s.io           false        SubjectAccessReview
tokenreviews                                   authentication.k8s.io          false        TokenReview
validatingwebhookconfigurations                admissionregistration.k8s.io   false        ValidatingWebhookConfiguration
volumeattachments                              storage.k8s.io                 false        VolumeAttachment
```



| NAME                       | SHORTNAMES | APIGROUP                  | NAMESPACED | KIND                      |
| -------------------------- | ---------- | ------------------------- | ---------- | ------------------------- |
| apiservices                |            | apiregistration.k8s.io    | false      | APIService                |
| bindings                   |            |                           | true       | Binding                   |
| certificatesigningrequests | csr        | certificates.k8s.io       | false      | CertificateSigningRequest |
| clusterrolebindings        |            | rbac.authorization.k8s.io | false      | ClusterRoleBinding        |
| clusterroles               |            | rbac.authorization.k8s.io | false      | ClusterRole               |
| componentstatuses          | cs         |                           | false      | ComponentStatus           |
| configmaps                 | cm         |                           | true       | ConfigMap                 |
| controllerrevisions        |            | apps                      | true       | ControllerRevision        |
| cronjobs                   | cj         | batch                     | true       | CronJob                   |
| csidrivers                 |            | storage.k8s.io            | false      | CSIDriver                 |
| csinodes                   |            | storage.k8s.io            | false      | CSINode                   |
| customresourcedefinitions  | crd,crds   | apiextensions.k8s.io      | false      | CustomResourceDefinition  |
| daemonsets                 | ds         | apps                      | true       | DaemonSet                 |
| deployments                | deploy     | apps                      | true       | Deployment                |
| endpoints                  | ep         |                           | true       | Endpoints                 |
| endpointslices             |            | discovery.k8s.io          | true       | EndpointSlice             |
|                            |            |                           |            |                           |
|                            |            |                           |            |                           |
|                            |            |                           |            |                           |
|                            |            |                           |            |                           |
|                            |            |                           |            |                           |
|                            |            |                           |            |                           |
|                            |            |                           |            |                           |
|                            |            |                           |            |                           |
|                            |            |                           |            |                           |
|                            |            |                           |            |                           |
|                            |            |                           |            |                           |
|                            |            |                           |            |                           |
|                            |            |                           |            |                           |
|                            |            |                           |            |                           |
|                            |            |                           |            |                           |
|                            |            |                           |            |                           |
|                            |            |                           |            |                           |
|                            |            |                           |            |                           |
|                            |            |                           |            |                           |
|                            |            |                           |            |                           |
|                            |            |                           |            |                           |
|                            |            |                           |            |                           |
|                            |            |                           |            |                           |
|                            |            |                           |            |                           |
|                            |            |                           |            |                           |
|                            |            |                           |            |                           |
|                            |            |                           |            |                           |
|                            |            |                           |            |                           |
|                            |            |                           |            |                           |
|                            |            |                           |            |                           |
|                            |            |                           |            |                           |
|                            |            |                           |            |                           |
|                            |            |                           |            |                           |

