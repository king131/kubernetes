# Deprecated！

k8s 1.13之后使用CSI，1.13版本之前还可以使用这个方式创建storageclass

### Ceph RBD动态供给存储
---
#### ceph集群的安装  
[参考ceph官方安装指导就行——点我](http://docs.ceph.com/docs/master/start/#)  
preflight:  
> 需要在node节点安装ceph-common #注意安装ceph-common的版本应与ceph server version保持一致。避免因为'feature set mismatch'导致pod无法挂载pvc

安装完成后，在ceph管理节点创建pool
```bash
#创建命令
#pgNum根据osd节点：官方建议3节点128
ceph osd pool create poolName pgNum
#此处创建kube pool
ceph osd pool create kube 128
```  
创建ceph用户：  
```bash
# admin用户在部署的时候已经创建
# 创建k8s访问ceph的用户kube 在ceph的mon或者admin节点
ceph auth get-or-create client.kube mon 'allow r' osd 'allow class-read object_prefix rbd_children, allow rwx pool=kube' -o ceph.client.kube.keyring
```
```bash
# 查看key 在ceph的mon或者admin节点
ceph auth get-key client.admin
ceph auth get-key client.kube
```
```bash
# 创建 admin secret
# CEPH_ADMIN_SECRET 替换为 client.admin 获取到的key
export CEPH_ADMIN_SECRET='AQBBAnRbSiSOFxAAEZXNMzYV6hsceccYLhzdWw=='
kubectl create secret generic ceph-secret --type="kubernetes.io/rbd" \
--from-literal=key=$CEPH_ADMIN_SECRET \
--namespace=kube-system
```

```bash
# 在 default 命名空间创建pvc用于访问ceph的 secret
# CEPH_KUBE_SECRET 替换为 client.kube 获取到的key
export CEPH_KUBE_SECRET='AQBZK3VbTN/QOBAAIYi6CRLQcVevW5HM8lunOg=='
kubectl create secret generic ceph-secret-user --type="kubernetes.io/rbd" \
--from-literal=key=$CEPH_KUBE_SECRET \
--namespace=default
```

```bash
# 查看 secret
kubectl get secret ceph-secret-user -o yaml
kubectl get secret ceph-secret -n kube-system -o yaml
```
# 此处有坑，请注意！
```bash
# 配置 StorageClass (官方写法)
# 如果使用kubeadm创建的集群 provisioner 使用如下方式
# 此处有个坑 官方的  provisioner 里没有rbd命令导致pvc处于pending状态
# 解决办法是：
# 1，自己在kube-controller-manager里安装rbd
# apt update && apt install ceph-common
# 2，使用第三方镜像


#cat StorageClass.yaml

apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast
provisioner: kubernetes.io/rbd
parameters:
  #修改为自己的ceph集群的mon节点
  monitors: 1.1.1.1:6789
  #ceph集群用户
  adminId: admin
  #上面用户的secret
  adminSecretName: ceph-secret
  adminSecretNamespace: kube-system
  #ceph pool name
  pool: kube
  #要求此用户有操作上面pool的权限
  userId: kube
  userSecretName: ceph-secret-user
  userSecretNamespace: default
  fsType: ext4
  imageFormat: "2"
  imageFeatures: "layering"
  ```
```bash
kubectl apply -f StorageClass.yaml
```
```bash
验证：
# kubectl get sc
NAME   PROVISIONER         AGE
fast   kubernetes.io/rbd   5h33m
```
创建pvc  

```bash
#cat scpvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: scpvc
spec:
  accessModes:
  - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 1Gi
  storageClassName: fast
```

应用：  
```bash
kubectl apply -f scpvc.yml
```
验证pvc
```bash
# kubectl get pvc
NAME    STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
scpvc   Bound    pvc-c15d3198-073c-48eb-adaa-5964aafb6e05   1Gi        RWO            fast           5h29m
```

创建pods

```bash
#cat scpod.yaml

apiVersion: v1
kind: Pod
metadata:
  labels:
    test: rbd-pvc-pod
  name: ceph-rbd-sc-pod1
spec:
  containers:
  - name: ceph-rbd-sc-nginx
    image: nginx
    volumeMounts:
    - name: ceph-rbd-vol1
      mountPath: /mnt
      readOnly: false
  volumes:
  - name: ceph-rbd-vol1
    persistentVolumeClaim:
      claimName: scpvc
```
应用pod配置文件：  
```bash
kubectl apply -f scpod.yaml
```
验证：  
```bash
kubectl get pods
```
