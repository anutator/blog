---
title: Список всех подов сервиса k8s
share: "true"
tags:
  - k8s
---

В Kubernetes (сокращенно k8s), Service Endpoint — абстракция, которая представляет сетевое размещение сервиса. Это внутренний или внешний IP адрес, который может быть использован для доступа к сервисам приложения, работающим в кластере Kubernetes. Рассмотрим, как отобразить список всех подов, принадлежащих сервису.
## Подготовка окружения для теста
Перед запуском проверим окружение k8s. Для разворачивания тестового Kubernetes можно использовать [minikube](https://minikube.sigs.k8s.io/docs/), для продакшн используется [kubespray](https://github.com/kubernetes-sigs/kubespray), для настроек коменда [kubectl](https://kubernetes.io/docs/tasks/tools/). Создадим под Nginx.

```yaml title="nginx-pod.yml"
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace: default
  labels:
    app: nginx
spec:
  containers:
  - image: nginx:latest
    name: nginx
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"

```

Создадим Deployment для управления подами, который создаст ещё два пода (число реплик равно 2).

```yaml title="nginx-deploy.yml"
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 2
      maxUnavailable: 1
  minReadySeconds: 5
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:latest
        name: nginx
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
```

Создадим под и деплой, проверим статус подов. Всего три пода:

```bash
$ kubectl apply -f nginx-pod.yml       # создаст под с именем nginx
$ kubectl apply -f nginx-deploy.yml    # создаст 2 поды с именами, которые заканчиваются на какие-то буквы и цифры

# статус
$ kubectl get pods --show-labels

NAME                     READY   STATUS    RESTARTS   AGE     LABELS
nginx                    1/1     Running   0          5h40m   app=nginx                               # под, созданный из nginx-pod.yml
nginx-6d7c8c89f7-rrszr   1/1     Running   0          5h7m    app=nginx,pod-template-hash=6d7c8c89f7  # под, созданный из nginx-deploy.yml
nginx-6d7c8c89f7-n2n6c   1/1     Running   0          5h7m    app=nginx,pod-template-hash=6d7c8c89f7  # аналогично
```

Создадим 2 сервиса (NodePort + Cluster) для маршрутизации на поды Nginx, свяжем их с подами через `selector`. В одном файле можно создать несколько сервисов, если разделять строкой с тремя дефисами.

```yaml title="nginx-svc.yml"
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  selector:
    app: nginx
  ports:
  - name: http
    protocol: TCP
    port: 3000
    targetPort: 80

---
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
  - name: http
    protocol: TCP
    port: 8080
    targetPort: 80
```

Создадим сервисы и проверим их статус.

```bash
$ kubectl apply -f nginx-svc.yml

# проверка статуса сервиса
$ kubectl get svc -o wide
NAME             TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE     SELECTOR
kubernetes       ClusterIP   10.43.0.1      <none>        443/TCP          5h49m   <none>
nginx            ClusterIP   10.43.61.137   <none>        3000/TCP         5h41m   app=nginx
nginx-nodeport   NodePort    10.43.51.4     <none>        8080:30140/TCP   5h41m   app=nginx
```

## Список всех подов, принадлежащих сервису
Сначала посмотрим список Endpoints ткущего сервиса

```bash
$ kubectl get endpoints
NAME             ENDPOINTS                                   AGE
kubernetes       192.168.5.15:6443                           5h51m
nginx-nodeport   172.17.0.6:80,172.17.0.7:80,172.17.0.9:80   5h44m
nginx            172.17.0.6:80,172.17.0.7:80,172.17.0.9:80   5h44m
```

Как видно выше, сервис **nginx** маршрутизирует на три пода, сервис **nginx-nodeport** также маршрутизирует на три пода. Подробности:

```bash
$ kubectl get endpoints nginx -o yaml

apiVersion: v1
kind: Endpoints
metadata:
  creationTimestamp: "2023-05-21T07:15:41Z"
  name: nginx
  namespace: default
  resourceVersion: "1164"
  uid: 686d68e6-a58b-4a12-a2d4-0f4ea88b9b56
subsets:
- addresses:
  - ip: 172.17.0.6
    nodeName: colima
    targetRef:
      kind: Pod
      name: nginx
      namespace: default
      uid: e717d901-4319-4ea6-b33d-31dff7c79549
  - ip: 172.17.0.7
    nodeName: colima
    targetRef:
      kind: Pod
      name: nginx-6d7c8c89f7-n2n6c
      namespace: default
      uid: b6e7c74c-3d1f-494e-b05d-9242e367cfe3
  - ip: 172.17.0.9
    nodeName: colima
    targetRef:
      kind: Pod
      name: nginx-6d7c8c89f7-rrszr
      namespace: default
      uid: 7528bfd6-d6dd-4557-873f-6f8ed9b9ebd4
  ports:
  - name: http
    port: 80
    protocol: TCP
```

Как отсюда выудить имена всех подов:

```bash
$ kubectl get endpoints nginx -o jsonpath='{.subsets[*].addresses[*].targetRef.name}'
nginx nginx-6d7c8c89f7-n2n6c nginx-6d7c8c89f7-rrszr
```

Теперь, зная имена подов, отобразим информацию по ним:

```bash
$ kubectl get pods nginx nginx-6d7c8c89f7-n2n6c nginx-6d7c8c89f7-rrszr
```

Одна обобщающая команда для отображения подов сервиса nginx:

```bash
kubectl get pods $(kubectl get endpoints nginx -o jsonpath='{.subsets[*].addresses[*].targetRef.name}')
# или
kubectl get endpoints nginx -o jsonpath='{.subsets[*].addresses[*].targetRef.name}' | xargs kubectl get pods -o wide
```

Оригинал: https://blog.wu-boy.com/2023/05/list-pods-from-k8s-service/