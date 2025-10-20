# Развертывание в Yandex Cloud
Убедитесь что вы получили все [доступы](https://console.yandex.cloud/)


![Выглядит так](https://github.com/Fergoth/k8s-test-django/raw/main/github_images/image.png)
* Интрукции по [настройке](https://disk.yandex.ru/i/QDZGXUP7aG3KOw)

после настройки убедитесь что кластер доступен
```shell
$ kubectl cluster-info
Kubernetes control plane is running at https://84.201.152.191
CoreDNS is running at https://84.201.152.191/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```
## Как подготовить dev окружение
Запустите Lens, [Инструкция по установке](https://docs.k8slens.dev/getting-started/install-lens/)
Вы должны увидеть кластер

![](https://github.com/Fergoth/k8s-test-django/raw/main/github_images/image-1.png)

### [Получите ssl сертификат](https://yandex.cloud/ru/docs/managed-postgresql/operations/connect) 
Добавьте новый секрет в свой namespace

В поле value вставьте значение своего сертификата

![alt text](https://github.com/Fergoth/k8s-test-django/raw/main/github_images/image-2.png)

Пример запуска простого контейнера с подключением сертификата
```yaml
apiVersion: v1
kind: Pod
metadata:
  namespace: [ВАШ namespace]
  name: static-web
  labels:
    role: myrole
spec:
  volumes:
    - name: secret-volume
      secret: 
        secretName: [Имя секрета которое указывали выше]
        defaultMode: 429
  containers:
    - name: ubuntu
      image: ubuntu:latest
      command: ["/bin/sleep", "infinity"]
      volumeMounts: 
      - name : secret-volume
        readOnly: true
        mountPath: "root/.postgresql/"
```
Подключитесь из терминала контейнера к базе данных чтобы проверить сертификат:
```shell
psql -h HOST -p PORT -U USER -d NAME
```
Все данные для подключения лежат в секрете postgres

### Добавьте Секрет через Lens c именем django-secrets
DATABASE_URL : postgres://[Имя пользователя]:[Пароль ]@[Хост ]:[Порт ]/[Имя бд]  (все значения лежат в secrete postgres)
SECRET_KEY : [Ваш супер секретный ключ]
ALLOWED_HOSTS : [Ваш домен]
### Создайте сервис для приложения django

```yaml
apiVersion: v1
kind: Service
metadata:
  name: django-service
  namespace: [ВАШ namespace]
spec:
  type: ClusterIP
  selector:
    app: django
  ports:
    - port: 8000
      targetPort: 80
```

### Поменяйте в Config Maps настройки Nginx и перезапустите deployment nginx

```
server {
            listen       80;
            server_name  localhost;
            location / {
                proxy_pass http://django-service:8000;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Proto $scheme;
            }
        }
```

### Зайдите в под примините миграции и создайте суперпользователя
```shell
python manage.py migrate
python manage.py createsuperuser
```

После чего зайдите в админку по вашему выделеному домену 
Например https://edu-andrey-trefilov.yc-sirius-dev.pelid.team/

Для запуска прод версии поменяйте DEBUG на 0

Пример выделенной инфраструктуры
https://sirius-env-registry.website.yandexcloud.net/edu-andrey-trefilov.html