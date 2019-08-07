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
