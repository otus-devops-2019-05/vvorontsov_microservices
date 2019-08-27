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
