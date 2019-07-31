### k8s
---
#### 目标
实现服务的快速部署、服务的高可用、服务运行时的监控。  
##### 我们有什么需求？
1， 敏捷开发，快速部署  
2， 服务之间的通信  
3， 服务进程死掉或者不可读时重启    
4， 快速括缩容  
5， 快速回滚  
6， 配置的集中管理以及私密信息的管理   
7， 负载均衡 (nginx个性化的一些配置)   
8， 数据持久化  
9， 日志可视化  
10，服务资源的监控  
11，对新手友好  
1,6,9,10
##### 敏捷开发，快速部署  
CI／CD
##### 服务之间通信  
1， Pod之间通信  
2， service通信  
3， ingress通信  
##### 服务进程死掉或者不可读时重启  
1， livenessProbe  
2， readinessProbe  
##### 快速括缩容
kubectl scale deployment/name --replicas=1  
##### 快速回滚  
kubectl rollout history deployment nginx-deployment  
kubectl rollout undo deployment nginx-deployment --to-revision=2  
##### 配置的集中管理以及私密信息的管理    
1, configMap  
2, Secret  
##### 负载均衡  
##### 数据持久化  
1， pv  
2， pvc  
3， storageClass  
##### 日志可视化
##### 服务资源的监控   
##### 对新手友好
rancher
