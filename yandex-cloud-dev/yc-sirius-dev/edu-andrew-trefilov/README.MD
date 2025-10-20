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

[Получите ssl сертификат](https://yandex.cloud/ru/docs/managed-postgresql/operations/connect) 
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
