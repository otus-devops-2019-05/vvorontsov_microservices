FROM alpine:3.9

WORKDIR /app
COPY . /app

RUN apk add --no-cache build-base gcc musl-dev python3 python3-dev  \
    && pip3 install -r /app/requirements.txt \
    && rm -rf /var/cache/* \
    && rm -rf /root/.cache/*

ENV POST_DATABASE_HOST post_db
ENV POST_DATABASE posts

ENTRYPOINT ["python3", "post_app.py"]
