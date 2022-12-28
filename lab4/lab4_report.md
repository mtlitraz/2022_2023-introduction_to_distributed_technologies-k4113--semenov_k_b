University: [ITMO University](https://itmo.ru/ru/)
Faculty: [FICT](https://fict.itmo.ru)
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)
Year: 2022/2023
Group: 
Author: 
Lab: Lab4
Date of create: 
Date of finished: 
### Ход работы.
1. Необходимо при запуске minikube установить плагин CNI=calico и режим работы Multi-Node Clusters одновеременно, в рамках данной лабораторной работы нужно развернуть 2 ноды.
#### Что такое Calico? 
CNI (Container Networking Interface)- сетевой интерфейс, его задача — обеспечить всё необходимое для стандартизированного управления сетевыми интерфейсами в Linux-контейнерах и гибкого расширения сетевых возможностей.
Одной из главных ценностей CNI, конечно же, являются сторонние плагины, обеспечивающие поддержку различных современных решений для Linux-контейнеров. Среди них как раз находится Project Calico.
Плагин Calico может использоваться в интеграции с Flannel (подпроект Canal) или самостоятельно, покрывая как функции по обеспечению сетевой связности, так и возможности управления доступностью. Calico расширяет функционал NetworkPolicy следующим образом: 
- политики могут применяться к любому объекту: pod, контейнер, виртуальная машина или интерфейс;
- правила могут содержать конкретное действие (запрет, разрешение, логирование);
- в качестве цели или источника правил может быть порт, диапазон портов, протоколы, HTTP- или ICMP-атрибуты, IP или подсеть (4 или 6 поколения), любые селекторы (узлов, хостов, окружений);
- дополнительно можно регулировать прохождение трафика с помощью настроек DNAT и политик проброса трафика.
Именно поэтому, а ещё потому что CNI, который используется в Minikube по-умолчанию, не поддерживает никакую NetworkPolicy, нам необходимо установить Calico.
### Установка Calico
После запуска minikube при помощи оригинальной инструкции для установки Calico в Minikube и инструкции для включения двух нод  устанавливаем плагин при помощи следующей команды `minikube start --driver=docker -p multinode-cluster --network-plugin=cni --cni=calico --nodes 2 --kubernetes-version=v1.24.0`. Эта же команда запускает режим работы multi-node clusters и разворачивает 2 ноды.
2. Проверьте работу CNI плагина Calico и количество нод, результаты проверки приложите в отчет.
- Убеждаемся в запуске двух нод: `kubectl get nodes` .
![image](https://user-images.githubusercontent.com/121423344/209806320-6e577cd6-a40d-45e3-8592-5a4e65b43d7d.png)
- Проверим ноды с пометкой calico-node, чтобы убедиться в работе плагина: `kubectl get pods -l k8s-app=calico-node -A`
![image](https://user-images.githubusercontent.com/121423344/209806366-369fcdd1-c803-4e5a-8463-d439a915a82e.png)
3. Для проверки работы Calico попробовать одну из функций под названием IPAM Plugin.
- Для проверки режима IPAM для запущеных ранее нод указываем label по признаку стойки или географического расположения Соглавно инструкции по назначению ip-адресов удаляем дефолтный ippool: `kubectl delete ippool default-ipv4-ippool`
![image](https://user-images.githubusercontent.com/121423344/209806425-65a5668e-67f5-4824-a605-4180e77a7af4.png)
- убеждаемся что никаких айпи-пулов не осталось: `kubectl get ippools`
![image](https://user-images.githubusercontent.com/121423344/209806468-8c3e1902-2250-450f-97ed-21dfa759bea5.png)
- Следуя оригинальной инструкции для назначения IP адресов в Calico пишем манифесты для IPPool-ресурсов
С помощью IPPool можно создать IP-pool (блок IP-адресов), который выделяет IP-адреса только для узлов с определенной меткой (label).
Чтобы назначить метки узлам, используем команды:
```
kubectl label nodes multinode-demo zone=east  
![image](https://user-images.githubusercontent.com/121423344/209806523-0a486223-f040-410b-a676-d6ab580413d1.png)
kubectl label nodes multinode-demo-m02 zone=west
![image](https://user-images.githubusercontent.com/121423344/209806545-2225dbc1-b5aa-4123-b0af-ef70d1f2bf97.png)
```
- Скачиваем шаблон манифеста IPPools, но пока не разворачиваем его, для этого нам нужен Calico.
```
apiVersion: projectcalico.org/v3
kind: IPPool
metadata:
   name: zone-east-ippool
spec:
   cidr: 192.168.0.0/24
   ipipMode: Always
   natOutgoing: true
   nodeSelector: zone == "east"
---
apiVersion: projectcalico.org/v3
kind: IPPool
metadata:
   name: zone-west-ippool
spec:
   cidr: 192.168.1.0/24
   ipipMode: Always
   natOutgoing: true
   nodeSelector: zone == "west"
   ```
4. После этого нужно разработать манифест для Calico который бы на основе ранее указанных меток назначал бы IP адреса "подам" исходя из пулов IP адресов которые мы указали в манифесте.   
- Чтобы взаимодействовать с Calico, нужно скачать и задеплоить его yaml образ с офф.сайта `kubectl apply -f lab4_calico.yaml
![image](https://user-images.githubusercontent.com/121423344/209806635-869fed3f-321f-459b-8cdd-efd0920b0cec.png)
- Теперь, взаимодействуя с Calico через неймспейс kube-system, разворачиваем наши IPPools командой `kubectl exec -i -n kube-system calicoctl -- /calicoctl --allow-version-mismatch create -f - < lab4_ippool.yaml`. Команду пишем в cmd, так как в обычном терминале не получается. Добавка --allow-version-mismatch нужна чтобы устранить разницу в версиях клиента и кластера.
- ![image](https://user-images.githubusercontent.com/121423344/209806741-5a8b33a1-e740-497e-80e6-095810265791.png)
- ![image](https://user-images.githubusercontent.com/121423344/209806836-8bf63bd5-32b3-4138-add4-d979bb4ae578.png)
- Проверяем что создалиcь два IPPols командой `kubectl exec -i -n kube-system calicoctl -- /calicoctl --allow-version-mismatch get ippools -o wide`
![image](https://user-images.githubusercontent.com/121423344/209806884-95151684-6ba6-4e39-a0e5-ce92924120f7.png)
5. Далее необходимо создать deployment с 2 репликами контейнера ifilyaninitmo/itdt-contained-frontend:master и передать переменные в эти реплики: REACT_APP_USERNAME, REACT_APP_COMPANY_NAME. Всё, как в предыдущих двух лабах.
- Разворачиваем `deployment` командой  `kubectl apply -f lab4_deployment.yaml`
- ![image](https://user-images.githubusercontent.com/121423344/209807069-4811a405-0653-4449-98d2-6d662ac824b7.png)
6. Создать сервис через который у вас будет доступ на эти "поды". Выбор типа сервиса остается на ваше усмотрение.
- Сервис так же берём из прошлых лабораторных и разворачиваем `kubectl apply -f lab4_service.yaml`
- Сервис работает нормально, чего не скажешь о подах:
![image](https://user-images.githubusercontent.com/121423344/209807112-3b329fa2-fe8e-436c-976b-38e455ea18cd.png)
Error: ImagePullBackOff. Чтобы исправить эту ошибку, пробовал пушить образ контейнера в докер - ответом было следующее:
![image](https://user-images.githubusercontent.com/121423344/209807230-425aeff0-7c8f-45f6-92b7-e638003f5dc8.png)
- подробное описание ошибки можем посмотреть в `kubectl describe pod lab4-deployment-68b9cb4c4b-z5k9l`
![image](https://user-images.githubusercontent.com/121423344/209807277-e765049a-5e46-4062-9b49-c7425f68963f.png)
- Дальнейший проброс порта так же не представляется возможным:
![image](https://user-images.githubusercontent.com/121423344/209807305-33b2f6f5-3061-4c54-aaad-b99f92befb77.png)
7. Схема организации контейнеров и сервисов.

