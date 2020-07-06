# Kubernetes 暴力拆解实战教程



# 文档说明

Kubernetes 是谷歌开源的容器集群管理系统，是 Google 多年大规模容器管理技术 Borg 的开源版本，也是 CNCF 最重要的项目之一。

- 基于容器的应用部署、维护和滚动升级
- 负载均衡和服务发现
- 无状态服务和有状态服务
- 插件机制保证扩展性



本文档通过观察kubeadm 创建的集群一步步分析其架构和部署规范细节,最后将所有组件都拆解出来，甚至包括CNI 插件也拆解为直接运行在操作系统上而非运行在容器里。



在策划这门课程之前 我没有接触过kubeadm  当时思考课程的切入点，觉得这个kubeadm 是一个不错的切人方式

文档使用

# 实战演练

- 准备工作：
  
  - 第一式  [创建第一个集群](./first-cluster/first-cluster.md)
  
- 基础拆解: 

  - 第二式  [将etcd独立部署](./etcd/etcd-add.md)
  
  - 第三式  [让kubeadm使用自己创建的证书](./certs/2-暴力拆解第二式-使用自己的证书.md) 
      - 1  [nginx证书示例](./certs/https-双向认证.md)
     - 2  [让kubeadm使用手工创建的证书](./certs/2-暴力拆解第二式-使用自己的证书.md)
     
  - 第四式  添加一个node节点 并分析添加过程
      - 1 [指定bootstrap tokens创建一个集群](./kubelet/define-token-install-cluster.md)
      - 2 [查看kubeadm init 输出信息](./kubelet/output.md)
      - 3 [使用kubeadm join 添加一个node](./kubelet/kubeadm-join-node.md)
      - 4  [观察kubeadm设置的bootstrap token权限](./kubelet/kubelet.md)
      - 5  [add node 手动](./kubelet/add-node.md)
      
  - 第五式  拆解kube-proxy
    
      - [拆解kube-proxy](./kube-proxy/daemonset-kube-proxy.md)

  - 第六式 kube-apiserver
    
      - [kube-apiserver](./kube-apiserver/kube-apiserver.md)
  - 第七式 kube-controller-manager
    
      - [kube-controller-manager](./kube-controller-manager/kube-controller-manager.md)
  - 第八式 kube-scheduler
    
      - [kube-scheduler](./kube-scheduler/kube-scheduler.md)
    
  - 第九式 拆解CNI
    
      - flanneld
      
          - [查看当前flaneld信息](./cni/flanneld.md)
          - [将flannel作为cni插件直接运行在操作系统上](./cni/run-flanneld.md)
      
  - 补充  [kube-apiserver](./kube-api/kube-api.md) 
  


- 必要理论+实践: 
  - 1  架构概览
      - [架构概览](./arch/arch.md)

  - 2  taint and toleration
      - [环境部署](./taints-tolerations/etcd-certs.md)
      - [taints and tolerations](./taints-tolerations/taints-tolerations.md)
  - 3  名词简要解释
      - [words](./word/word.md)

   



> http://www.howtopronounce.cc





适配docker版本

> https://v1-18.docs.kubernetes.io/zh/docs/setup/production-environment/container-runtimes/#docker



> https://v1-18.docs.kubernetes.io/docs/setup/production-environment/container-runtimes/#docker



> https://kubernetes.io/docs/setup/production-environment/container-runtimes/#docker

















