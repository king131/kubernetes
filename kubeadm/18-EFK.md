### EFK  
---
### 日志采集的几种形式  
- 原生方式：使用 kubectl logs 直接在查看本地保留的日志，或者通过docker engine的 log driver 把日志重定向到文件、syslog、fluentd等系统中。  
![](images/loggin-node-level.png)
- DaemonSet方式：在K8S的每个node上部署日志agent，由agent采集所有容器的日志到服务端。  
![](images/loggin-with-node-agent.png)
- Sidecar方式：一个POD中运行一个sidecar的日志agent容器，用于采集该POD主容器产生的日志。  
![](images/loggin-with-sidecar-agent.png)


#### EFK
E： elasticsearch  
L： logstash  
F： filebeat  
K： kibana  

k8s集群中的EFK指的是是elasticsearch-fluentd-kibana     

helm 部署  
参考官方文档部署（有坑）：
https://github.com/kubernetes/kubernetes/tree/master/cluster/addons/fluentd-elasticsearch

### 坑：
按照上述文档部署完成后，kibana并不能访问，报错信息如下
```html
{"statusCode":404,"error":"Not Found","message":"Not Found"}
```
解决方案：  
修改上述路径中 kibana-deployment.yaml 文件：
删除或者注释掉：
```html
- name: SERVER_BASEPATH
  value: /api/v1/namespaces/kube-system/services/kibana-logging/proxy
```
