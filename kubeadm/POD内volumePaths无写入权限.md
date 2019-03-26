Docker容器中使用非root用户启动程序后，对挂载路径无写入权限。
以POD为例：  
在.spec.containers 下添加：
securityContext：
  runAsUser: UID
  fsGroup: GID 
