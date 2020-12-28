# DockerCompose

## docker-compose 配置文件
模板文件是使用 Compose 的核心，默认的模板文件名称为 docker-compose.yml，格式为 YAML 格式。
```shell script
version: "3.2"
services:
  webapp:
    image: examples/web
    ports:
      - "80:80"
    volumes:
      - "/data"
```

> `version` 为模板文件的版本，`services` 为一系列容器组成的服务，服务下面就是相关的配置指令了。

YAML 文件格式语法说明：
- 大小写敏感
- 使用缩进表示层级关系
- 缩进时不允许使用Tab键，只允许使用空格
- 缩进的空格数目不重要，只要相同层级的元素左侧对齐即可

### 配置指令

#### image 
指定服务所使用的镜像。如果镜像不存在，Compose将尝试从互联网拉取这个镜像。
```shell script
image: ubuntu
image: mysql:5.7
```

#### build 
指定 Dockerfile 所在文件夹的路径，意味着服务所使用的竟像是本地构建的而非拉取的。
```shell script
build:
  context: ./nginx
  dockerfile: Dockerfile
  args:
    - buildno=1
    - user=someuser
```
#### container_name
指定容器名称。默认将会使用`项目名称_服务名称_序号`这样的格式。
```shell script
container_name: myNginx
```
注意：
> 指定容器名称后，该服务将无法进行扩展，因为 Docker 不允许多个容器实例重名。


#### link
链接到其他服务容器，使用服务名称(同时作为别名)或服务别名（SERVICE:ALIAS）都可以。
```shell script
links:
 - nginx:nginx
 - db:database
 - redis
```

#### external_links
链接到 docker-compose.yml 外部的容器，甚至并非是Compose管理的容器。参数格式和 links 类似。
```shell script
external_links:
 - redis_1
 - project_db_1:mysql
 - project_php_1:php
```

#### expose 
暴露端口供外部服务连接使用，仅可以指定内部端口为参数，跟 Dockerfile 中 expose 一个作用。
```shell script
expose:
    - "6379"
    - "8080"
```

#### ports 
端口绑定，【HOST:CONTAINER】格式或者【仅仅指定容器的端口】（宿主将会随机选择端口）都可以。
> 如果你使用的容器端口小于 60 你可能会得到错误得结果，因为 YAML 将会解析 xx:yy 
> 这种数字格式为 60 进制。所以建议采用字符串格式。
```shell script
ports:
    - "3000"
    - "3000-3005"
    - "80:80"
    - "9090-9091:8080-8081"
    - "49100:22"
    - "127.0.0.1:8001:8001"
    - "127.0.0.1:5000-5010:5000-5010"
```

#### environment 
设置环境变量。你可以使用数组或字典两种格式。
如果只给定变量的名称则会自动加载它在Compose宿主机上的值，可以用来防止泄露不必要的数据。
```shell script
environment:
  RACK_ENV: development
  SESSION_SECRET:

environment:
  - RACK_ENV=development
  - SESSION_SECRET
```
注意：
> 如果变量名称或者值中用到 true/false/yes/no 等表达布尔含义的词汇，最好放到引号里，避免 YAML 自动解析某些内容为对应的布尔语义。

#### env_file
从文件中获取环境变量，可以为单独的文件路径或列表。
如果通过 docker-compose -f FILE 指定了模板文件，则 env_file 中路径会基于模板文件路径。
如果有变量名称与 environment 指令冲突，则以后者为准。
```shell script
env_file: .env

env_file:
  - ./common.env
  - /data/secrets.env
```

#### volumes
将本机的文件夹挂载到container中。可以设置【host:container】，还可以加上访问模式【host:container:ro】。
```shell script
volumes:
 - /var/lib/mysql
 - cache/:/tmp/cache
 - ~/configs:/etc/configs/:ro
```

#### volumes_from
挂载另一个服务或容器的所有数据卷。
```shell script
volumes_from:
 - service_name
 - container_name
```

#### command 
覆盖容器启动后默认执行的命令。可以为字符串格式或 JSON 数组格式。
```shell script
nginx:
    command: nginx -g 'daemon off;'
    command: [ "bash", "-c", "echo", "hello world"]
```

#### entrypoint
覆盖容器中默认的人口命令。注意，也会取消掉镜像中指定的人口命令和默认启动命令。
例如，覆盖为新的入口命令：
```shell script
entrypoint : python app.py
```

#### extends
基于已有的服务进行扩展。例如我们已经有了一个webapp服务，模板文件为 common.yml。
```shell script
# common.yml
webapp:
  build: ./webapp
  environment:
    - DEBUG=false
    - SEND_EMAILS=false
```
编写一个新的 development.yml 文件，使用 common.yml 中的 webapp 服务进行扩展。
```shell script
# development.yml
web:
  # 继承服务
  extends:
    file: common.yml
    service: webapp
  ports:
    - "8000:8000"
  links:
    - db
  # 重写变量
  environment:
    - DEBUG=true
db:
  image: postgres
```
注意： 
> extends不会继承 links、depends_on、volumes_from 中定义的容器和数据卷资源。

一般情况下， 推荐在基础模板中只定义一些可以共享的镜像和环境变量， 在扩展模板中
具体指定应用变量、链接、数据卷等信息。

#### network_mode
设置网络模式。使用和 docker client 的 --net 参数一样的值。
```shell script
net: "bridge"
net: "none"
net: "container:[name or id]"
```

#### networks
所加入的网络，需要在顶级的 networks 宇段中定义具体的网络信息。
```shell script
services:
  web:
    networks:
      web_net:
        aliases : web_app
        ipv4_address: 172.16.0.10
networks:
  web_net:
    driver: bridge
    enable_ipv6: true
    ipam:
      driver: default
      config: 
        subnet: 172.16.0.0/24              
```

#### logging
跟日志相关的配置，包括一系列子配置。
```shell script
logging:
  driver: "syslog"
  options:
      syslog_address:"tcp: //192.168.0.42:123"
```
driver 类似于 Docker 中的 --log-driver 参数，目前有三种：`json_file`、`syslog`、`none`。

#### healthcheck
指定检测应用健康状态的机制， 包括检测方法(test)、间隔( interval)、超时(timeout)、
重试次数(retries)、启动等待时间(start_period)等。

例如指定检测方法为访问 8080 端口， 间隔为30秒， 超时为15秒，重试3次，启动后等待30秒再做检查。
```shell script
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhos七： 8080"]
  interval: 30s
  timeout: 15s
  retries: 3
  start_period: 30s
```

#### pid
跟主机系统共享进程命名空间。打开该选项的容器可以相互通过进程 ID 来访问和操作。
```shell script
pid: "host"
```

#### dns
配置 DNS 服务器。可以是一个值，也可以是一个列表。
```shell script
dns: 8.8.8.8
dns:
  - 8.8.8.8
  - 9.9.9.9
```

#### dns_search
配置 DNS 搜索域，可以是一个值，也可以是一个列表。
```shell script
dns_search: example.com
dns_search:
  - domain1.example.com
  - domain2.example.com
```

#### depends_on 
依赖的服务，用来控制服务的先后启动顺序。
```shell script
nginx:
    depends_on:
      - php-fpm
```

#### cap_add, cap_drop
添加或放弃容器的 Linux 能力（Capabiliity）。
```shell script
cap_add:
  - ALL

cap_drop:
  - NET_ADMIN
  - SYS_ADMIN
```

#### 其它选项
working_dir、 entrypoint、user、hostname、domainname、mem_limit、privileged、restart、stdin_open、tty、cpu_sharesv 这些都是和 docker run 支持的选项类似。
```shell script
cpu_shares: 73
working_dir: /code
entrypoint: /code/entrypoint.sh
user: postgresql
hostname: foo
domainname: foo.com
mem_limit: 1000000000
privileged: true
restart: always/no/on-failure/unless-stopped
stdin_open: true
tty: true
```

#### 读取环境变量
Compose模板文件支持动态读取主机的系统环境变量，例如，下面的Compose文件将从运行它的环境中
读取变量${MONGO_VERSION}的值（不指定时则采用默认值3.2), 并写入执行的指令中。
```shell script
db:
  image: "mango:${MONGO_VERSION-3.2}"
```

### 验证配置文件语法
可以通过 config 命令来检查配置文件的语法问题，如果你的配置文件不是默认的 docker-compose.yml，那么就需使用 `-f` 来指定配置文件。
```shell script
docker-compose -f {yourCompose.yml} config
```

## docker-compose 容器管理

### 项目管理
启动项目：
```shell script
docker-compose up
```
还可以以守护模式启用，所有容器将后台运行：
```shell script
docker-compose up -d
```

如果用户只想重新部署某个服务，并不想影响到其所依赖的服务，可以这样：
```shell script
docker-compose up --no-deps -d <SERVICE_NAME> 
```

停止项目：
```shell script
docker-compose down
```

重新构建项目：
> build 常用于修改了 Dockerfile 后重新构建项目，功能类似于 `docker build .`

```shell script
docker-compose build
```


关于 `up` 和 `down`:
> 这两个命令十分强大，以前者为例，它将尝试自动完成包括`构建镜像`，`重新创建服务`，`启动服务`,
> 并关联服务相关容器的一系列操作.


### 服务管理
停止服务,如果不指定 service 则默认停止所有的容器：
```shell script
docker-compose stop [options] [SERVICE...]
```

启动服务,用法跟上面的 stop 刚好相反：
```shell script
docker-compose  start [options] [SERVICE...]
```

重启服务,用法跟上面的 stop,start 一样:
```shell script
docker-compose  restart [options] [SERVICE...]
```

查看服务日志：
```shell script
docker-compose logs [options] [SERVICE...]
```

运行容器中的进程：
```shell script
docker-compose run --rm -w {project} composer install
```

### 容器和镜像
查看 docker-compose 生成的镜像：
```shell script
docker-compose images
```
 

查看 docker-compose 生成的容器：
```shell script
docker-compose ps 
```

删除容器:
```shell script
docker-compose rm {container/Id}
```

docker-compose 生成的镜像和容器与 docker生成的有什么区别?
```shell script
没有区别,只不过对于生成的镜像和容器,docker-compose 有一套默认的命名规则.
```

docker-compose 默认的命名规则:
```shell script
遵循 {project}_{service} 的规则,project 为 docker-compose 配置文件所在的目录,
service为配置文件中的服务,你还可以在配置文件通过 container_name 来自定义容器名称.
```


如何进入 docker-compose 生成的容器?
```shell script
docker exec -it {container/Id} bash
```

## 参考
- https://www.cnblogs.com/saneri/p/12851729.html
- https://yeasy.gitbook.io/docker_practice/compose/compose_file





