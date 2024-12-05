University: [ITMO University](https://itmo.ru/ru/)

Faculty: [FICT](https://fict.itmo.ru)

Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)

Year: 2024/2025

Group: K4112c

Author: Gomzyakov Alexander Nikolaevich

Lab: Lab3

Date of create: 04.12.2024

Date of finished: 05.12.2024

# Сертификаты и "секреты" в Minikube, безопасное хранение данных. #

## Цель работы: ##
Познакомиться с сертификатами и "секретами" в Minikube, правилами безопасного хранения данных в Minikube. 

## Ход работы: ##

### Создание ConfigMap ###

Первым делом создадим configMap с переменными REACT_APP_USERNAME и REACT_APP_COMPANY_NAME.

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: itdt-contained-frontend-configmap
  labels:
    name: itdt-contained-frontend-configmap
data:
  REACT_APP_USERNAME: Alexander Gomzyakov
  REACT_APP_COMPANY_NAME: ITMO
```
Теперь применим его с помощью следующей команды: 

![изображение](https://github.com/user-attachments/assets/5c5459c0-689c-4c80-8c67-1f25a5d95fd2)

### Создание ReplicaSet ###

Теперь создадим replicaSet с двумя репликами контейнера и используя раннее созданный configMap передадим параметры REACT_APP_USERNAME и REACT_APP_COMPANY_NAME.

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: itdt-contained-frontend
  labels:
    app: lab3-sevice 
spec:
  selector:
    matchLabels:
      app: lab3-service
  replicas: 2
  template:
    metadata:
      labels:
        app: lab3-service
    spec:
      containers:
        - name: itdt-contained-frontend
          image: ifilyaninitmo/itdt-contained-frontend:master
          env:
            - name: REACT_APP_USERNAME
              valueFrom:
                configMapKeyRef:
                  name: itdt-contained-frontend-configmap
                  key: REACT_APP_USERNAME
            - name: REACT_APP_COMPANY_NAME
              valueFrom:
                configMapKeyRef:
                  name: itdt-contained-frontend-configmap
                  key: REACT_APP_COMPANY_NAME
```

Применение ReplicaSet:

![изображение](https://github.com/user-attachments/assets/937cb88f-d029-4674-95e4-eafe8adbdeeb)


### Ingress enable и сертификат TLS ###

Первым делом включим addon ingress: 

![изображение](https://github.com/user-attachments/assets/85e02214-343d-4dd1-81f4-5835ca894307)

Далее нужно сгенировать сертификат TLS. Для этого возпользуемся следующей командой: 

![изображение](https://github.com/user-attachments/assets/871d6d56-93e6-4cb2-8386-c12bf9ac6089)

Теперь добавим созданный сертификат в качестве secret:

![image_2024-12-05_16-15-50](https://github.com/user-attachments/assets/6989b37a-9cb9-4846-a011-ec43d2c2609e)


### Создание ingress и service ###

Создадим конфигурацию файла service.yml: 

```
apiVersion: v1
kind: Service
metadata:
  name: lab3-service
spec:
  type: ClusterIP
  selector:
    app: lab3-service
  ports:
  - port: 3000
    targetPort: 3000
```

Создадим манифест Ingress в котором укажем ранее импортированный сертификат и FQDN по которому в будущем будет производится заход:

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: lab3-ingress
  labels:
    name: lab3-ingress
spec:
  tls:
    - hosts: 
      - gomzyakov.com
      secretName: secret-tls
  rules:
  - host: gomzyakov.com
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: lab3-service
            port: 
              number: 3000
```

### Настройка hosts ###

Перейдем по следующему пути:
```
sudo nano /etc/hosts
```

В этом файле пропишем FQDN и IP ingress. 

![изображение](https://github.com/user-attachments/assets/d2d18a55-e87a-4860-b727-82900fdaa17e)

### Проверка работоспособности ###

Перейдем в брузере по адресу gomzyakov.com

![изображение](https://github.com/user-attachments/assets/366a2c4d-936f-4d42-9b2d-819ac1800a44)


Как можем заметить все работает. 

Проверим сертификат который мы ранее создали:

![изображение](https://github.com/user-attachments/assets/7cc59d35-92da-4858-9e01-bd7aac58c080)

## Схема организации контейеров и сервисов ##


![изображение](https://github.com/user-attachments/assets/804e7d73-5c0c-4b72-b89a-c47dc4bc035a)
