# ДЗ #8 к лекции #10 (kubernetes-monitoring)

## Создание кластера k8s
Для выполнения ДЗ будем использовать кластер kubernetes, развернутый в GKE.
```bash
gcloud beta container --project "test-lab" clusters create "monitoring-lab" --zone "europe-west1-d" --no-enable-basic-auth --cluster-version "1.14.10-gke.17" --machine-type "n1-standard-1" --image-type "COS" --disk-type "pd-standard" --disk-size "100" --scopes "https://www.googleapis.com/auth/devstorage.read_only","https://www.googleapis.com/auth/logging.write","https://www.googleapis.com/auth/monitoring","https://www.googleapis.com/auth/servicecontrol","https://www.googleapis.com/auth/service.management.readonly","https://www.googleapis.com/auth/trace.append" --num-nodes "3" --enable-stackdriver-kubernetes --enable-ip-alias --network "projects/apigee-test-lab/global/networks/default" --subnetwork "projects/apigee-test-lab/regions/europe-west1/subnetworks/default" --default-max-pods-per-node "110" --addons HorizontalPodAutoscaling,HttpLoadBalancing --enable-autoupgrade --enable-autorepair
```

## Установка prometheus-operator
Для установки prometheus-operartor будем использовать helm3. Так как установка производится в GKE кластере, то будем публиковать сервисы в интренет с помощью nginx-ingress.

Добавляем репозиторий с charts
```bash
helm repo add stable https://kubernetes-charts.storage.googleapis.com/
```

### Установка nginx-ingress

```bash
kubectl create ns nginx-ingress
helm upgrade --install nginx-ingress stable/nginx-ingress --namespace=nginx-ingress --version=1.29.5
```
Смотрим IP адрес контроллера ingress, он нам пригодится для указания имен хостов публикуемых сервисов.
```bash
kubectl get service -n nginx-ingress
```

### Установка prometheus-operator

Выбираем последнюю версию prometheus-operator
```bash
helm search repo -l stable/prometheus-operator | head -n 2
```

Сохраняем стандартный файл values.yaml в качестве справки
```bash
helm inspect values stable/prometheus-operator --version=8.5.14 > kubernetes-monitoring/prometheus-operator.values.yaml.default
```
В файле prometheus-operator.values.yaml добавляем описание Ingresses. В качестве имен используем сервис nip.io с указанием IP адреса ingress контроллера.

Устанавливаем prometheus-operator
```bash
kubectl create ns monitoring
helm upgrade --install prometheus-operator stable/prometheus-operator --version=8.5.14 --namespace=monitoring -f kubernetes-monitoring/prometheus-operator.values.yaml
```

#### Проверка установки
По адресу grafana.*your_ip*.nip.io открывается Grafana. Стандартный пароль можно посмотреть в стандартном файле с переменными.
По адресу prometheus.*your_ip*.nip.io (на 80 порту) открывается prometheus. Alertmanager также открывается на 80 порту по адресу alertmanager.*your_ip*.nip.io
