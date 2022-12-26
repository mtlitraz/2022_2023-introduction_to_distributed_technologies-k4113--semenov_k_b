University: [ITMO University](https://itmo.ru/ru/)
Faculty: [FICT](https://fict.itmo.ru)
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)
Year: 2022/2023
Group: К4113с
Author: Semenov K.B.
Lab: Lab1
Date of create:   
Date of finished: 

# Ход работы
1. Устанавливаем и запускаем Docker, далее по оригинальной инструкции устанавливаем Minikube- одноузловой кластер Kubernetes для его возможностей на локальном компьютере. Так же дополнительно устанавливаем инструмент командной строки kubectl.
2. Разворачиваем minikube cluster при помощи команды `minikube start` 
3. Пишем манифест для создания пода на основе образа HashiCorp Vault и развёртываем первый "под" -минимальную единицу Кубернетис.
- Развертывание происходит при помощи команды `kubectl apply -f vault.yaml`, где vault.yaml- написанный ранее манифест.
- Проверяем созданный под при помощи команды `kubectl get pods`
- ![image](https://user-images.githubusercontent.com/121423344/209536294-74b28b2a-5569-4714-aaaf-2d72c8d8c784.png)
4. Создаём сервис доступа к этому контейнеру, выполнив команду
- `minikube kubectl -- expose pod vault --type=NodePort --port=8200`
5. При помощи команды, которая создает сервис для открытия доступа к приложению по порту 8200 `minikube kubectl -- port-forward service/vault 8200:8200` попадаем в наш контейнер, перейдя по адресу http://localhost:8200/
![image](https://user-images.githubusercontent.com/121423344/209532525-af188bc5-f66d-4e51-928d-ffe596c2a39b.png)
6. Чтобы войти в Vault, нужно найти токен, для этого заходим в логи контейнера при помощи команды `minikube kubectl -- logs vault`
![image](https://user-images.githubusercontent.com/121423344/209535835-9b53893e-ef10-4124-be9f-9a14cc697932.png)
- При помощи найденного токена входим в Vault:
- ![image](https://user-images.githubusercontent.com/121423344/209536150-a9054f4c-d862-47fc-8bf7-f8360ae5c6c4.png)
7. Останавливаем minicube cluster при помощи команды `minikube stop`.
8. Схема организации контейнеров, нарисованная в draw.io
9. 
