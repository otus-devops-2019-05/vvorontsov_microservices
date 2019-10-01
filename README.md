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
  tls.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUREekNDQWZlZ0F3SUJBZ0lVREdraWpKL3U3aFQyZk94SFdxQXZIWUNHb3dFd0RRWUpLb1pJaHZjTkFRRUwKQlFBd0Z6RVZNQk1HQTFVRUF3d01NelF1T1RZdU56RXVNVE15TUI0WERURTVNVEF3TVRFME5EY3dNMW9YRFRJdwpNRGt6TURFME5EY3dNMW93RnpFVk1CTUdBMVVFQXd3TU16UXVPVFl1TnpFdU1UTXlNSUlCSWpBTkJna3Foa2lHCjl3MEJBUUVGQUFPQ0FROEFNSUlCQ2dLQ0FRRUFyazkwNGIvSDFUQWt4Y3J5OEtPZXJLUXNZWitaV1FRNmx3eSsKMnpFejdSdGptamUwanFhVTNoU2xFcDlQdzVXdHF3cGUwV3NQK2JKVHFqVlE2TXNQNWo0T1VCMVllbTdhTlRpawpTUjQ0QVZlbFVNK0JmVW9pTkNmK1FDUDZpSUNFbDZpSnZ1TFI1NUtBOUs4QTBIalhNdk4vcDUyWHoxU1NDMnJpCjVKV0srZm50Zm9KL0VBRTl2RTU5ZTVBemxMSUxqbklNcUQ5NmFnUWJvTUdXdHlScnBSeXlkbkdIWHJKSEtHVHUKbVhWdlN1RVA0eVVCY3FWQi8rSnZCSnBacWg4amlTTEF4L2lIaW8xTXlEclpQMU4vZFo3UUIrMXdjb3NLaG9XUwpqa21yQzJ1c05JYmxtTGdra0FUUEs1cHY3M2pHNmFjeW52cXZFS0dwdG9DUjNEY3J4UUlEQVFBQm8xTXdVVEFkCkJnTlZIUTRFRmdRVWNMQlNvNFdVNGdnTkIvVHJVOFgvaXllOGNlUXdId1lEVlIwakJCZ3dGb0FVY0xCU280V1UKNGdnTkIvVHJVOFgvaXllOGNlUXdEd1lEVlIwVEFRSC9CQVV3QXdFQi96QU5CZ2txaGtpRzl3MEJBUXNGQUFPQwpBUUVBUjY5aEdtSFVDM2FjTE1tZWZjeUJyOUxEaThWS3Fqci83VXd6K2RNR2Q5OWd6VXd5TXpjQUlpOTA4VHhlCjNDcnZKdDkxQU5tZ2o4K1J4dkI4ZUVBeVlEajQ2aWI4Vjg5UTBHcFZLM0JZMytaR1pkZWlQYU14T1VaUWtTSmMKSVQyY1RFSmNOc3pKSEpFQ1hCQ25CWmtFOStDTGp1WWZNVVJEOTI5QXJrMUZzTkdTQ2ZJbnNocE5sc0xhNjJ0UApHc24rcUVET2JoUGFkeFRzVnNqZ0tsem5ETzZNbHl6LzMrZ2E3TWhGWW9BeEhVOFFUa2E0WERSeUxQZUNrdUVSCmNuWXNxZkE4dlFvaGQ3ZHh2U1VRZjFHeVpZRkpJZlFYZnczdFo2VmN3aVkvNnZGWjNPaCttU0RJVTVhSkFUK2MKUTNkR25RTFhUOGt3clVlVVJoRWUzTUY1TXc9PQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
  tls.key: LS0tLS1CRUdJTiBQUklWQVRFIEtFWS0tLS0tCk1JSUV2UUlCQURBTkJna3Foa2lHOXcwQkFRRUZBQVNDQktjd2dnU2pBZ0VBQW9JQkFRQ3VUM1RodjhmVk1DVEYKeXZMd281NnNwQ3hobjVsWkJEcVhETDdiTVRQdEcyT2FON1NPcHBUZUZLVVNuMC9EbGEyckNsN1Jhdy81c2xPcQpOVkRveXcvbVBnNVFIVmg2YnRvMU9LUkpIamdCVjZWUXo0RjlTaUkwSi81QUkvcUlnSVNYcUltKzR0SG5rb0QwCnJ3RFFlTmN5ODMrbm5aZlBWSklMYXVMa2xZcjUrZTErZ244UUFUMjhUbjE3a0RPVXNndU9jZ3lvUDNwcUJCdWcKd1phM0pHdWxITEoyY1lkZXNrY29aTzZaZFc5SzRRL2pKUUZ5cFVILzRtOEVtbG1xSHlPSklzREgrSWVLalV6SQpPdGsvVTM5MW50QUg3WEJ5aXdxR2haS09TYXNMYTZ3MGh1V1l1Q1NRQk04cm1tL3ZlTWJwcHpLZStxOFFvYW0yCmdKSGNOeXZGQWdNQkFBRUNnZ0VBSi9MMTZyYzFhVnE3VXNtTW5ESWpyNVdBeG03NWlqazU1Rng3Y1dqNWFhSVoKVmNMbWtyc0M2MUwzOGlpbzh6NWVxeENaWW8yUy83c3JDWnRtaTdQZVZQcHI5VmlFYXFyMVA3VlhrWnBuWTRkZQowQUordnVRNnFwRnY1K3RYQStuUWVhR2EwRERUd3FzRSt3OFF0TGE5TzJRcklaTXdzOGZDSVBQQ1JIa3hmTkMxCjNiQVdpLzdHSEZCRHFPRU10bDl5L0UrclljQXJLcnZmc1h3R0NZVHZFQ0pWSVRqdTQ3NWkza3h5eWt3aXhDNFEKbmU2dVNHdExrK2NlQTdLRlhqWDRYWmpwK1cveUhEL2NVS1A2VFBTS3BsMTRYUjFhM2M1b1JwNmsxYmRKVGFiMgpYUmFXTE5kbHBrMzduWDJOZ1JlWUJzWTR0Sm53UGNBTlRqM1lWb29TRVFLQmdRRGg1MWd4eTY0TjI1bTFPK0c3CnZJQURzS0E4aTg2UFYzcC9JRFF5TkdPbU9ISGFEeHdnVHRncWg3aVAwYVRNVzFZeXNkaUMxTm9kWHBxOS9IUlQKMkNFUUdIWFhLVTQ1TXplb2xTbXpMNHhuNGplejJ2elJpcTVXOTIzNjVsS0YyeGJ4YWlMZlBpaEt1RENTYlZ2UQpIejF5bE1BL2xNZHBtNmhyYlFJdWVubEJmd0tCZ1FERmlIbGh0OXhBcWlBRjdhZHJQZWt5STdrQWoxTXRqLzlQClBHR2h4a082RnM5dkpkbUdlcnJYd2lqRnYwK3lyYzZNaW1xZjVzaHFoaG1qNU9SODE3c0Q1MGVLNGpIWTN2aUEKOGE1eXZsUFFTRmJXZExNZ1lzTGxQZGhWcCtVcDRINGNzS3daWW5HSjJBb3pJNnVXRmwycndzTmUySVgwUTlCYQptKzdFSGQyc3V3S0JnQTI5VmwzZzkzc2NTaUw5dTJNQnVmOS9kSjk2R1Z3YWcvYkxiS0ZKRW01L3JGMEk0anNNCjBKVDFvRUlQRENqcHZUcGtHcmtLWFNIQUtVVTQzNUpoNk5EankvY0VLaC9NZjZ3Zk5tUVJsa2FUT2JRVXM1L0QKQVl1RWpRbmZqRkZiMis5ZTl6UUF3YzZabVVxdW9CRHVkWHhNazh6S0xiNjhtdWU5djQ0NElMdmRBb0dCQUxMOAp6ci8wMGVibVFNNGVZaTJTazlPUyt2ZSs1eWZKNnhYcEtLNWw4TWlXRXJBc0k4YnZQbzV5cUc5R3d2aXM5UlB6CitGbWJ6TTU1WkpKVnZaUkNCbnVxL2ZDaXRYaEYwZmRGQjBXQm9JQ0NpKzBYSVppZTVPckQ5MXJtSDRpcW1wdDAKbXYwRmJzdndybzFZTFFwNmliWXhiTVpzZkRTeG5nSDhlWVhMYWlveEFvR0FMYjEyM3UxMFpIbjlzVEx6VWxyMwpKOHNoM1BuL3BxUHovaFlRRHExKzFMTSs0UmthcEtmeUFIMmhkNysvNWt1TTJ4UUhEWVNXczViREt6ZTh4dVVuClUxMmRMWHFSYm8rRnVwejhvNkRiQmNCWXFWMmZVYXlPVzI2YVpSNlcyKzgvakpkdkJwaVhWRFoyK2wvVFlBOXEKSEJ2K0F0UW1PTGhUS084TWxhRkhsOVU9Ci0tLS0tRU5EIFBSSVZBVEUgS0VZLS0tLS0K

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
