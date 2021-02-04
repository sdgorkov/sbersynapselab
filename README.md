# sbersynapselab
Репозиторий для лаб.работы


## SberSynapse Анамнез:

В рамках работы попробуем создать уникальную точку входа, воспользовавшись Symple маркетом
А затем, произвести вызов сервиса из-за пределов кластера


Попробуем развернуть "сложный интеграционный сценарий" и вызвать его.


## SberSynapse Маркет 


### Шаги:

#### Создаем конфигурацию для загрузки:
1. Логинимся в Kuber dashboard под своим SA

2. Идем по ссылке http://37.18.100.236:33333/#  ( будет ругаться на самоподписанные сертификаты - не обращайте внимания )

3. Открываем вкладку Общие компоненты выбираем там Ingress-gateway

4. Проматываем вниз специфику процесса и нажимаем "Создать"
##### заполняем параметры:
5. Называем шаблон именем своего namespace (вместе с токеном должен был быть)
6. Имя кластера любое, название проекта - ваш неймспейс, название контрольной панели  istio-system
7. Порт ингресса - любой в диапазоне, название целевого сервиса WhatToDo  порт:7788, префикс /ToDoList, протокол http

8. Нажимаем Сгенерировать (если не нажимается, вероятно что-то не заполнили)
9. Нажимаем Просмотр, разрешаем скачать файлы, если всплывет уведомление безопасности
 
#### Загружаем полученную конфигурацию:
10. В целом файл для загрузки в личный проект готов. Дальше остается соединить DevOps этап для автоматизации установки компонент, но это уже в рамках других экспериментов. Пока при необходимости можно загружать по кусочкам.
11. После загрузки конфигураций можно пробовать вызвать сервис, но для этого надо сперва загрузить сложный сервис


## SberSynapse Сложный сервис:

### Смотрим как это выглядит в целом:



### Шаги:
#### Если нужно воспроизвести в своем проекте то:

  1. Создаем Deployment сложного сервиса через ресурсы Kubernates:
```
kind: Deployment
apiVersion: apps/v1
metadata:
  name: wtdservice
  labels:
    k8s-app: wtdservice
spec:
  replicas: 1
  selector:
    matchLabels:
      k8s-app: wtdservice
  template:
    metadata:
      name: wtdservice
      creationTimestamp: null
      labels:
        k8s-app: wtdservice
    spec:
      containers:
        - name: wtdservice
          image: 'swr.ru-moscow-1.hc.sbercloud.ru/sber/wtdservice:0.8'
          resources: {}
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
          imagePullPolicy: IfNotPresent
          securityContext:
            privileged: false
```
2. Затем приложения "заглушки"
```
kind: Deployment
apiVersion: apps/v1
metadata:
  name: stub
  labels:
    k8s-app: stub
spec:
  replicas: 1
  selector:
    matchLabels:
      k8s-app: stub
  template:
    metadata:
      name: stub
      creationTimestamp: null
      labels:
        k8s-app: stub
    spec:
      containers:
        - name: stub
          image: 'swr.ru-moscow-1.hc.sbercloud.ru/sber/fstub:0.8'
          resources: {}
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
          imagePullPolicy: IfNotPresent
          securityContext:
            privileged: false
```

3. Создасть список сервисов которые вызывает сложный сервис:

```
kind: Service
apiVersion: v1
metadata:
  name: getbooks
  labels:
    k8s-app: stub
spec:
  ports:
    - name: http-12221-12221-wppg4
      protocol: TCP
      port: 12221
      targetPort: 12221
  selector:
    k8s-app: stub
```
```
kind: Service
apiVersion: v1
metadata:
  name: getfilms
  labels:
    k8s-app: stub
spec:
  ports:
    - name: http-12222-12222-wppg4
      protocol: TCP
      port: 12222
      targetPort: 12222
  selector:
    k8s-app: stub
```
```
kind: Service
apiVersion: v1
metadata:
  name: getgames
  labels:
    k8s-app: stub
spec:
  ports:
    - name: http-12223-12223-wppg4
      protocol: TCP
      port: 12223
      targetPort: 12223
  selector:
    k8s-app: stub
```
```
kind: Service
apiVersion: v1
metadata:
  name: getserials
  labels:
    k8s-app: stub
spec:
  ports:
    - name: http-12224-12224-wppg4
      protocol: TCP
      port: 12224
      targetPort: 12224
  selector:
    k8s-app: stub
 ```
    
    
4. Можно проверить вызов сервиса зайдя в терминал пода wtdservice, и выполнив там curl 0.0.0.0:12220
В ответ должно вернуться:
 Bad Film, Short Film
 Long Serial, Fun Serial
 Doom Game, Skyrim Game
4.1 Если выполнить curl 0.0.0.0:12220?read=true
вернется немного другой результат.
 Red Book, Big Book
 Long Serial, Fun Serial
 Doom Game, Skyrim Game

5. Осталось два дела: вызвать это через внешний интерфейс и внутренний интерфейсы:

Создаем внутрению ссылку:
```  
kind: Service
apiVersion: v1
metadata:
  name: wtdservice
  labels:
    k8s-app: wtdservice
spec:
  ports:
    - name: http-12220-12220-wppg4
      protocol: TCP
      port: 12220
      targetPort: 12220
  selector:
    k8s-app: wtdservice
 ```

Теперь, по идее на нее идет трафик из ingress а на ингресс можно попасть из-за пределов кластера. Для этого можно  с помощью curl вызвать сервис
curl -v http://37.18.121.127:33333
  
6. Посмотреть всю цепочку вызовов трейсинге
7. Посмотреть граф вызовов в Kiali


