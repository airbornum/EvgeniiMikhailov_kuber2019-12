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
