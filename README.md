
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
