University: [ITMO University](https://itmo.ru/ru/)
Faculty: [FICT](https://fict.itmo.ru)
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)
Year: 2022/2023
Group: К4113с
Author: Semenov K.B.
Lab: Lab3
Date of create:   
Date of finished: 
# Ход работы
1. Необходимо создать `configMap` с переменными: `REACT_APP_USERNAME`, `REACT_APP_COMPANY_NAME`.
- `configMap`- объект API, используемый для хранения неконфиденциальных данных в парах ключ-значение. Поды могут использовать ConfigMap как переменные среды, аргументы командной строки или как файлы конфигурации в томе. Развёртывание его предполагает так же наличие .yaml файла, который мы можем найти в официальной документации.
![image](https://user-images.githubusercontent.com/121423344/209566895-e0ae1586-a979-425f-9ca9-991b75ea84e7.png)
- Развёртываем манифест при помощи команды `kubectl apply -f configmaplr3.yaml`
- ![image](https://user-images.githubusercontent.com/121423344/209568107-a690fc3b-cf4e-41c5-9311-b973183596de.png)
2. Необходимо создать `replicaSet` с двумя репликами контейнера ifilyaninitmo/itdt-contained-frontend:master и используя ранее созданный `configMap` передать переменные `REACT_APP_USERNAME`, `REACT_APP_COMPANY_NAME`.
- целью `replicaSet` является поддержание стабильного набора реплик "подов", работающих в любой момент времени. Таким образом, это часто используется для гарантии наличия заданного количества идентичных модулей. Пример манифеста можно так же найти в документации.
![image](https://user-images.githubusercontent.com/121423344/209571542-fafd94a2-cff6-491b-aef8-5b98afa150fa.png)
- В манифест при помощи `env` добавляем в поды переменные окружение `REACT_APP_USERNAME` и `REACT_APP_COMPANY_NAME`, с соответствующими ранее созданному `configMap` значениями.
- Разворачиваем replicaSet командой `kubectl create -f replicasetlr3.yaml`.
- с помощью команды `kubectl get rs` убеждаемся в развертывании ![image](https://user-images.githubusercontent.com/121423344/209644448-fc91de9e-eb31-431b-b06e-d2688e01286a.png)
3. Включить `minikube addons enable ingress` и сгенерировать TLS сертификат, импортировать сертификат в minikube.
- для начала создадим сервис для работы ingress аддона (который работает с сервисами типа `NodePort` и `LoadBalancer`, манифест пишём по шаблону из документации.
![image](https://user-images.githubusercontent.com/121423344/209572937-c42f7d07-fea9-43c3-b90c-0660557ac865.png)
- Убеждаемся что сервис создан ![image](https://user-images.githubusercontent.com/121423344/209644154-21465d0d-480d-4d10-b689-762cc2f057e8.png)
- Исходя из официальной документации minikube, TLS сертификат с ingress аддоном можно создать при помощи утилиты `mkcert` - простого инструмента для создания локально доверенных сертификатов разработки, не требующего настройки.
- Устанавливаем утилиту в minikube командой `mkcert install`.
- После установки генерируем сертификат командой `mkcert "произвольное название хоста"`
- ![image](https://user-images.githubusercontent.com/121423344/209646741-938a2842-514d-4ee1-abe7-fa0948302258.png)
- Сгенерированный сертификат и ключ к нему появились в папке с лабой
- ![image](https://user-images.githubusercontent.com/121423344/209646224-12d34434-f91e-4f96-a8d2-f41a0f9db44e.png)
- Теперь нужно создать secret командой `kubectl create secret tls mkcert --key semenov.cloud-key.pem --cert semenov.cloud.pem` (Секрет – это объект, содержащий небольшой объем конфиденциальных данных, например пароль, маркер или ключ. В противном случае такая информация может быть помещена в pod спецификация или в образ контейнера. Использование secret означает, что вам не нужно включать конфиденциальные данные в свой код приложения.)
- Проверяем создание сикрета командой `kubectl get secret`
- ![image](https://user-images.githubusercontent.com/121423344/209646921-82f4833e-9753-4507-9c83-a1ff6b1c7c63.png)
- После создания генерации сертификата и создания сикрета можно подключить аддон ingress в minikube
minikube addons enable ingress; 
minikube addons enable ingress-dns.
- Для ingress нужно написать манифест. Шаблон берём тут https://kubernetes.io/docs/concepts/services-networking/ingress/
- ![image](https://user-images.githubusercontent.com/121423344/209647900-ccf463b6-5ff7-4eff-8920-ebf1045dd296.png)
- ![image](https://user-images.githubusercontent.com/121423344/209648650-0f8fcdb5-c12b-45ac-ae92-75def7badff9.png)
- В поля host указываем наш хост, указанный при генерации сертификата - semenov.cloud (он же наш FQDN), так же указываем нужный secretName
- По информации из документации TLS не будет работать пока не добавить IP адрес нашего ingress и FQDN, то есть 127.0.0.1 в файл hosts, лежащий в C:\Windows\System32\drivers\etc
![image](https://user-images.githubusercontent.com/121423344/209648396-860c4738-9b8c-48b6-ade8-9abb1ed30a76.png)
4. Войдите в веб приложение по вашему FQDN используя HTTPS и проверьте наличие сертификата.
- Развернув ingress и добавив FQDN и IP в файл hosts, запускаем туннелирование `minikube tunnel` и переходим по ссылке https://semenov.cloud
- ![image](https://user-images.githubusercontent.com/121423344/209717149-d4670797-0129-4a65-b657-12bf009e1d4e.png)
- ReactApp белый экран, по информации от преподавателя на винде это нормально сертификат при этом показывает исправно.
5. Схема органазации контейнеров и сервисов

