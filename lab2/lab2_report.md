University: [ITMO University](https://itmo.ru/ru/)
Faculty: [FICT](https://fict.itmo.ru)
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)
Year: 2022/2023
Group: К4113с
Author: Semenov K.B.
Lab: Lab2
Date of create:   
Date of finished: 
# Ход работы
1. Необходимо создать `deployment` c двумя репликами контейнера https://hub.docker.com/repository/docker/ifilyaninitmo/itdt-contained-frontend и передать переменные в эти реплики: `REACT_APP_USERNAME`, `REACT_APP_COMPANY_NAME`.
- Первым делом запуллим в докер образ нужного нам deploy `minikube ssh docker pull ifilyaninitmo/itdt-contained-frontend:master`
- Далее, пишем манифест для нашего деплоя и разворачиваем его при помощи команды `kubectl apply -f deploymentlr2` 
- ![image](https://user-images.githubusercontent.com/121423344/209550102-bd5f09e6-d6c4-46ad-9811-2f1e983add9d.png)
Манифест для развёртывания взят из официальной документации, чтобы запустить 2 экземляра пода, добавляем в `spec` значение `replicas: 2`. Так же, добавляем свойство `env` внутри которого объявляем переменные окружения `REACT_APP_USERNAME` и `REACT_APP_COMPANY_NAME`, подставив произвольные значения.
- Проверяем созданные поды командой `kubectl get pods`, т.к. в задании указывались 2 реплики контейнера, подов должно быть соответственно 2.
- ![image](https://user-images.githubusercontent.com/121423344/209542084-48ced062-37f8-425a-9c5b-4529b102ff47.png)
2. Необходимо создать сервис, через который у нас будет доступ на эти "поды". Выбор сервиса производим произвольно из четырёх возможных (согласно инструкции на хабре: https://habr.com/ru/company/southbridge/blog/358824/):
- `ClusterIP` - сервис к8с по умолчанию, обеспечивает сервис внутри кластера, к которому могут обращаться другие приложения внутри кластера. Внешнего доступа нет.
- `NodePort` - самый примитивный способ направить внешний трафик в сервис. NodePort, как следует из названия, открывает указанный порт для всех Nodes (виртуальных машин), и трафик на этот порт перенаправляется сервису.
- `LoadBalancer`  — стандартный способ предоставления сервиса в интернете. На GKE он развернет Network Load Balancer, который предоставит IP адрес. Этот IP адрес будет направлять весь трафик на сервис.
- `ingress` - сам по себе не сервис. Он стоит перед несколькими сервисами и действует как «интеллектуальный маршрутизатор» или точка вхождения в кластер.
- Выбрав сервис, пишем манифест для его создания и так же как и деплой, разворачиваем его командой `kubectl apply -f servicelr2.yaml`.
- ![image](https://user-images.githubusercontent.com/121423344/209542162-88879618-6ccb-4b06-8f08-d1df0e27e80c.png)
3. Необходимо запустить в `minikube` режим проброса портов и подключиться к контейнерам через браузер.
- Алгоритм запуска режима проброса портов был описан в первой лабе: `minikube kubectl -- expose deployment deploymentlr2 --port=3000 --target-port=3000 --name=servicelr2 --type=LoadBalancer`
- Подключаемся к контейнеру при помощи команды `minikube kubectl -- port-forward service/servicelr2 3000:3000` и открываем страницу `http://localhost:3000`.
- ![image](https://user-images.githubusercontent.com/121423344/209543745-3b6fb042-74c0-44f6-a1a1-85d0258d0088.png)
4. Проверить на странице в веб-браузере переменные, описанные выше и `Container name`.
- Не изменяются. При обновлении ReactApp страница становится полностью белой. Говорят, это ошибка винды. В идеале должна была поменяться переменная `container name`, потому что это особенность сервиса- он распределяет нагрузку между нодами и каждый раз открывает новый контейнер.
5. Проверить логи контейнеров и приложить в отчёт.
- Логи контейнеров проверяем при помощи команды `minikube kubectl -- logs pod/deploymentlr2-768755448b-grdzc` и, соответственно второй: `minikube kubectl -- logs pod/deploymentlr2-768755448b-ttxn6`
- ![image](https://user-images.githubusercontent.com/121423344/209549732-56dabb49-9605-4246-a0ff-a13a11466e03.png)
- ![image](https://user-images.githubusercontent.com/121423344/209549852-1adb680b-f681-4fea-8ad7-d602ba1badca.png)
6. Останавливаем minikube `minikube stop`
7. Схема организации контейнеров и сервиса, нарисованная в draw.io
