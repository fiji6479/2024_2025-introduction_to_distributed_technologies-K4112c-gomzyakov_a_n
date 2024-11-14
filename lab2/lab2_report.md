University: [ITMO University](https://itmo.ru/ru/)

Faculty: [FICT](https://fict.itmo.ru)

Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)

Year: 2024/2025

Group: K4112c

Author: Gomzyakov Alexander Nikolaevich

Lab: Lab2

Date of create: 13.11.2024

Date of finished: 14.11.2024

# Развертывание веб сервиса в Minikube, доступ к веб интерфейсу сервиса. Мониторинг сервиса. #

## Цель работы: ##
Ознакомиться с типами "контроллеров" развертывания контейнеров, ознакомится с сетевыми сервисами и развернуть свое веб приложение.

## Ход работы: ##

### Создание deployment  ###
Первым делом необходимо написать файл deployment.yml
Для этого был написан следующий файл:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: lab2-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: lab2-deployment
  template:
    metadata:
      labels:
        app: lab2-deployment
    spec:
      containers:
        - name: itdt-contained-frontend
          image: ifilyaninitmo/itdt-contained-frontend:master
          env:
            - name: REACT_APP_USERNAME
              value: Alexander Gomzyakov
            - name: REACT_APP_COMPANY_NAME
              value: ITMO
```
В этом файле были использованы две реплики контейнера  ifilyaninitmo/itdt-contained-frontend:master. Также были переданы переменные REACT_APP_USERNAME: Alexander Gomzyakov и REACT_APP_COMPANY_NAME: ITMO

Для того чтобы применить эту конфигурацию воспользуемся следующей командой: 

`
minikube kubectl -- apply -f deployment.yml
`


![изображение](https://github.com/user-attachments/assets/e78bd178-3fda-411f-baf8-cfb0ee3851d7)

### Сервис для доступа ###

Далее необходимо создать сервис через который у нас будет доступ к этим подам. Воспользуемся сервисом ClusterIP. Также укажем порт который будет открывать сервис для доступа.

`
minikube kubectl -- expose deployment lab2-deployment --port=3000 --name=lab2-service --type=ClusterIP
`


![изображение](https://github.com/user-attachments/assets/91364fa1-729d-4ffe-bf8b-4ba74a6d93b6)

Для того, чтобы зайти в контейнер, нужно прокинут порт локального компьютера в контейнер с помощью команды:

`
minikube kubectl -- port-forward service/lab2-service 3000:3000
`

После этого переходим по ссылке <http://localhost:3000>, где открывается следующий интерфейс:

![изображение](https://github.com/user-attachments/assets/1ed224af-7a9b-4fd0-8fb3-0478352aeba8)

Переменные REACT_APP_USERNAME и REACT_APP_COMPANY_NAME не изменяются. Это связанно с тем, что это переменные среды. Они были заданы при создании объекта deployment и соотвественно указаны в deployment.yml.
 
Логи двух контейнеров: 
lab2-deployment-7f9dd6767c-88pjx:

![изображение](https://github.com/user-attachments/assets/fd3b5736-0442-44d3-a679-41c2446c0643)

lab2-deployment-7f9dd6767c-hk2bl:

![изображение](https://github.com/user-attachments/assets/b4830e5d-9fe0-430b-8979-d1c24c3d4e2e)

Схема организации контейеров и сервисов:

![изображение](https://github.com/user-attachments/assets/f2fc9452-f693-429d-ab62-bc3fd523538f)



