###在阿里云的k8s集群上部署apollo
---
apollo：[链接](https://github.com/ctripcorp/apollo/wiki／%E5%88%86%E5%B8%83%E5%BC%8F%E9%83%A8%E7%BD%B2%E6%8C%87%E5%8D%97)  
参考其中的kubernetes部署方式： [链接](https://github.com/qct/apollo-helm)  

将此chart克隆至本地(请先阅读下面的NOTE)：
```bash
$ git clone https://github.com/qct/apollo-helm.git
$ cd apollo-helm
$ helm install --name apollo .
```

####NOTE  
1、注意此处的 helm install --name apollo . 名字一定要为apollo，因为里面有涉及访问数据库的时候，路径指定的为apollo-proconfigservicemysql  
~~2、修改values.yml 中关于"xxxconfigservice.mysql"中的storageClass，修改为：  
```bash  
storageClass: "alicloud-disk-efficiency"  
size: 20Gi  
```

2、修改hack/run.sh 中存储类，和你需要安装的namespace。
阿里提供的可用存储类：[链接](https://help.aliyun.com/document_detail/86612.html?spm=a2c4g.11186623.6.654.2a164542oNwhNl)  

```
alicloud-disk-common：普通云盘。
alicloud-disk-efficiency：高效云盘。
alicloud-disk-ssd：SSD云盘。
alicloud-disk-available：提供高可用选项，先试图创建高效云盘；如果相应可用区的高效云盘资源售尽，再试图创建SSD盘；如果SSD售尽，则试图创建普通云盘
```  
并且这些存储类对大小有要求：  
```
说明 对创建的云盘容量有如下要求：
普通云盘：最小5Gi
高效云盘：最小20Gi
SSD云盘：最小20Gi
```


稍等几分钟，服务就可以起来了 

3、服务起来之后建议都看一下日志
```bash
kubectl logs ${POD_NAME} -n ${NAMESPACE}
```
可能会存在configservice向metadataservice注册不成功。需要将要metadataservice的解析改到127.0.0.1


