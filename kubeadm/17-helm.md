#### Helm  
---
核心术语：  
  Chart： 一个helm程序包  
  Repository： Chart仓库，http服务器  
  Release: 特定的Chart部署于目标集群上的一个实例  

  chart-》config-》Release  

  程序架构：  
    helm： 客户端，管理Chart仓库，管理Chart，与Tiller交互，实现安装、查询、卸载等操作  
    Tiller：  服务端，接收helm发来的charts和config，合并生成release  
  rabc配置：  
  helm：  
    https://hub.kubeapps.com  
