## HomeWork #23 (kubernetes-5)
- В кластере k8s развернул из chart'а prometheus и несколько инстансов приложения reddit в разных окружениях:
```
➜  kubernetes git:(kubernetes-5) helm ls
NAME            REVISION        UPDATED                         STATUS          CHART                   APP VERSION     NAMESPACE
nginx           1               Fri Oct  4 22:09:53 2019        DEPLOYED        nginx-ingress-1.24.0    0.26.1          default
production      1               Fri Oct  4 22:47:48 2019        DEPLOYED        reddit-1.0.0            1               production
prom            8               Sat Oct  5 18:57:17 2019        DEPLOYED        prometheus-9.1.1        2.11.1          default
reddit-test     1               Fri Oct  4 22:47:20 2019        DEPLOYED        reddit-1.0.0            1               default
staging         1               Fri Oct  4 22:48:15 2019        DEPLOYED        reddit-1.0.0            1               staging
```
- Для prometheus создан файл [custom-values.yml](https://github.com/otus-devops-2019-05/vvorontsov_microservices/blob/kubernetes-5/kubernetes/Charts/prometheus/custom_values.yml) в который добавлены настройки endpoints для сервисов приложения. Также включен node-exporter и настроен Service Discovery.

`helm upgrade prom . -f custom_values.yml --install`

- Развернул в кластере grafana:
```
helm upgrade --install grafana stable/grafana \
--set "adminPassword=admin" \
--set "service.type=NodePort" \
--set "ingress.enabled=true" \
--set "ingress.hosts={reddit-grafana}"
```

- Загрузил и кастомизировал дашборды для grafana:
```
➜  kubernetes git:(kubernetes-5) tree ./Charts/prometheus/dashboards
./Charts/prometheus/dashboards
├── Business_Logic_Monitoring-1570381023663.json
├── Docker_and_system_monitoring-1570381007597.json
├── Kubernetes_cluster_monitoring-1570380968800.json
├── Kubernetes_Deployment_metrics-1570381042241.json
└── UI_Service_Monitoring-1570381074900.json
```
Добавил возможность фильтровать графики с метриками сервисов по namespace:

```
"expr": "rate(post_count{kubernetes_namespace=~\"$namespace\"}[1h])"
"expr":"rate(ui_request_count{kubernetes_namespace=~\"$namespace\",http_status=\"200\"}[1m])"
```
## HomeWork #22 (kubernetes-4)
- Создал кластер kubernetes в GCP
- Составил Helm Chart'ы для всех сервисов (ui, post, comment, mongodb)
- Потестил разворачиваение приложения в кластере k8s с помощью Helm+Tiller
- Потестил работу [helm2 tiller plugin](https://github.com/rimusz/helm-tiller) и Helm3(beta)
- Развернул GitLab используя готовый Chart из репозитория:
```
➜  gitlab-omnibus git:(kubernetes-4) kubectl get pods                                                                           
NAME                                        READY   STATUS    RESTARTS   AGE
gitlab-gitlab-748cb4b566-mlrld              1/1     Running   2          174m
gitlab-gitlab-postgresql-7b99699f5b-zt97p   1/1     Running   0          174m
gitlab-gitlab-redis-58bdb6bb8b-q9srb        1/1     Running   0          3h11m
gitlab-gitlab-runner-558c8695b8-xqnkt       1/1     Running   8          174m
```
- Создал проекты для билда сервисов и проект для деплоя приложения reddit
```
 -reddit-deploy
- post
- ui
- comment
```
- Создал для всех проектов файлы  .gitlab-ci
- Потестировал pipline с auto_devops и с динамическими окружениями. 
- Переписал все pipline'ы без использования auto_devops

#### Задание со *
- Настроил деплой по триггеру согласно документации Gitlab ([Triggering pipelines through the API
](https://docs.gitlab.com/ee/ci/triggers/))

`curl --request POST --form token=$DEPLOY --form ref=master http://gitlab-gitlab/api/v4/projects/4/trigger/pipeline`

## HomeWork #21 (kubernetes-3)
Создал следующие ресурсы:
- LoadBalancer Service
```
➜  reddit git:(kubernetes-3) ✗ kubectl get service  -n dev --selector component=ui
NAME   TYPE           CLUSTER-IP   EXTERNAL-IP     PORT(S)        AGE
ui     LoadBalancer   10.77.6.30   34.76.209.112   80:32092/TCP   4m35s
```
- Ingress
```
➜  vvorontsov_microservices git:(kubernetes-3) kubectl get ingress -n dev 
NAME   HOSTS   ADDRESS         PORTS     AGE
ui     *       35.244.232.51   80, 443   69m
```
- Secret (*)
```
apiVersion: v1
kind: Secret
metadata:
  name: ui-ingress
  labels:
    app: reddit
    component: ui
type: kubernetes.io/tls
data:
  tls.crt: LS0tLS1CRUdJTi...
  tls.key: LS0tLS1CRUdJTi...

```

```
➜  vvorontsov_microservices git:(kubernetes-3) kubectl get secret -n dev 
NAME                  TYPE                                  DATA   AGE
default-token-b6j6s   kubernetes.io/service-account-token   3      105m
ui-ingress            kubernetes.io/tls                     2      57m

```
- TLS сертификат и ключ

`openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=35.244.232.51"`

- Network Policies
```
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-db-traffic
  labels:
    app: reddit
spec:
  podSelector:
    matchLabels:
      app: reddit
      component: mongo
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: reddit
          component: comment
    - podSelector:
        matchLabels:
          app: reddit
          component: post      

```
- PersistentVolumes, PersistentVolumeClaims
```
➜  reddit git:(kubernetes-3) ✗ kubectl get persistentvolume -n dev
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM                   STORAGECLASS   REASON   AGE
pvc-52faeec9-e461-11e9-87b3-42010a840072   10Gi       RWO            Delete           Bound       dev/mongo-pvc-dynamic   fast                    67s
pvc-724a50de-e460-11e9-87b3-42010a840072   15Gi       RWO            Delete           Bound       dev/mongo-pvc           standard                7m24s
reddit-mongo-disk                          25Gi       RWO            Retain           Available     
```

## HomeWork #20 (kubernetes-2)
- Развернул локальное окружение для работы с Kubernetes
- Развернул Kubernetes в GKE
```
➜  vvorontsov_microservices git:(kubernetes-2) ✗ kubectl get nodes -o wide
NAME                                                STATUS   ROLES    AGE     VERSION         INTERNAL-IP   EXTERNAL-IP      OS-IMAGE                             KERNEL-VERSION   CONTAINER-RUNTIME
gke-standard-cluster-1-default-pool-1221882e-28j5   Ready    <none>   8m50s   v1.13.7-gke.8   10.128.0.3    35.222.133.4     Container-Optimized OS from Google   4.14.127+        docker://18.9.3
gke-standard-cluster-1-default-pool-1221882e-86r6   Ready    <none>   8m48s   v1.13.7-gke.8   10.128.0.4    35.226.193.250   Container-Optimized OS from Google   4.14.127+        docker://18.9.3

```
- Запустил reddit в Kubernetes: http://35.222.133.4:30325

## HomeWork #19 (kubernetes-1)
- Развернул кластер Kubernetes используя инструкции в [Kubernetes The Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way) и разобрался с основными его компонентами.

- Создал deployment файлы для приложения reddit:
```
 - kubernetes/reddit/comment-deployment.yml
 - kubernetes/reddit/mongo-deployment.yml
 - kubernetes/reddit/post-deployment.yml
 - kubernetes/reddit/ui-deployment.yml

```
- Запустил контейнеры с приложением reddit:
```
➜  the_hard_way git:(kubernetes-1) ✗ kubectl get pods -o wide 
NAME                                  READY   STATUS    RESTARTS   AGE   IP            NODE       NOMINATED NODE   READINESS GATES
busybox-78c88d76df-4fgdn              1/1     Running   11         11h   10.200.0.4    worker-0   <none>           <none>
comment-deployment-7dfd97bb5d-gmxxd   1/1     Running   0          10h   10.200.0.8    worker-0   <none>           <none>
mongo-deployment-86d49445c4-ndkgz     1/1     Running   0          10h   10.200.0.10   worker-0   <none>           <none>
nginx-7bb7cd8db5-gnh8f                1/1     Running   0          11h   10.200.0.5    worker-0   <none>           <none>
post-deployment-7bc8df4f55-mhlhq      1/1     Running   0          10h   10.200.0.7    worker-0   <none>           <none>
ui-deployment-95f7bf4cd-wn2lm         1/1     Running   0          10h   10.200.0.9    worker-0   <none>           <none>
untrusted                             1/1     Running   0          11h   10.200.0.6    worker-0   <none>           <none>

```

## HomeWork #18 (logging-1)
- Собрал образ fluentd с необходимыми плагинами:
```
# Dockerfile
FROM fluent/fluentd:v0.12
RUN gem install fluent-plugin-elasticsearch --no-rdoc --no-ri --version 1.9.5
RUN gem install fluent-plugin-grok-parser --no-rdoc --no-ri --version 1.0.0
ADD fluent.conf /fluentd/etc
```
- Создал конфиг для fluentd :
```
<source>
  @type forward
  port 24224
  bind 0.0.0.0
</source>

<match *.**>
  @type copy
  <store>
    @type elasticsearch
    host elasticsearch
    port 9200
    logstash_format true
    logstash_prefix fluentd
    logstash_dateformat %Y%m%d
    include_tag_key true
    type_name access_log
    tag_key @log_name
    flush_interval 1s
  </store>
  <store>
    @type stdout
  </store>
</match>
```
- Создал файл docker-compose-logging.yml для EFK стэка. Для elasticsearch и kibana пришлось установить environment variables чтобы всё заработало. Официальная документация elastic:

  [Install Elasticsearch with Docker](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html#docker)

  [Running Kibana on Docker](https://www.elastic.co/guide/en/kibana/current/docker.html#docker)

```
version: '3'

services:
  fluentd:
    image: ${USER_NAME}/fluentd
    container_name: fluentd
    ports:
      - "24224:24224"
      - "24224:24224/udp"
    networks:
      - back_net

  elasticsearch:
    image: elasticsearch:7.3.1
    container_name: elasticsearch
    environment:
      - node.name=elasticsearch
      - cluster.name=docker-cluster
      - discovery.type=single-node
      - "ES_JAVA_OPTS=-Xmx256m -Xms256m"
    volumes:
      - es_data:/usr/share/elasticsearch/data
    ports:
      - "9200:9200"
    networks:
      - back_net

  kibana:
    image: kibana:7.3.1
    container_name: kibana
    environment:
      ELASTICSEARCH_HOSTS: http://elasticsearch:9200
    ports:
      - "5601:5601"
    networks:
      - back_net

volumes:
  es_data:

networks:
  front_net:
  back_net:
```
- Для сервисов ui и post в качестве logging driver установил fluentd:

```
    logging:
      driver: "fluentd"
      options:
        fluentd-address: localhost:24224
        tag: service.name
```

- Настроил для fluentd фильтры логов:
```
# фильтр структурированных логов (json)
<filter service.post>
  @type parser
  format json
  key_name log
</filter>
```
   
```
# фильтр неструктурированных логов (grok)
<filter service.ui>
  @type parser
  key_name log
  format grok
  grok_pattern %{RUBY_LOGGER}
</filter>
<filter service.ui>
  @type parser
  format grok
  <grok>
    pattern service=%{WORD:service} \| event=%{WORD:event} \| request_id=%{GREEDYDATA:request_id} \| message='%{GREEDYDATA:message}'
  </grok>
  key_name message
  reserve_data true
</filter>
```
- Добавил zipkin для трассировки запросов к нашему приложению:

```
 zipkin:
    image: openzipkin/zipkin
    container_name: zipkin
    ports:
      - "9411:9411"
    networks:
      - back_net
      - front_net 
```

#### Задание со *
1. Доработал grok фильтр для парсинга сообщений такого формата: `service=ui | event=request | path=/ | request_id=a9df9574-47d0-45fb-ace7-4d43f1216a46 | remote_addr=92.241.102.253 | method= GET | response_status=200`

```
pattern service=%{WORD:service} \| event=%{WORD:event} \| path=%{URIPATH:path} \| request_id=%{GREEDYDATA:request_id} \| remote_addr=%{IPV4:remote_addr} \| method=%{GREEDYDATA:method} \| response_status=%{INT:response_status}
```
Для отладки фильтров можно использовать [Grok Debugger](https://grokdebug.herokuapp.com/)

2. С помощью zipkin нашел баг в коде, который подвешивает post сервис на 3 секунды:
![](https://github.com/otus-devops-2019-05/vvorontsov_microservices/blob/logging-1/.github/zipkin.jpg)
```
# Retrieve information about a post
def find_post(id):
---
        app.post_read_db_seconds.observe(resp_time)
        time.sleep(3)
        log_event('info', 'post_find',
                  'Successfully found the post information',
                  {'post_id': id})
        return dumps(post)
```
## HomeWork #17 (monitoring-2)
- Отделил контейнеры с приложением и мониторингом:

   [docker-compose-monitoring.yml](https://github.com/otus-devops-2019-05/vvorontsov_microservices/blob/monitoring-2/docker/docker-compose-monitoring.yml)

   [docker-compose.yml](https://github.com/otus-devops-2019-05/vvorontsov_microservices/blob/monitoring-2/docker/docker-compose.yml)

- Добавил в мониторинг cAdvisor:
```
cadvisor:
    image: google/cadvisor:v0.29.0
    volumes:
      - '/:/rootfs:ro'
      - '/var/run:/var/run:rw'
      - '/sys:/sys:ro'
      - '/var/lib/docker/:/var/lib/docker:ro'
    ports:
      - '8080:8080'
    networks:
      - back_net
```
- Добавил в мониторинг Grafana:
```
grafana:
    image: grafana/grafana:5.0.0
    volumes:
      - grafana_data:/var/lib/grafana
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=secret
    depends_on:
      - prometheus
    ports:
      - 3000:3000
    networks:
      - back_net
```

- Добавил дашбоард [Docker and system monitoring](https://grafana.com/grafana/dashboards/893)

- Создал дашборд для мониторинга работы сервисов  [UI_Service_Monitoring.json](https://github.com/otus-devops-2019-05/vvorontsov_microservices/blob/monitoring-2/monitoring/grafana/dashboards/UI_Service_Monitoring.json)
- Создал дашборд для мониторинга бизнес-метрик [Business_Logic_Monitoring.json](https://github.com/otus-devops-2019-05/vvorontsov_microservices/blob/monitoring-2/monitoring/grafana/dashboards/Business_Logic_Monitoring.json)
- Добавил alertmanager
```
  alertmanager:
    image: ${USER_NAME}/alertmanager
    command:
      - '--config.file=/etc/alertmanager/config.yml'
    ports:
      - 9093:9093
    networks:
      - back_net
```
   Настроил интеграцию alertmanager со slack:
```
global:
  slack_api_url: 'https://hooks.slack.com/services/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'

route:
  receiver: 'slack-notifications'

receivers:
- name: 'slack-notifications'
  slack_configs:
  - channel: '#viktor_vorontsov'
```
   Настриол правило оповещения при недоступности любого сервиса:
 ```
 groups:
  - name: alert.rules
    rules:
    - alert: InstanceDown
      expr: up == 0
      for: 1m
      labels:
        severity: page
      annotations:
        description: '{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 1 minute'
        summary: 'Instance {{ $labels.instance }} down'
 ```
 - Все образы запушил в свой docker hub https://cloud.docker.com/u/xvikx
## HomeWork #16 (monitoring-1)
- Сконфигурирован и запущен Prometheus в докере:
```
# Dockerfile
FROM prom/prometheus:v2.1.0
ADD prometheus.yml /etc/prometheus/
```
```
# prometheus.yml
---
global:
  scrape_interval: '5s'

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets:
        - 'localhost:9090'

  - job_name: 'ui'
    static_configs:
      - targets:
        - 'ui:9292'

  - job_name: 'comment'
    static_configs:
      - targets:
        - 'comment:9292'

```
- Настроен сбор метрик хоста с использованием node экспортера:
```
  - job_name: 'node'
    static_configs:
      - targets:
        - 'node-exporter:9100' 

```
- Сбилдил образы и залил на docker hub:
https://cloud.docker.com/u/xvikx

#### Задание со *
1.Добавил в Prometheus мониторинг MongoDB с использованием [percona/mongodb_exporter](https://github.com/percona/mongodb_exporter)

- Взял текущую версию экспортера, сбилдил образ и залил в свой реп на docker hub:
https://cloud.docker.com/repository/docker/xvikx/mongodb-exporter

- В конфиг прометея добавил job:
```
  - job_name: 'mongodb'
    static_configs:
      - targets:
        - 'mongodb-exporter:9216'
```
- В docker-compose добавил запуск контейнера. Переменная `${MONGODB_HOST}` устанавливается в .env файле:
```
  mongodb-exporter:
    image: ${USER_NAME}/mongodb-exporter:latest
    environment:
      - MONGODB_URI=mongodb://${MONGODB_HOST}:27017
    networks:
      - back_net

```
2. Настроил blackbox мониторинг http запросов c помощью `prom/blackbox-exporter`
```
  blackbox-exporter:
    image: prom/blackbox-exporter:v0.14.0
    cap_add:
      - CAP_NET_RAW
    ports:
      - '9115:9115'
    networks:
      - back_net

```
- Добавил job для прометея:
```
  - job_name: 'blackbox'
    params:
      module: [http_2xx]  # Look for a HTTP 200 response.
    static_configs:
      - targets:
        - 'localhost:9090'
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: blackbox-exporter:9115

```

## HomeWork #15 (gitlab-ci-1)
- С помощью docker-machine создал VM для gitlab-ci:

```
NAME            ACTIVE   DRIVER   STATE     URL                        SWARM   DOCKER     ERRORS
docker-gitlab   -        google   Running   tcp://35.205.171.44:2376           v19.03.1   
```
- Создал docker-compose файл как описано в документации ([Install GitLab using docker-compose](https://docs.gitlab.com/omnibus/docker/README.html#install-gitlab-using-docker-compose))

- Установил docker-compose и запустил контейнер с gitlab:
```
CONTAINER ID        IMAGE                         COMMAND                  CREATED             STATUS                 PORTS                                                            NAMES
fea64adcf40d        gitlab/gitlab-ce:latest       "/assets/wrapper"        5 hours ago         Up 5 hours (healthy)   0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp, 0.0.0.0:2222->22/tcp   gitlab_web_1

```
- Поднял и зарегистрировал `gitlab-runner`:

```
1a2980824c29        gitlab/gitlab-runner:latest   "/usr/bin/dumb-init …"   5 hours ago         Up 5 hours                                                                              gitlab-runner
```
 
 - Создал проект и настроил CI:
 ```
 image: ruby:2.4.2

stages:
  - build
  - test
  - review
  - stage
  - production

variables:
  DATABASE_URL: 'mongodb://mongo/user_posts'

before_script:
  - cd reddit
  - bundle install

build_job:
  stage: build
  script:
    - echo 'Building'

test_unit_job:
  stage: test
  services:
    - mongo:latest
  script:
    - ruby simpletest.rb

test_integration_job:
  stage: test
  script:
    - echo 'Testing 2'

deploy_dev_job:
  stage: review
  script:
    - echo 'Deploy'
  environment:
    name: dev
    url: http://dev.example.com

branch review:
  stage: review
  script: echo "Deploy to $CI_ENVIRONMENT_SLUG"
  environment:
    name: branch/$CI_COMMIT_REF_NAME
    url: http://$CI_ENVIRONMENT_SLUG.example.com
  only:
    - branches
  except:
    - master

stage:
  stage: stage
  when: manual
  only:
    - /^\d+\.\d+\.\d+/
  script:
    - echo 'Deploy'
  environment:
    name: stage
    url: https://beta.example.com

production:
  stage: production
  when: manual
  only:
    - /^\d+\.\d+\.\d+/
  script:
    - echo 'Deploy'
  environment:
    name: production
    url: https://example.com

 ```
 

## HomeWork #14 (docker-4)
#### Самостоятельная работа 
- Протестировал работу всех [network drivers](https://docs.docker.com/network/):

`bridge`: The default network driver. If you don’t specify a driver, this is the type of network you are creating. Bridge networks are usually used when your applications run in standalone containers that need to communicate.

`host`: For standalone containers, remove network isolation between the container and the Docker host, and use the host’s networking directly. host is only available for swarm services on Docker 17.06 and higher. See use the host network.

`overlay`: Overlay networks connect multiple Docker daemons together and enable swarm services to communicate with each other. You can also use overlay networks to facilitate communication between a swarm service and a standalone container, or between two standalone containers on different Docker daemons. This strategy removes the need to do OS-level routing between these containers. See overlay networks.

`macvlan`: Macvlan networks allow you to assign a MAC address to a container, making it appear as a physical device on your network. The Docker daemon routes traffic to containers by their MAC addresses. Using the macvlan driver is sometimes the best choice when dealing with legacy applications that expect to be directly connected to the physical network, rather than routed through the Docker host’s network stack. See Macvlan networks.

`none`: For this container, disable all networking. Usually used in conjunction with a custom network driver. none is not available for swarm services. See disable container networking.

- Создал docker-compose для приложения reddit:
```
version: '3.3'
services:
  post_db:
    image: mongo:3.2
    volumes:
      - post_db:/data/db
    networks:
      - back_net
  ui:
    build: ./ui
    image: ${USER_NAME}/ui:${VERSION}
    ports:
      - ${HOST_PORT}:${SERVICE_PORT}/tcp
    networks:
      - front_net
  post:
    build: ./post-py
    image: ${USER_NAME}/post:${VERSION}
    networks:
      - back_net
      - front_net
  comment:
    build: ./comment
    image: ${USER_NAME}/comment:${VERSION}
    networks:
      - back_net
      - front_net
volumes:
  post_db:

networks:
  front_net:
  back_net:

```
Переменные подставляются из .env файла:
```
COMPOSE_PROJECT_NAME=dockermicroservices
USER_NAME=xvikx
HOST_PORT=9292
SERVICE_PORT=9292
VERSION=2.0
```
P.S. `COMPOSE_PROJECT_NAME` устанавливает префикс для контейнеров и сетей.

- Создал docker-compose.override.yml файл для переопределения volumes и запуска puma.
```
version: '3.3'
services:
  ui:
    command: ["puma", "--debug", "-w", "2"]
    volumes:
      - ./ui:/app

  post:
    volumes:
      - ./post-py:/app

  comment:
    command: ["puma", "--debug", "-w", "2"]
    volumes:
      - ./comment:/app

```
В [документации сказано](https://docs.docker.com/compose/extends/), что при запуске `docker-compose up`, переопределения применяются автоматически.

## HomeWork #13 (docker-3)
#### Самостоятельная работа 
- Приложение docker-monolith разбил на микросервисы:
```
src/
├── comment
│   ├── comment_app.rb
│   ├── config.ru
│   ├── docker_build.sh
│   ├── Dockerfile
│   ├── Gemfile
│   ├── Gemfile.lock
│   ├── helpers.rb
│   └── VERSION
├── post-py
│   ├── docker_build.sh
│   ├── Dockerfile
│   ├── helpers.py
│   ├── post_app.py
│   ├── requirements.txt
│   └── VERSION
├── README.md
└── ui
    ├── config.ru
    ├── docker_build.sh
    ├── Dockerfile
    ├── Gemfile
    ├── Gemfile.lock
    ├── helpers.rb
    ├── middleware.rb
    ├── ui_app.rb
    ├── VERSION
    └── views
        ├── create.haml
        ├── index.haml
        ├── layout.haml
        └── show.haml

```
- Для каждого микросервиса был создан свой Dockerfile и произведен билд образов(v1.0).
```
xvikx/post          1.0                 1e105ac99f0b        14 hours ago        216MB
xvikx/ui            1.0                 0d68083a4902        37 hours ago        781MB
xvikx/comment       1.0                 f6f9ae9d3ac1        37 hours ago        779MB
```
- Создан volume reddit_db сохранности данных после рестарта контейнера:

```
docker volume create reddit_db
docker run -d --network=reddit --network-alias=post_db --network-alias=comment_db -v reddit_db:/data/db --name=mongo mongo:latest
```

- Для проверки Dockerfile использовал hadolint:

 `docker run --rm -i hadolint/hadolint < Dockerfile`

#### Задание со *
- Использование env переменных в `docker run`:
```
docker run -d --network=reddit \
    --network-alias=mongo_db \
    mongo:latest
docker run -d --network=reddit \
    --network-alias=post_src \
    -e POST_DATABASE_HOST=mongo_db \
    xvikx/post:1.0
docker run -d --network=reddit \
    --network-alias=comment_src \
    -e COMMENT_DATABASE_HOST=mongo_db \
    xvikx/comment:1.0
docker run -d --network=reddit \
    -e POST_SERVICE_HOST=post_src \
    -e COMMENT_SERVICE_HOST=comment_src \
    -p 9292:9292 xvikx/ui:1.0
```
- Оптмизировал все Dockerfile. Использовал alpine как базовый образ и добавил удаление кэша после установки пакетов:
```
FROM alpine:3.9

ENV APP_HOME /app
RUN mkdir $APP_HOME

WORKDIR $APP_HOME
COPY . $APP_HOME

RUN apk update && apk add --no-cache build-base ruby-full ruby-dev ruby-bundler \
    && gem install bundler --no-ri --no-rdoc \
    && bundle install \
    && rm -rf /var/cache/* \
    && rm -rf /root/.cache/*

ENV POST_SERVICE_HOST post
ENV POST_SERVICE_PORT 5000
ENV COMMENT_SERVICE_HOST comment
ENV COMMENT_SERVICE_PORT 9292

CMD ["puma"]

```
Размер образов после оптимизации:
```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
xvikx/ui            2.0                 ca3001222619        13 hours ago        235MB
xvikx/post          2.0                 ae274849cd7d        13 hours ago        268MB
xvikx/comment       2.0                 97c5e960d780        13 hours ago        232MB
```

## HomeWork #12 (docker-2)
#### Самостоятельная работа 
- Установил docker-ce и docker-machine.
```
Docker version 18.09.5, build e8ff056
docker-machine version 0.16.0, build 702c267f
```
- В GCP создал новый проект docker-248610 и настроил аккаунт для подключения с хоста.
- Создал Dockerfile для билда образа с приложением reddit:
```
FROM ubuntu:16.04

RUN apt-get update
RUN apt-get install -y mongodb-server ruby-full ruby-dev build-essential git
RUN gem install bundler
RUN git clone -b monolith https://github.com/express42/reddit.git

COPY mongod.conf /etc/mongod.conf
COPY db_config /reddit/db_config
COPY start.sh /start.sh

RUN cd /reddit && bundle install
RUN chmod 0777 /start.sh

CMD ["/start.sh"]

```
С помощью docker-machine был создан docker-host в GCP:
```
docker-machine create --driver google \
 --google-machine-image https://www.googleapis.com/compute/v1/projects/ubuntu-os-cloud/global/images/family/ubuntu-1604-lts \
 --google-machine-type n1-standard-1 \
 --google-zone europe-west1-b \
 docker-host
```

```
# docker-machine ls             
NAME          ACTIVE   DRIVER   STATE     URL                       SWARM   DOCKER     ERRORS
docker-host   *        google   Running   tcp://34.76.185.68:2376           v19.03.1   
```

На этом хосте сбилдил образ reddit:
```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
reddit              latest              592662396b18        3 hours ago         685MB
```

- Зарегистрировался на dockerhub и запушил туда образ xvikx/otus-reddit
