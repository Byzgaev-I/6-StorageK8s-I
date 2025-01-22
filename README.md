# Домашнее задание к занятию «Хранение в K8s. Часть 1»

## Цель задания

В тестовой среде Kubernetes необходимо:
1. Создать Deployment приложения с общим хранилищем для контейнеров
2. Создать DaemonSet для чтения логов ноды

## Чеклист готовности к домашнему заданию

1. Установленное k8s-решение (MicroK8S)
2. Установленный локальный kubectl
3. Редактор YAML-файлов с подключённым git-репозиторием

## Задание 1. Создание Deployment приложения

### Манифест Deployment

Файл [deployment.yaml](task1/deployment.yaml):
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: busybox-multitool
spec:
  replicas: 1
  selector:
    matchLabels:
      app: busybox-multitool
  template:
    metadata:
      labels:
        app: busybox-multitool
    spec:
      containers:
      - name: busybox
        image: busybox
        command: 
        - /bin/sh
        - -c
        - "while true; do echo $(date) >> /shared-data/date.log; sleep 5; done"
        volumeMounts:
        - name: shared-data
          mountPath: /shared-data
      - name: multitool
        image: wbitt/network-multitool
        volumeMounts:
        - name: shared-data
          mountPath: /shared-data
      volumes:
      - name: shared-data
        emptyDir: {}
```

### Результаты

1. Создание и проверка подов:
```bash
kubectl get pods
```

![image](https://github.com/Byzgaev-I/6-StorageK8s-I/blob/main/1-1.png)


2. Проверка работы общего хранилища:
```bash
kubectl exec busybox-multitool-c8d568fd9-gd6m5 -c multitool -- tail -f /shared-data/date.log
```

![image](https://github.com/Byzgaev-I/6-StorageK8s-I/blob/main/1-2.png)


## Задание 2. Создание DaemonSet для работы с логами

### Манифест DaemonSet

Файл [daemonset.yaml](task2/daemonset.yaml):
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: multitool-ds
spec:
  selector:
    matchLabels:
      app: multitool-ds
  template:
    metadata:
      labels:
        app: multitool-ds
    spec:
      containers:
      - name: multitool
        image: wbitt/network-multitool
        securityContext:
          privileged: true
        volumeMounts:
        - name: varlog
          mountPath: /host/var/log
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
          type: Directory
```

### Результаты

1. Проверка работы DaemonSet:
```bash
kubectl get pods -l app=multitool-ds
```
```
NAME                 READY   STATUS    RESTARTS   AGE
multitool-ds-pjggl   1/1     Running   0          16m
```

2. Проверка доступа к логам:
```bash
kubectl exec $(kubectl get pod -l app=multitool-ds -o jsonpath='{.items[0].metadata.name}') -- tail /host/var/log/boot.log
```
```
[  OK  ] Started systemd-timedated.service
[  OK  ] Finished snapd.seeded.service
[  OK  ] Started snap.microk8s.daemon-containerd.service
[  OK  ] Started snap.microk8s.daemon-kubelite.service
```

## Выполненные задачи

### Задание 1
- ✅ Создан Deployment с общим томом
- ✅ Контейнер busybox успешно записывает данные
- ✅ Контейнер multitool успешно читает данные
- ✅ Подтверждена работа общего хранилища

### Задание 2
- ✅ Создан DaemonSet для работы с логами
- ✅ Pod успешно запущен на ноде
- ✅ Настроен доступ к системным логам
- ✅ Подтверждена возможность чтения логов

## Использованное окружение

- Virtual Box 7
- Debian 12
- MicroK8S v1.31.3

## Полезные команды, использованные в работе

```bash
# Создание ресурсов
kubectl apply -f task1/deployment.yaml
kubectl apply -f task2/daemonset.yaml

# Проверка статуса
kubectl get pods
kubectl get pods -l app=multitool-ds

# Просмотр данных
kubectl exec busybox-multitool-c8d568fd9-gd6m5 -c multitool -- tail -f /shared-data/date.log

# Просмотр логов
kubectl exec $(kubectl get pod -l app=multitool-ds -o jsonpath='{.items[0].metadata.name}') -- tail /host/var/log/boot.log
```
