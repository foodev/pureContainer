# pureContainer

## Create custom network

A custom network is necessary so that the containers can reach each other via their hostname (the name of the container).

```
$ docker network create custom_network
```

## Start containers

PHP-FPM
```
# docker run --rm php-fpm:7.4.2
```

nginx
```
$ docker run --rm --publish 80:8080 --publish 443:44300 --name nginx --network custom_network --volume `pwd`:/srv/http:ro nginx:1.16.1
```
