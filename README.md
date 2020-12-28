# docker-compose 

关于docker-compose 的教程见[这里](./compose-tutorial.md)。

## 环境变量
在本项目根目录下每个单独的服务都已拆成独立的 compose-xxx.yml 文件。

例如 compose-nginx.yml :
```shell script
version: '3.2'
services:
  nginx:
    build:
      context: ./nginx
      args:
        - NGINX_IMAGE=${NGINX_IMAGE}
    volumes:
      - ${WWW_LOCAL_DIR}:/data/www
      - ${NGINX_LOG}:/var/log/nginx
      - ${NGINX_CONF}:/etc/nginx/nginx.conf
      - ${NGINX_VHOST}:/etc/nginx/conf.d
    working_dir: /data/www
    expose:
      - ${NGINX_HTTP_PORT}
      - ${NGINX_HTTPS_PORT}
    ports:
      - "${NGINX_HTTP_PORT}:80"
      - "${NGINX_HTTPS_PORT}:443"
    restart: always
    command: nginx-debug -g 'daemon off;'
```

模板文件中读取了环境变量，这些环境变量可以在项目根目录下的 [.env](./.env) 文件中设置，
根据自己身的项目目录自行修改。
```shell script
$ cat .env

# common
TZ="Asia/Shanghai"
WWW_LOCAL_DIR="./data/www"

# Mysql
MYSQL_IMAGE=mysql:5.7
MYSQL_DATA="./mysql/data"
MYSQL_CONF="./mysql/conf"
MYSQL_PORT=3307
MYSQL_ROOT_PASSWORD=123456

# Nginx
nginx_image=nginx:1.19
NGINX_LOG="./nginx/logs"
NGINX_CONF="./nginx/conf/nginx.conf"
NGINX_VHOST="./nginx/conf/vhost"
NGINX_HTTP_PORT=81
NGINX_HTTPS_PORT=443

# PHP
PHP_IMAGE=php:7.2-fpm
PHP_CONF="./php/conf"
PHP_FPM_PORT=9000

# Redis
REDIS_IMAGE=redis:6.0
REDIS_PORT=6379

```

运行服务：
```shell script
docker-compose --env-file .env  -f compose-nginx.yml up -d
```

注意：
- 环境变量文件 .env 你可以自定义，只要启动服务时通过 --env-file 指定它就可以了。
- 默认情况 docker-compose 会默认读取项目的下 docker-compose.yml 文件，但如果你的文件名不是默认的，
就需要通过 -f 来指定。


## 服务组合
由于 docker-compose 支持服务继承，这样我们基于单个 compose 服务，通过继承组合的方式就可以构造出一个聚合服务。

例如构建一个 [lnmp](./compose-lnmp.yml) 服务：
```shell script
version: '3.2'
services:
  php:
     extends:
       file: compose-php.yml
       service: php

  nginx:
    extends:
      file: compose-nginx.yml
      service: nginx
    depends_on:
      - php
    links:
      - php

  mysql:
    extends:
      file: compose-mysql.yml
      service: mysql
``` 

接着我们启动它，这样我们就构建了一套`lnmp`服务了：

```shell script
$ docker-compose --env-file .env -f compose-lnmp up -d

         Name                       Command               State                        Ports
------------------------------------------------------------------------------------------------------------------
docker_compose_mysql_1   docker-entrypoint.sh --cha ...   Up      0.0.0.0:3307->3306/tcp, 33060/tcp, 3307/tcp
docker_compose_nginx_1   /docker-entrypoint.sh ngin ...   Up      0.0.0.0:443->443/tcp, 0.0.0.0:81->80/tcp, 81/tcp
docker_compose_php_1     docker-php-entrypoint php-fpm    Up      0.0.0.0:9000->9000/tcp

```




