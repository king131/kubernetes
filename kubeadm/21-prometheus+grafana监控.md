### 监控

1, prometheus 安装  
参考官方安装文档：  
https://github.com/kubernetes/kubernetes/tree/a5fcaa87f6bd06af87c9e10eff2a8c539f571cf8/cluster/addons/prometheus
国内安装其中有几个镜像无法拉取，在hub.docker.com上找一下，替换掉就行了。 有几个地方需要存储，用到了storageclass，参考一下“20-ceph存储.md”的实现。  

2，grafana 安装

pvc+deployment+service
```bash
cat grafana.yaml

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: grafana-pvc
  namespace: kube-system
spec:
  storageClassName: fast
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: grafana-deployment
  namespace: kube-system
  labels:
    app: grafana-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: grafana-pod
  template:
    metadata:
      labels:
        app: grafana-pod
    spec:
      initContainers:
      - name: init-chown-data
        image: busybox
        imagePullPolicy: IfNotPresent
        command: ["chown","-R","472:472","/var/lib/grafana"]
        volumeMounts:
        - name: grafana-data
          mountPath: /var/lib/grafana
          subPath: ""
      containers:
      - name: grafana-pod
        image: grafana/grafana:6.2.2
        imagePullPolicy: IfNotPresent
        ports:
        - name: grafana
          containerPort: 3000
        env:
        - name: GF_SECURITY_ADMIN_USER
          value: admin
        - name: GF_SECURITY_ADMIN_PASSWORD
          value: admin
        readinessProbe:
          httpGet:
            path: /api/health
            port: 3000
            scheme: HTTP
          initialDelaySeconds: 60
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 30
        livenessProbe:
          httpGet:
            path: /api/health
            port: 3000
            scheme: HTTP
          initialDelaySeconds: 60
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 30
        resources:
          limits:
            cpu: 500m
            memory: 1Gi
          requests:
            cpu: 200m
            memory: 500Mi
        volumeMounts:
        - name: grafana-data
          mountPath: /var/lib/grafana
          subPath: ""
        securityContext:
          runAsUser: 472
      volumes:
      - name: grafana-data
        persistentVolumeClaim:
          claimName: grafana-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: grafana-service
  namespace: kube-system
  labels:
    app: grafana-service
spec:
  type: NodePort
  ports:
  - port: 3000
    targetPort: 3000
  selector:
    app: grafana-pod
```

3, 登录grafana，添加数据源： http://prometheus.kube-system.svc.cluster.local:9090  
4, 添加dashboard  
