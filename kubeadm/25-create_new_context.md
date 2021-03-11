# 如何为用户单独创建kubeconfig
为了便于集群管理，为用户提供更精确的权限控制，我们可以在给用户单独的`kubeconfig`。
> Note: 假设我们创建的用户ID为：testprivate   

### 创建证书
1. 创建私钥

    ```bash
    (umask 077; openssl genrsa -out testprivate.key 2048)
    ```
1. 生成证书签署请求

    ```bash
    openssl req -new -key testprivate.key -out testprivate.csr -subj "/CN=testprivate"
    ```

1. 签署请求
    ```bash
    openssl x509 -req -in testprivate.csr  -CA ./ca.crt -CAkey ./ca.key -CAcreateserial -out testprivate.crt -days 3650
    ```

1. 查看证书信息

    ```bash
    openssl x509 -in testprivate.crt -text -noout
    ```

### 创建上下文
1. 创建 cluster

    ```bash
    kubectl config set-cluster  CLUSTERNAME --server=https://1.2.3.4:8443  --certificate-authority=path/to/certificate/ca.crt --embed-certs=true
    ```
1. 创建 user

    ```bash
    kubectl config set-credentials  testprivate --client-certificate=./testprivate.crt --client-key=./testprivate.key  --embed-certs=true
    ```
1. 设置上下文

    ```bash
    kubectl config set-context username@clustername --cluster=CLusterName --user=user_nickname --namespaces=NameSpace
    ```

1. 切换上下文

    ```bash
    kubectl use-context username@clustername
    ```
### 创建RBAC

1. 创建role
    ```bash
    kubectl create role pods-reader --verb=get,list,watch --resource=pods --dry-run=client  -o yaml  > testrole.yaml
    修改`namespace`的值为限制用户权限的namespace
    ```
    ```yaml
    apiVersion: rbac.authorization.k8s.io/v1
    kind: Role
    metadata:
      namespace: default
      name: pods-reader
    rules:
    - apiGroups:
      - ""
      resources:
      - pods
      verbs:
      - get
      - list
      - watch
    ```

    ```bash
    kubectl apply -f testrole.yaml
    ```
1. 创建rolebinding

    ```bash
    kubectl create rolebinding BindingName --role=pods-reader --user=testprivate
    ```


### teardown

  ```bash
  kubectl delete rolebinding testrolebinding  
  kubectl delete pods-reader
  kubectl delete-context test-context
  kubectl delete-cluster test-cluster
  ```
