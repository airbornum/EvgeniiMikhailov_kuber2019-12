# EvgeniiMikhailov_platform
EvgeniiMikhailov Platform repository

## ДЗ #1 (к лекции #2)
### Разберитесь почему все pod в namespace kube-system восстановились после удаления.
**core-dns** запускается в deployment, поэтому при удаление подов replicaset восставливает поды.

**kube-proxy** управляется и создается Daemonset.

**kube-apiserver**, **etcd**, **kube-controller-manager**, **kube-scheduler** являются статическими подами, их запускает kubelet ноды, так как их описание находится в папке /etc/kubernetes/manifests/
В параметрах запуска kubelet (в файле /etc/systemd/system/kubelet.service.d/10-kubeadm.conf) указан путь к директории из которой создавать статические поды --pod-manifest-path /etc/kubernetes/manifests

**kube-addon-manager** и **storage-provisioner** являются аддонами minikube. Они управляются с помощью команды minikube addons

### Создать pod
Создан Docker image на основе стандартного image nginx. Pod использует volume emptyDir который монтируется в /app. Nginx отдает содержимое этой директории. Файл index.html скачивается с помощью initContainer.
Для проверки работы поды можно выполнить команду:
```
kubectl port-forward --address 0.0.0.0 pod/web 8000:8000
```

### Hipster shop
Создать файл описания пода можно с помощью команды
```
kubectl run frontend --image evgeniim/hipster-frontend --restart=Never --dry-run -o yaml > frontend-pod.yaml
```
Под не запускался из-за отсутсвия переменных окружений.

## ДЗ #2 (к лекуии #3)
### ReplicaSet

Создано описание rs для hipster-frontend. Replicaset не умеет обновлять поды если изменился шаблон (template). Шаблон применяется для новых контейнеров.

### Deployment

Создан deployment дл hipster-paymentservice. Протестировано обновление образа.

#### Задание со звездочкой
Статус rs после изменения deployment (для blue-green). 
```
NAME                        DESIRED   CURRENT   READY   AGE
paymentservice-74fb5b848d   3         3         3       10s
paymentservice-7669454bd5   3         0         0       1s
paymentservice-7669454bd5   3         0         0       1s
paymentservice-7669454bd5   3         3         0       1s
paymentservice-7669454bd5   3         3         1       3s
paymentservice-7669454bd5   3         3         2       4s
paymentservice-74fb5b848d   2         3         3       14s
paymentservice-7669454bd5   3         3         3       6s
paymentservice-74fb5b848d   2         3         3       17s
paymentservice-74fb5b848d   2         2         2       18s
paymentservice-74fb5b848d   0         2         2       18s
paymentservice-74fb5b848d   0         2         2       20s
paymentservice-74fb5b848d   0         0         0       20s
```

Статус rs после измениния deployment (для rolling update).
```
NAME                        DESIRED   CURRENT   READY   AGE
paymentservice-74fb5b848d   3         3         3       17s
paymentservice-7669454bd5   0         0         0       0s
paymentservice-7669454bd5   0         0         0       0s
paymentservice-74fb5b848d   2         3         3       17s
paymentservice-74fb5b848d   2         3         3       17s
paymentservice-7669454bd5   1         0         0       0s
paymentservice-74fb5b848d   2         2         2       17s
paymentservice-7669454bd5   1         0         0       0s
paymentservice-7669454bd5   1         1         0       0s
paymentservice-7669454bd5   1         1         1       3s
paymentservice-74fb5b848d   1         2         2       20s
paymentservice-7669454bd5   2         1         1       3s
paymentservice-74fb5b848d   1         2         2       20s
paymentservice-74fb5b848d   1         1         1       21s
paymentservice-7669454bd5   2         1         1       4s
paymentservice-7669454bd5   2         2         1       4s
paymentservice-7669454bd5   2         2         2       6s
paymentservice-74fb5b848d   0         1         1       23s
paymentservice-7669454bd5   3         2         2       6s
paymentservice-74fb5b848d   0         1         1       23s
paymentservice-74fb5b848d   0         0         0       24s
paymentservice-7669454bd5   3         2         2       7s
paymentservice-7669454bd5   3         3         2       7s
paymentservice-7669454bd5   3         3         3       10s
```

### Probes

Создан deployment для hipster-frontend с readiness probe.

### DaemonSet

Создан daemonset с node-exporter, который запускается на всех нодах кластера
