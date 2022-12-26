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
3. Включить `minikube addons enable ingress` и сгенерировать TLS сертификат, импортировать сертификат в minikube.
