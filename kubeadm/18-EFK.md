#### EFK  
---
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
