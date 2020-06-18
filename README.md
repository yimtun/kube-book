# Kubernetes 暴力拆解实战教程



# 稳定说明

Kubernetes 是谷歌开源的容器集群管理系统，是 Google 多年大规模容器管理技术 Borg 的开源版本，也是 CNCF 最重要的项目之一，主要功能包括：

- 基于容器的应用部署、维护和滚动升级
- 负载均衡和服务发现
- 跨机器和跨地区的集群调度
- 自动伸缩
- 无状态服务和有状态服务
- 广泛的 Volume 支持
- 插件机制保证扩展性



本文档通过观察kubeadm 创建的集群一步步分析其架构和部署规范细节,最后将所有组件都拆解出来，甚至包括CNI 插件也拆解为直接运行在操作系统上而非运行在容器里





# 实战演练

- 准备工作：
  - 第一式  [创建第一个集群](./first-cluster/first-cluster.md)
  
- 基础拆解: 

  - 第一式  [将etcd独立部署](./etcd/etcd.md)

































