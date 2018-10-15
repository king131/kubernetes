### kubernetes 学习过程和脚本  
#####(本文档完全来源于mageedu)
***
#### k8s 拓扑图
![image](https://github.com/king131/kubernetes/blob/master/kubeadm/images/k8stuopu.png)




master/node  
  master: API server,Scheduler Server,   Controller-Manager  
  node: kubelet,docker,kuber-proxy  

Pod  
  label: key=value  
  Label Selector  

Pod:  
  自主式Pod  
  控制器管理的Pod  
    ReplicationController  
    Replicaset  
    Deployment  
    StatefulSet  
    DaemonSet  
    Job,Cronjob  
HPA：  
  HorizontalPodAutoscaler  

Addons:  

网络:  
  同一个Pod内的多个容器间：lo  
  各Pod之间通信：overlay Network 叠加网络  
  Pod与service之间通信：iptables或者ipvs
#### k8s 网络图
![image](https://github.com/king131/kubernetes/blob/master/kubeadm/images/k8sk8s_network.png)  

CNI：  
  flannel：网络配置  
  calico：网络配置，网络策略  
