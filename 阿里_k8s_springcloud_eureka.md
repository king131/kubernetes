###如何在阿里云k8s集群上部署eureka  
参考文档：[链接](https://yq.aliyun.com/articles/596889?spm=a2c4e.11153940.blogcont596892.18.1fa775dbOi1juk)  
按照这个文档操作，有几个坑：  
1、 如果不做修改的话，svc的name会非常长，超过100个字符，无法申请到slb的ip，需手动调整名称，可以通过：
```
kubectl edit svc SVC_NAME
```
来修改svc的名称。  
2、注册上来的是hostname。需要修改测试微服务的application.yml
内容如下：
```
server:
  port: ${SERVER_PORT:5678}

feign:
  hystrix:
    enabled: true

eureka:
  client:
    enabled: true
    serviceUrl:
      defaultZone: ${EUREKA_SERVER_ADDRESS:http://xxxxxxxxx.default.svc.cluster.local:8761/eureka/}
  instance:
    #hostname: ${HOSTNAME:localhost}
    instance-id: ${spring.cloud.client.ipAddress}:${server.port}
    preferIpAddress: true

#cloud:
#  conablefig:
#    failFast: true
#    discovery:
#      end: true
```
xxxxxxxxx为你修改的svc的name。之后注册上来的就是pod的ip了。
