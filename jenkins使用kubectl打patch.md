[参考简书](https://www.jianshu.com/p/f38e1767bf19)  
背景：  
jenkins版本比较老了，升级比较麻烦。没有kubernetes可用的插件。  

解决方案：  
1. 在jenkins master机器上安装kubectl，并添加$HOME/.kube/config。保证服务的连通性。  
2. 因为想在jenkins build之后直接将镜像版本号传到 patch 中，但是因为patch的格式问题，无法向patch的内容中传递参数。  
```bash  
kubectl patch deployment patch-demo --patch '{"spec": {"template": {"spec": {"containers": [{"name": "patch-demo-ctr-2","image": "redis"}]}}}}'
```
3. 考虑通过yaml文件的方式打补丁。
```bash
cat > /tmp/patch-file.yaml <<EOF
spec:
  template:
    spec:
      containers:
      - name: ${CONTAINER_NAME}
        image: ${IMAGE_URL}/${IMAGE_NAME}:${VERSION}
EOF
```

```bash
kubectl patch deployment ${DEPLOY_NAME} --type merge --patch "$(cat /tmp/patch-file.yaml)"
```
这样就可以在更新镜像到最新版本了。  
