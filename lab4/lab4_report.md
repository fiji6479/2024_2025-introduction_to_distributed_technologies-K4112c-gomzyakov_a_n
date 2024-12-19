University: [ITMO University](https://itmo.ru/ru/)

Faculty: [FICT](https://fict.itmo.ru)

Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)

Year: 2024/2025

Group: K4112c

Author: Gomzyakov Alexander Nikolaevich

Lab: Lab4

Date of create: 18.12.2024

Date of finished: 19.12.2024

# Сети связи в Minikube, CNI и CoreDNS #

## Цель работы: ##
Познакомиться с CNI Calico и функцией IPAM Plugin, изучить особенности работы CNI и CoreDNS.

## Ход работы: ##

Первым делом запустим Minkube с плагином cni=calico и режиме Multi-Node clusters. 

Для этого запустим следующую команду:

```
minikube start --network-plugin=cni --cni=calico --nodes 2
```

Проверим что поды и ноду успешно равернулись: 

![изображение](https://github.com/user-attachments/assets/d6958ae5-c526-41b1-9cfb-20323c66a9df)

![изображение](https://github.com/user-attachments/assets/956d678f-f84c-4d7d-999c-37e4406e0f85)

Далее удалим стандартный ippool

![изображение](https://github.com/user-attachments/assets/53403436-dc6b-492c-b140-bc901ee28ab0)

Создадим ippool для региона россия и европа:

`
europa-ippool.yml
`


```
apiVersion: projectcalico.org/v3
kind: IPPool
metadata: 
  name: europe-ipv4-ippool
spec:
  nodeSelector: zone == "europe"
  cidr: 192.168.2.0/24
  ipipMode: Always
  natOutgoing: true
```

`
russia-ippool.yml
`

```
apiVersion: projectcalico.org/v3
kind: IPPool
metadata: 
  name: russia-ipv4-ippool
spec:
  nodeSelector: zone == "russia"
  cidr: 192.168.1.0/24
  ipipMode: Always
  natOutgoing: true
```

Далее для каждой ноды укажем label
![изображение](https://github.com/user-attachments/assets/89d77928-7362-475b-8af4-c4fca8c293ff)

Теперь необходимо создать deployment с двумя репликами контейнера и передать переменные как в предидущей лабораторной работе. 

![изображение](https://github.com/user-attachments/assets/0a533ac4-7e8f-4fcd-a9d1-26e63e36adef)

![изображение](https://github.com/user-attachments/assets/7be852d3-fdcd-4650-aa2c-cdcfa7efd3b4)

Пробросим порты и подключимся через браузер к нашему контейнеру: 

![изображение](https://github.com/user-attachments/assets/197f5b49-1557-402d-80cc-002692969368)

Как можем заметить по ip, мы подключились из зоны russia.

Проверим ip контейнеров чтобы убедится в том, что все работает правильно. 
![изображение](https://github.com/user-attachments/assets/101a6543-7807-4122-aa5c-3d09b508e9a7)

Проверим доступность одного пода внутри другого. Для этого воспользуемся командой kubectl exec и пропингуем другой под. 

![изображение](https://github.com/user-attachments/assets/5b2fb1d3-6a2b-46db-86f4-986b4c1bf8c6)

Пинги успешно проходят, а это значит что все сделанно правильно.


## Схема организации контейеров и сервисов ##
