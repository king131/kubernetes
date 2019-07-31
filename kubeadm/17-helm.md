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

1,获取helm安装包  
https://github.com/helm/helm/releases  
解压后，将helm和tiller放在PATH路径下  

2，install helm with RBAC:  

```bash
cat rbac-config.yaml

apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: tiller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: tiller
    namespace: kube-system

```

```bash
kubectl apply -f rbac-config.yaml
```

3，初始化helm
```bash
helm init --history-max 200  --service-account tiller
```

4, 验证
```bash
kubectl get pods -n kube-system
```
5，添加阿里云helm 源
```bash
helm repo add ali-incubator 	https://aliacs-app-catalog.oss-cn-hangzhou.aliyuncs.com/charts-incubator/  
helm repo add ali-stable	https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts  

#更新repo
helm repo update
#搜索软件
helm search mysql
```
