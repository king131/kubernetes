[metric-server简单部署](https://github.com/kubernetes-incubator/metrics-server/blob/master/deploy/1.8%2B/metrics-server-deployment.yaml)  

按照这个文档部署之后发现pod无法解析到相应的主机，需要修改deloyment里的command：  
```bash
   containers:
      - name: metrics-server
        image: k8s.gcr.io/metrics-server-amd64:v0.3.1
        command:
        - /metrics-server
        - --metric-resolution=30s
        - --kubelet-insecure-tls
        - --kubelet-preferred-address-types=InternalIP,Hostname,InternalDNS,ExternalDNS,ExternalIP

```


[参考文档](http://blog.51cto.com/newfly/2294112?source=dra)
