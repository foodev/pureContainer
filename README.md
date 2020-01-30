# pureContainer

## Start containers

PHP-FPM
```
# docker run --rm php-fpm:7.4.2
```

nginx
```
# docker run --rm --publish 80:80 --name nginx --network foodev nginx:1.16.1
```
