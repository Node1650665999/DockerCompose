## docker-compose 配置文件
yml 文件语法注意事项：
- 大小写敏感
- 使用缩进表示层级关系
- 缩进时不允许使用Tab键，只允许使用空格
- 缩进的空格数目不重要，只要相同层级的元素左侧对齐即可



```shell script
image 
    指定为镜像名称或镜像ID。如果镜像不存在，Compose将尝试从互联网拉取这个镜像

build 
    指定Dockerfile所在文件夹的路径。Compose将会利用他自动构建这个镜像，然后使用这个镜像

command 
     覆盖容器启动后默认执行的命令。

links 
     链接到其他服务容器，使用服务名称(同时作为别名)或服务别名（SERVICE:ALIAS）都可以

external_links 
    链接到docker-compose.yml外部的容器，甚至并非是Compose管理的容器。参数格式和links类似

ports 
    端口绑定。[宿主机器端口:容器端口]格式或者仅仅指定容器的端口（宿主机器将会随机分配端口）都可以

expose 
  暴露端口，与posts不同的是expose只可以暴露端口而不能映射到主机，只供外部服务连接使用；仅可以指定内部端口为参数

volumes 
  设置卷挂载的路径。可以设置宿主机路径:容器路径（host:container）或加上访问模式（host:container:ro）ro就是readonly的意思，只读模式



depends_on 
    依赖的服务，用来控制服务的先后启动顺序
environment 设置环境变量。可以属于数组或字典两种格式。如果只给定变量的名称则会自动加载它在Compose主机上的值，可以用来防止泄露不必要的数据
env_file 从文件中获取环境变量，可以为单独的文件路径或列表。如果通过docker-compose -f FILE指定了模板文件，则env_file中路径会基于模板文件路径。如果有变量名称与environment指令冲突，则以后者为准(环境变量文件中每一行都必须有注释，支持#开头的注释行)

net 设置网络模式。使用和docker client 的 --net 参数一样的值
dns 配置DNS服务器。可以是一个值，也可以是一个列表
dns_search 配置DNS搜索域。可以是一个值也可以是一个列表
volunes_from ：  挂载另一个服务或容器的所有数据卷
extends 基于已有的服务进行服务扩展。例如我们已经有了一个webapp服务，模板文件为common.yml。编写一个新的 development.yml 文件，使用 common.yml 中的 webapp 服务进行扩展。后者会自动继承common.yml中的webapp服务及相关的环境变量
pid 和宿主机系统共享进程命名空间，打开该选项的容器可以相互通过进程id来访问和操作
cap_add,cap_drop 添加或放弃容器的Linux能力（Capability）
```

## docker-compose 容器管理
### 命令说明

验证配置文件的格式：
```shell script
# 如果你的配置文件不是默认的 docker-compose.yml,那么就需使用 -f 来指定配置文件.
docker-compose -f {yourCompose.yml} config
```


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
```shell script
# 当你修改了 Dockerfile 后,此时你可能想重新构建项目,功能类似于 docker build .
docker-compose build
```


关于 `up` 和 `down`:
> 这两个命令十分强大, 以前者为例,它将尝试自动完成包括构建镜像,重新创建服务,启动服务,
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
> docker-compose images 

查看 docker-compose 生成的容器：
> docker-compose ps 

删除容器:
> docker-compose rm {container/Id}

docker-compose 生成的镜像和容器与 docker生成的有什么区别?
> 没有区别，只不过对于生成的镜像和容器,docker-compose 有一套默认的命名规则。

docker-compose 默认的命名规则:
> 遵循 {project}_{service} 的规则,project 为 docker-compose 配置文件所在的目录，
> service为配置文件中的服务,当然,你还可以在配置文件通过 container_name 来自定义容器名称。

如何进入 docker-compose 生成的容器?
> docker exec -it {container/Id} bash





