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

## ДЗ #2 (к лекции #3)
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

## ДЗ #3 (к лекции #4)
Знакомство с RBAC

### task01

Созданы 2 serviceaccount bob и dave.  Bob дана ClusterRole admin.

Проверить корректность настройки можно с помощью команд
```bash
kubectl get pods -n kube-system --as system:serviceaccount:default:bob
kubectl get pods -n kube-system --as system:serviceaccount:default:dave
```
При этом команда, запущенная от bob, выполнится усспешно, а от имени dave с ошибкой Forbidden.

### task02

Создан отдельный namespace prometheus и роль с возможностью выполнять get,list,wathc в отношении pods всего кластера. Роль применена ко всем аккаунтам namespace prometheus.

### task03

Создан отдельный namespace dev с двумя serviceaccounts (jane и ken). Jane назначена ClusterRole admin с помощью RoleBinding. Ken назначена ClusterRole view с помощью RoleBinding.

## ДЗ #4 (к лекции #5)

### Deployment
#### Тестирование разных вариантов maxSurge и maxUnavailable. 
##### maxUnavailable = **0**; maxSurge = **100%**

Вначале создаются новые поды. По готовности нового пода удаляется старый под.

##### maxUnavailable = **0**; maxSurge = **0**

Не валидная спецификация. Оба этих поля не могут равняться нулю однвоременно.

##### maxUnavailable = **100%**; maxSurge = **100%**

Одновременно создаются новые поды и удаляются старые. Данная стратегия может привести к простою сервиса.

#####  maxUnavailable = **100%**; maxSurge = **0**

Похоже на стратегию Recreate. Вначале старые поды удаляются, после удаления создаются новые поды.
В отличие от Recreate, который дожидается полного удаления старых подов, при данных значениях maxUnavailable  и maxSurge новые поды начинаю создаваться сразу после изменения статуса старых подов на terminating.

### Service

Создан сервис ClusterIP

### Metallb

Сервис web доступен через metallb.

#### Задание со звездочкой

Создан metallb serice для dns.
Проверка
```
kubectl apply -f kubernetes-networks/coredns/
nslookup web-svc-lb.default.svc.cluster.local 172.17.255.10
```

### Ingress

Опубликован сервис через ingress

#### Задание со звездочкой (dashboard)

Манифест Ingress находится в папке dashboard. Сам dashboard находится не в работоспособном состоянии, при попытке его открыть вижу пустую страницу. Точно такой же результат можно наблюдать если подключиться к его сервису напрямую.

#### Задание со звездочкой (canary)

Создано 2 deployment с nginx вместе с serice и configmap с конфигурацией nginx чтобы отличать различные версии deployment по возвращаемым nginx данным. Все манифесты применяются в отдельный namespace canary.

В качестве теста реализовал перенапрелвение 50% травфика на новые поды. В процессе реализации столкнулся с проблемой, что трафик не балансировался. Логи ingress controller подсказали, что проблема в отсутсвие имени хоста в настройках ingress (cannot merge alternative backend canary-deployment-2-80 into hostname  that does not exist). Добавление опции host решило проблему.
