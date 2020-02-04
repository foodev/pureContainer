# pureContainer

## Create custom network

A custom network is necessary so that the containers can reach each other via their hostname (the name of the container).

```
$ docker network create custom_network
```

You can use any name instead of `custom_network` but then you have to change it in the `docker run ...` commands accordingly.

## Start containers

PHP-FPM
```
$ docker run --rm \
    --name php-fpm \
    --network custom_network \
    --volume `pwd`:/scripts:ro \
    php-fpm:7.4.2
```

nginx
```
$ docker run --rm \
    --name nginx \
    --network custom_network \
    --volume `pwd`:/srv/http:ro \
    --publish 80:8080 \
    --publish 443:44300 \
    nginx:1.16.1
```
