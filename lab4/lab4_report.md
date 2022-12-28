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
image.png
- Проверим ноды с пометкой calico-node, чтобы убедиться в работе плагина: `kubectl get pods -l k8s-app=calico-node -A`
image.png
3. Для проверки работы Calico попробовать одну из функций под названием IPAM Plugin.
- Для проверки режима IPAM для запущеных ранее нод указываем label по признаку стойки или географического расположения Соглавно инструкции по назначению ip-адресов удаляем дефолтный ippool: `kubectl delete ippool default-ipv4-ippool`
image.png
- убеждаемся что никаких айпи-пулов не осталось: `kubectl get ippools`
- Следуя оригинальной инструкции для назначения IP адресов в Calico пишем манифесты для IPPool-ресурсов
С помощью IPPool можно создать IP-pool (блок IP-адресов), который выделяет IP-адреса только для узлов с определенной меткой (label).
Чтобы назначить метки узлам, используем команды:
```
kubectl label nodes multinode-demo zone=east  
kubectl label nodes multinode-demo-m02 zone=west
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
- Чтобы взаимодействовать с Calico, нужно скачать и задеплоить его yaml образ с офф.сайта 
image.png
- Теперь, взаимодействуя с Calico через неймспейс kube-system, разворачиваем наши IPPools командой `kubectl exec -i -n kube-system calicoctl -- /calicoctl --allow-version-mismatch create -f - < lab4_ippool.yaml`. Команду пишем в cmd, так как в обычном терминале не получается. Добавка --allow-version-mismatch нужна чтобы устранить разницу в версиях клиента и кластера.
- Проверяем что создалиcь два IPPols командой `kubectl exec -i -n kube-system calicoctl -- /calicoctl --allow-version-mismatch get ippools -o wide`
image.png
5. Далее необходимо создать deployment с 2 репликами контейнера ifilyaninitmo/itdt-contained-frontend:master и передать переменные в эти реплики: REACT_APP_USERNAME, REACT_APP_COMPANY_NAME. Всё, как в предыдущих двух лабах.
- Разворачиваем `deployment` командой  `kubectl apply -f lab4_deployment.yaml`
6. Создать сервис через который у вас будет доступ на эти "поды". Выбор типа сервиса остается на ваше усмотрение.
- Сервис так же берём из прошлых лабораторных и разворачиваем `kubectl apply -f lab4_service.yaml`
- Сервис работает нормально, чего не скажешь о подах:
image.png
Error: ImagePullBackOff. Чтобы исправить эту ошибку, пробовал пушить образ контейнера в докер - ответом было следующее:
image.png
- подробное описание ошибки можем посмотреть в `kubectl describe pod lab4-deployment-68b9cb4c4b-z5k9l`
image.png
- Дальнейший проброс порта так же не представляется возможным:
image.png
7. Схема организации контейнеров и сервисов.

