### helm-upgrade-configmap-simple
Helm更新使用ConfigMap的例子

通常情况下，configmaps 或 secrets 被作为配置文件注入容器中。根据应用程序的不同，可能需要重新启动才能使用后续更新 helm upgrade，但如果 deployment spece 本身未更改，则应用程序会一直以旧配置运行，导致 deployment 不一致。

该 sha256sum 函数可用于确保在一个文件发生更改时更新 deployment 的注释部分：
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx-deployment
  template:
    metadata:
      labels:
        app: nginx-deployment
      annotations:
        checksum/config: {{ include (print $.Template.BasePath "/configmap.yaml") . | sha256sum }}
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - name: config-volume
          mountPath: /etc/nginx/conf.d/
      volumes:
        - name: config-volume
          configMap:
            name: nginx-cm
            
```

使用
```
# install
helm install nginx-cm helm-cm1-master/

# 更改default.conf

# upgrade
helm upgrade --recreate-pods  nginx-cm helm-cm1-master/
```
