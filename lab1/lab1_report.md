University: [ITMO University](https://itmo.ru/ru/)

Faculty: [FICT](https://fict.itmo.ru)

Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)

Year: 2024/2025

Group: K4112c

Author: Gomzyakov Alexander Nikolaevich

Lab: Lab1

Date of create: 05.11.2024

Date of finished: 06.11.2024

# Установка Docker и Minikube, мой первый манифест. #

## Цель работы: ##
Ознакомиться с инструментами Minikube и Docker, развернуть свой первый "под".

## Ход работы: ##

### Подготовка  ###
Первым делом установим minikube и docker.

Далее был развернут minikube cluster с помощью команды: 

`
minikube start
`

### Создание манифеста  ###
Для развертывания пода HashiCorp Vault был написан следующий манифест: 

```
apiVersion: v1
kind: Pod
metadata:
  name: vault-pod
  labels:
    name: vault-pod 
spec:
  containers:
  - name: vault
    image: vault:1.13.3
    ports:
      - containerPort: 8200  
```

Для создания пода в кластере воспользуемся этой командой:

`
kubectl apply -f vault-pod.yml
`

### Сервис для доступа к контейнеру  ###
После этого необходимо создать сервис для доступа к этому контейнеру с помощью слeдующей команды:

`
minikube kubectl -- expose pod vault-pod --type=NodePort --port=8200
`

Для того, чтобы зайти в контейнер, нужно прокинут порт локального компьютера в контейнер с помощью команды:

`
minikube kubectl -- port-forward service/vault-pod 8200:8200
`

После этого переходим по ссылке <http://localhost:8200>, где открывается следующий интерфейс:

![изображение](https://github.com/user-attachments/assets/18614400-d046-4980-ad59-cad3911de80e)

### Получение токена для доступа  ###

Для того чтобы получить токен, нужно посомтреть логи с помощью команды:

`
kubectl logs vault-pod
`

Результат выполнения команды:

![изображение](https://github.com/user-attachments/assets/f2d63a94-578e-4f5c-a474-f837cb650e19)

С помощью токена с предыдущего скриншота получилось войти в систему: 

![изображение](https://github.com/user-attachments/assets/b62002ba-3488-4382-8584-95489449e288)

### Ответы на вопросы  ###

1) Что сейчас произошло и что сделали команды указанные ранее?
  Развернуто приложение Vault в кластере Kubernetes. Создан манифест, согласно которому был создан под с контейнером. В контейнере запущено приложение Vault. Для работы контейнеру был указан порт 8200. Сервис открывает доступ на порт контейнера. Последней командой был настроен проброс порта 8200 на localhost в порт 8200 на контейнере.

2) Где взять токен для входа в Vault?
   С помощью следующей команды:
   
    `
    kubectl logs vault-pod
    `

Схема организации контейеров и сервисов:

![изображение](https://github.com/user-attachments/assets/b05eba2d-2cb3-42e0-8615-f2ec13b8ee45)


