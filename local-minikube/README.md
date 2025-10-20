# Как запустить сайт для локальной разработки
Перед развертыванием установите нужные инструменты:
- minikube и Hypervisor к [нему](https://kubernetes.io/ru/docs/tasks/tools/install-minikube/)
- [kubectl](https://kubernetes.io/docs/tasks/tools/)
- [helm](https://helm.sh/docs/intro/install/)
## Запуск minikube
```shell
minikube start
```
## Загрузка образа в minikube
Создайте образ из docker-compose.yml:
```shell
docker-compose image build ./backend_main_django/ -t django_app
```
Посмотрите image 
```shell
docker images
```
и загрузите образа в minikube:
```shell
minikube image load [name]
```
## Установка секретов
Создать файл kuber_secrets.yaml с переменными окружения:

```yaml
apiVersion : v1
kind : Secret
metadata :
  name : django-secrets
stringData :
  DATABASE_URL : ''
  SECRET_KEY : ''
```
Подставить серкреты SECRET_KEY и DATABASE_URL в файл kuber_secrets.yaml
Запустить:

```shell
kubectl apply -f kuber/kuber_secrets.yaml 
```

## Запуск приложения
Запустить:
```shell
kubectl apply -f kuber/kuber_deployment.yaml 
```
Применить миграции:
```shell
kubectl apply -f kuber/migrate_job.yaml 
```

Установить cronjob по удалению сессий:
```shell
kubectl apply -f kuber/clearsession.yaml 
```
Установить services:
```shell
kubectl apply -f kuber/services.yaml 
```

Установить ingress:
```shell
kubectl apply -f kuber/ingress.yaml 
```
 Установить постгресс через хелм в кластере или запустить в отдельном контейнере (подключаться через внутренний айпи)

 Установка через helm

 ```shell
 helm install my-release oci://registry-1.docker.io/bitnamicharts/postgresql
 ```

 Подключитесь через psql по инструкции после этой команды и создайте базу и пользователя

 После чего в секретах укажите
 подключение в формате  postgres://USER:PASSWORD@HOST:PORT/NAME
 HOST можно взять в информации после запуска через helm в этих строках
 ```shell
 PostgreSQL can be accessed via port 5432 on the following DNS names from within your cluster:

    my-release-postgresql.default.svc.cluster.local - Read/Write connection

 ```