dashboard:
  1、部署
    kubectl apply -f
  2、将service改为NodePort
    kubectl patch svc kubernetes-dashboard -p '{"spec":{"type":"NodePort"}}' -n kube-system
  3、认证
    认证时账号必须为ServiceAccount：被dashboard pod拿来由kubernetes进行认证：

    token：
      a，创建ServiceAccount，根据其管理目标，使用rolebinding或clusterbinding绑定至合理的role或者clusterrole
      b，获取到此SA的secret，查看secert的详细信息，其中就有token
    kubeconfig： 把SA的token封装为kubeconfig文件
      a，创建SA。根据其管理的目标。使用rolebinding或clusterbinding绑定至合理的role或者clusterrole
      b，kubectl get secret xxx -o jsonpath={.data.token}|base64 -d

    c、生成kubeconfig
      kubectl config set-cluster
      kubectl config set-credentials NAME
      kubectl config set-context
      kubectl config use-context

dashboard 暂时无法获取pod的cpu和内容信息。需要heapster部署完成之后才能获取到。


kubernetes集群管理方式：
  1、命令式 create，run， expost，delete，edit
  2、声明式配置文件： apply -f ／path／config ，patch
  3、命令式配置文件： create -f ／path／config
