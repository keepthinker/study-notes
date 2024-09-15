## Docker Command

### 一般操作

```bash
# 查看docker运行情况
docker stats
docker stats c6425e8f0695
# pull a image
docker pull ubuntu
# 以交互式界面来创建和启动容器，如下可以在docker容器中输入命令行
docker run -it ubuntu /bin/bash
# 采用-d，那么会将容器放在后台执行，若要进入容器，则需要用docker exec, --name设置容器名称
docker run -i -t -d --name ubuntu-test ubuntu /bin/bash
# 重命名容器
docker rename CONTAINER NEW_NAME
# 对已经启动的容器执行命令
docker exec -i -t d8b0c82491a6 /bin/bash
# -d:让容器在后台运行。
# -P:将容器内部使用的网络端口随机映射到我们使用的主机上。
docker run -d -P training/webapp python app.py
# 映射容器外的端口20080到容器内的端口80
docker run -d -p 127.0.0.1:20080:80 centos-nginx-01 /sbin/nginx -c /etc/nginx.conf
# 默认都是绑定 tcp 端口，如果要绑定 UDP 端口，可以在端口后面加上 /udp。
docker run -d -p 127.0.0.1:5000:5000/udp centos-nginx-01 python app.py
# 设置环境变量https://docs.docker.com/reference/cli/docker/container/run/#env
docker run -e MYVAR1 --env MYVAR2=foo --env-file ./env.list ubuntu bash
# 尝试测试dockerfile，可以参考如下名
docker run --rm -i -t --entrypoint /bin/sh --name test-dockerfile-1 --env-file config.properties test-image-name
# 查看进程映射的端口
docker port bf08b7f2cd89
# 查看进程日志，-f: 让 docker logs 像使用 tail -f 一样来输出容器内部的标准输出。
docker logs -f bf08b7f2cd89
# 查看正在活动的docker容器
docker ps 
# 显示容器所有信息
docker ps -a --no-trunc
# 查看容器内进程详情
docker top wizardly_chandrasekhar
# 使用 docker inspect 来查看 Docker 的底层信息，比如容器的配置和状态信息
docker inspect bf08b7f2cd89
# 从容器拷贝文件到宿主机
docker cp mycontainer:/opt/testnew/file.txt /opt/test/
# 从宿主机拷贝文件到容器
docker cp /opt/test/file.txt mycontainer:/opt/testnew/
```

### 起停服务

```bash
# 关闭docker服务
systemctl stop docker
# 开启docker服务
systemctl start docker
systemctl enable docker

# 启动docker
service docker start
# 停止docker 
service docker stop
# 重启docker
service docker restart
# 或者是
systemctl restart docker


# The docker stop commands issue the SIGTERM signal，其中非run启动命令，重启会消失。
docker stop 61e5fddeed45
docker start 61e5fddeed45
doker restart 61e5fddeed45
##  the docker kill commands sends the SIGKILL signal.
docker kill 9f215ed0b0d3
# The docker pause command suspends all processes in the specified containers.
# On Linux, this uses the cgroups freezer. Traditionally, when suspending a process 
# the SIGSTOP signal is used, which is observable by the process being suspended
# 后续通过docker exec 执行的命令，假如还在运行那么会保留。
docker pause 9f215ed0b0d3
docker unpause 9f215ed0b0d3
# 进入镜像，Use docker attach to attach your terminal's standard input, output, and 
# error (or any combination of the three) to a running container using the container's ID or name.
# 一般不适用该命令，用上述的docker exec来替换
docker attach 61e5fddeed45

# 导出容器
docker export 1e560fca3906 > ubuntu.tar
# 可以使用 docker import 从容器快照文件中再导入为镜像
cat docker/ubuntu.tar | docker import - test/ubuntu:v1
# 导入容器快照，可以通过指定 URL 或者某个目录来导入
docker import http://example.com/exampleimage.tgz example/imagerepo
# 删除容器
docker rm -f 1e560fca3906
# 下面的命令可以清理掉所有处于终止状态的容器。
$ docker container prune
```

### 容器备份、恢复和删除

```bash
# 导出容器
docker export 1e560fca3906 > ubuntu.tar
# 可以使用 docker import 从容器快照文件中再导入为镜像
cat docker/ubuntu.tar | docker import - test/ubuntu:v1
# 导入容器快照，可以通过指定 URL 或者某个目录来导入
docker import http://example.com/exampleimage.tgz example/imagerepo
# 删除容器
docker rm -f 1e560fca3906
# 下面的命令可以清理掉所有处于终止状态的容器。
$ docker container prune
```

### 持久化数据

#### volume和bind操作

```bash
# 创建volume
docker volume create $name
docker volume create --name **

# 使用命名为todo-db的volume，映射到容器的路径为/etc/todos
docker run -dp 3000:3000 -v todo-db:/etc/todos getting-started
# 使用bind，将本地目录/path/to/data映射到容器的路径为/usr/local/data
docker run -dp 3000:3000 -v /path/to/data:/usr/local/data getting-started

# 查看当前volume列表
docker volume ls

# 可以获取这个volume在docker 虚拟机中的位置
docker volume inspect $volumeName

# 通过inspect可以看到volume和Bind的信息
 docker inspect $containerName | grep Binds -A 2
 docker inspect $containerName | grep -C 3 volume
```

#### 参考

[docker volume](https://www.jianshu.com/p/8c22cdfc0ffd)

### 镜像管理

```bash
# 列出镜像列表, 
# REPOSITORY：表示镜像的仓库源
# TAG：镜像的标签
# IMAGE ID：镜像ID
# CREATED：镜像创建时间
# SIZE：镜像大小
docker image list
docker image ls
docker images

root@keepthinker# docker image list
REPOSITORY                    TAG              IMAGE ID       CREATED         SIZE
gcr.io/k8s-minikube/kicbase   v0.0.26          b0c9ec980b3d   12 months ago   1.08GB
ubuntu-nginx                  0.1              507ed75ca0a8   13 months ago   161MB
centos-nginx-01               latest           206860dce7f1   13 months ago   455MB
keepthinker/getting-started   latest           3792a77453fa   14 months ago   383MB
getting-started               latest           3792a77453fa   14 months ago   383MB
mysql                         5.7              09361feeb475   14 months ago   447MB
ubuntu                        latest           9873176a8ff5   14 months ago   72.7MB
docker/getting-started        latest           083d7564d904   14 months ago   28MB
node                          12-alpine        deeae3752431   16 months ago   88.9MB
grokzen/redis-cluster         5.0.12           004cbba7d676   17 months ago   549MB

# 登录和退出
docker login
docker logout
# 搜索镜像
docker search httpd
# 拉去镜像
docker pull ubuntu
# 删除镜像
docker rmi hello-world
docker image rm hello_world

# 更新镜像
docker run -t -i ubuntu:15.10 /bin/bash
# 在运行的容器内使用 apt-get update 命令进行更新。
# 接下来提交容器，语法：docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]
# -m: 提交的描述信息
# -a: 指定镜像作者
# ubuntu-nginx-01: 指定要创建的目标镜像名
docker commit -m "ubuntu nginx" -a "keepthinker" c6425e8f0695 ubuntu-nginx-01

# 构建镜像
vim Dockerfile
# edit Dockerfile 

# -t ：指定要创建的目标镜像名
# . ：Dockerfile 文件所在目录，可以指定Dockerfile 的绝对路径
docker build -t keepthinker/centos:6.7 .

# 为镜像添加一个标签，":"后面就是标签，其中第一个英文时自己的用户名
docker tag 860c279d2fec runoob/centos:dev


# 用户登录后，可以通过 docker push 命令将自己的镜像推送到 Docker Hub。
# 以下命令中的 username 请替换为你的 Docker 账号用户名。
$ docker tag ubuntu:18.04 username/ubuntu:18.04
$ docker image ls

REPOSITORY      TAG        IMAGE ID            CREATED           ...  
ubuntu          18.04      275d79972a86        6 days ago        ...  
username/ubuntu 18.04      275d79972a86        6 days ago        ...  
$ docker push username/ubuntu:18.04
$ docker search username/ubuntu

# 查看当前登录用户名等信息
cat ~/.docker/config.json

# 查看docker用户密码
echo "XXXXXXXXXXXXXXX" | base64 -d -

# 导出镜像
docker save -o my_ubuntu_v3.tar runoob/ubuntu:v3
# 导入镜像
docker load -i my_ubuntu_v3.tar

# The docker system prune command is used to remove unused Docker objects.
docker system prune -af

```

#### 构建镜像

mkdir nginx-docker
vim Dockerfile

A Docker image consists of read-only layers each of which represents a Dockerfile instruction. The layers are stacked and each one is a delta of the changes from the previous layer. Consider this `Dockerfile`:

##### Dockerfile示例内容如下

一定注明syntax让Dockfile使用最新语法。

```dockerfile
# syntax=docker/dockerfile:1
FROM ubuntu:18.04
COPY . /app
RUN make /app
CMD python /app/app.py
```

Each instruction creates one layer:

- `FROM` creates a layer from the `ubuntu:18.04` Docker image.
- `COPY` adds files from your Docker client’s current directory.
- `RUN` builds your application with `make`.
- `CMD` specifies what command to run within the container.

When you run an image and generate a container, you add a new *writable layer* (the “container layer”) on top of the underlying layers. All changes made to the running container, such as writing new files, modifying existing files, and deleting files, are written to this writable container layer.

##### FROM指令

定制的镜像都是基于 FROM 的镜像，这里的 nginx 就是定制需要的基础镜像。后续的操作都是基于 nginx。

##### RUN指令

在docker build时候运行。

RUN <命令行命令>

> <命令行命令> 等同于，在终端操作的 shell 命令。

RUN ["可执行文件", "参数1", "参数2"]

> 例如：RUN ["./test.php", "dev", "offline"] 等价于 RUN ./test.php dev offline

**注意**：Dockerfile 的指令每执行一次RUN都会在 docker 上新建一层。所以过多无意义的层，会造成镜像膨胀过大。以 && 符号连接命令，这样执行后，只会创建 1 层镜像。

例子：

```bash
# escape=\
FROM nginx
ENV WELCOME_MSG=hello\ world
EXPOSE 80/tcp
WORKDIR /app
ADD . /app
RUN echo "${WELCOME_MSG}, this is a nginx image made locally with some tools installed" \
  && apt-get update \
  && apt-get install -y procps \
  && apt-get install -y iproute2
```

##### 构建镜像

```bash
docker build -t nginx:v3 .
```

###### 上下文路径

最后的 . 代表本次执行的上下文路径，上下文路径，是指 docker 在构建镜像，有时候想要使用到本机的文件（比如复制），docker build 命令得知这个路径后，会将路径下的所有内容打包。

##### COPY指令

复制指令，从上下文目录中复制文件或者目录到容器里指定路径。

```dockerfile
COPY [--chown=<user>:<group>] <源路径1>...  <目标路径>
COPY [--chown=<user>:<group>] ["<源路径1>",...  "<目标路径>"]
```

**[--chown=<user>:<group>]**：可选参数，用户改变复制到容器内文件的拥有者和属组。

**<源路径>**：源文件或者源目录，这里可以是通配符表达式，其通配符规则要满足 Go 的 filepath.Match 规则。例如：

```dockerfile
COPY hom* /mydir/

COPY hom?.txt /mydir/
```

**<目标路径>**：容器内的指定路径，该路径不用事先建好，路径不存在的话，会自动创建。

##### ADD指令

ADD 指令和 COPY 的使用格类似（同样需求下，官方推荐使用 COPY）。功能也类似，不同之处如下：

- ADD 的优点：在执行 <源文件> 为 tar 压缩文件的话，压缩格式为 gzip, bzip2 以及 xz 的情况下，会自动复制并解压到 <目标路径>。
- ADD 的缺点：在不解压的前提下，无法复制 tar 压缩文件。会令镜像构建缓存失效，从而可能会令镜像构建变得比较缓慢。具体是否使用，可以根据是否需要自动解压来决定。

##### CMD指令

类似于 RUN 指令，用于运行程序，但二者运行的时间点不同:

- CMD 在docker run 时运行。
- RUN 是在 docker build。

**作用**：为启动的容器指定默认要运行的程序，程序运行结束，容器也就结束。CMD 指令指定的程序可被 docker run 命令行参数中指定要运行的程序所覆盖。

**注意**：如果 Dockerfile 中如果存在多个 CMD 指令，仅最后一个生效。

格式：

```dockerfile
CMD <shell 命令> 
CMD ["<可执行文件或命令>","<param1>","<param2>",...] 
CMD ["<param1>","<param2>",...] # 该写法是为 ENTRYPOINT 指令指定的程序提供默认参数
```

推荐使用第二种格式，执行过程比较明确。第一种格式实际上在运行的过程中也会自动转换成第二种格式运行，并且默认可执行文件是 sh。

##### ENTRYPOINT指令

类似于 CMD 指令，但其不会被 docker run 的命令行参数指定的指令所覆盖，而且这些命令行参数会被当作参数送给 ENTRYPOINT 指令指定的程序。  
但是, 如果运行 docker run 时使用了 --entrypoint 选项，将覆盖 ENTRYPOINT 指令指定的程序。  
在执行 docker run 的时候可以指定 ENTRYPOINT 运行所需的参数。  

```bash
ENTRYPOINT ["<executeable>","<param1>","<param2>",...]
```

可以搭配 CMD 命令使用：一般是变参才会使用 CMD ，这里的 CMD 等于是在给 ENTRYPOINT 传参，以下示例会提到。

示例：

假设已通过 Dockerfile 构建了 nginx:test 镜像：

```bash
FROM nginx

ENTRYPOINT ["nginx", "-c"] # 定参
CMD ["/etc/nginx/nginx.conf"] # 变参 
```

1、不传参运行

$ docker run  nginx:test  
容器内会默认运行以下命令，启动主进程。  

nginx -c /etc/nginx/nginx.conf  

2、传参运行  

$ docker run  nginx:test -c /etc/nginx/new.conf  
容器内会默认运行以下命令，启动主进程(/etc/nginx/new.conf:假设容器内已有此文件)  

nginx -c /etc/nginx/new.conf

##### ENV指令

设置环境变量，定义了环境变量，那么在后续的指令中，就可以使用这个环境变量。

格式：

```bash
ENV <key> <value>
ENV <key1>=<value1> <key2>=<value2>...
# 以下示例设置 NODE_VERSION = 7.2.0 ， 在后续的指令中可以通过 $NODE_VERSION 引用：

ENV NODE_VERSION 7.2.0

RUN curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION-linux-x64.tar.xz" \
  && curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/SHASUMS256.txt.asc"
```

##### ARG指令

构建参数，与 ENV 作用一致。不过作用域不一样。ARG 设置的环境变量仅对 Dockerfile 内有效，也就是说只有 docker build 的过程中有效，构建好的镜像内不存在此环境变量。

构建命令 docker build 中可以用 --build-arg <参数名>=<值> 来覆盖。

格式：

```bash
ARG <参数名>[=<默认值>]
```

##### VOLUME指令

定义匿名数据卷。在启动容器时忘记挂载数据卷，会自动挂载到匿名卷。

作用：

- 避免重要的数据，因容器重启而丢失，这是非常致命的。
- 避免容器不断变大。

格式：

```bash
VOLUME ["<路径1>", "<路径2>"...] VOLUME <路径>
```

在启动容器 docker run 的时候，我们可以通过 -v 参数修改挂载点。

##### EXPOSE

仅仅只是声明端口。

作用：

- 帮助镜像使用者理解这个镜像服务的守护端口，以方便配置映射。
- 在运行时使用随机端口映射时，也就是 docker run -P 时，会自动随机映射 EXPOSE 的端口。

格式：

```bash
EXPOSE <端口1> [<端口2>...]
```

##### WORKDIR

指定工作目录。用 WORKDIR 指定的工作目录，会在构建镜像的每一层中都存在。（WORKDIR 指定的工作目录，必须是提前创建好的）。

docker build 构建镜像过程中的，每一个 RUN 命令都是新建的一层。只有通过 WORKDIR 创建的目录才会一直存在。

格式：

```bash
WORKDIR <工作目录路径>
```

##### USER

用于指定执行后续命令的用户和用户组，这边只是切换后续命令执行的用户（用户和用户组必须提前已经存在）。

格式：

```bash
USER <用户名>[:<用户组>]
```

##### HEALTHCHECK

用于指定某个程序或者指令来监控 docker 容器服务的运行状态。

格式：

```bash
HEALTHCHECK [选项] CMD <命令>：设置检查容器健康状况的命令 
HEALTHCHECK NONE：如果基础镜像有健康检查指令，使用这行可以屏蔽掉其健康检查指令 
HEALTHCHECK [选项] CMD <命令> : 这边 CMD 后面跟随的命令使用，可以参考 CMD 的用法。
```

###### ONBUILD

用于延迟构建命令的执行。简单的说，就是 Dockerfile 里用 ONBUILD 指定的命令，在本次构建镜像的过程中不会执行（假设镜像为 test-build）。当有新的 Dockerfile 使用了之前构建的镜像 FROM test-build ，这时执行新镜像的 Dockerfile 构建时候，会执行 test-build 的 Dockerfile 里的 ONBUILD 指定的命令。

格式：

```bash
ONBUILD <其它指令>
```

##### LABEL

LABEL 指令用来给镜像添加一些元数据（metadata），以键值对的形式，语法格式如下：

```bash
LABEL <key>=<value> <key>=<value> <key>=<value> ...

比如我们可以添加镜像的作者：

LABEL org.opencontainers.image.authors="runoob"
```

#### 参考

[Docker Dockerfile | 菜鸟教程](https://www.runoob.com/docker/docker-dockerfile.html)

[Dockerfile reference | Docker Documentation](https://docs.docker.com/engine/reference/builder/)

### 网络管理

```bash
# 新建网络
# -d：参数指定 Docker 网络类型，有 bridge、overlay。
# 其中 overlay 网络类型用于 Swarm mode
> docker network create -d bridge test-net
> docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
a2617bd43c3e   bridge    bridge    local
b284a3bb9507   host      host      local
57f6d07cc2d2   none      null      local
e8d26617b103   test-net   bridge    local


> docker run -itd --name test1 --network test-net ubuntu /bin/bash
> docker run -itd --name test2 --network test-net ubuntu /bin/bash
# 上述两个容器可以互相ping通，可以用apt-get update && apt-get install iputils-ping安装ping命令

# --rm：容器退出时自动清理容器内部的文件系统。
# -h HOSTNAME 或者 --hostname=HOSTNAME： 设定容器的主机名，它会被写到容器内的 /etc/hostname 和 /etc/hosts。
# --dns=IP_ADDRESS： 添加 DNS 服务器到容器的 /etc/resolv.conf 中，让容器用这个服务器来解析所有不在 /etc/hosts 中的主机名。
# --dns-search=DOMAIN： 设定容器的搜索域，当设定搜索域为 .example.com 时，在搜索一个名为 host 的主机时，DNS 不仅搜索 host，还会搜索 host.example.com。
$ docker run -it --rm -h host_ubuntu  --dns=114.114.114.114 --dns-search=test.com ubuntu

# 查看网络信息
docker network inspect custom_network

# 将某个容器加入某网络
docker network connect custom_network my_container

# 断开网络
docker network disconnect custom_network my_container

```

### 参考
[Docker 网络模式详解及容器间网络通信](https://xie.infoq.cn/article/97355a6e7ac01bce8532d5ff5)

## Docker Compose

Compose is a tool for defining and running multi-container Docker applications. With Compose, you use a YAML file to configure your application’s services. Then, with a single command, you create and start all the services from your configuration.

docker compose示例文件

```bash
version: "3.9"  # optional since v1.27.0
services:
  web:
    build: .
    ports:
      - "8000:5000"
    volumes:
      - .:/code
      - logvolume01:/var/log
    depends_on:
      - redis
    environment:
      FLASK_DEBUG: True
    env_file:
      - ./Docker/api/api.env
  redis:
    image: redis
volumes:
  logvolume01: {}
```

### 相关命令

```bash
# 在前台启动
docker compose up
# 在后台启动，(for “detached” mode) 
docker compose up -d
# stop your services once you’ve finished with them
docker compose stop
# start your services
docker compose start
# 查看进程
docker compose ps
# run one-off commands for your services.
docker compose run web env
docker compose run echo $PATH
# see other available commands
docker compose --help
# You can bring everything down, removing the containers entirely, with the down command. 
# Pass --volumes to also remove the data volume used by the Redis container:
docker compose down
# see docker compose version
docker compose version 
docker-compose version 
# Execute a command in a running container.
docker compose exec $containerId /bin/sh
```

### 添加环境变量

It’s possible to **use environment variables in your shell** to populate values inside a Compose file.

If you have multiple environment variables, you can substitute them by adding them to a default environment variable **file named `.env`** or by providing a path to your environment variables file using the **`--env-file` command line option**.

#### Environment variables precedence

The order of precedence is as follows:

1. Passed from the command line `docker compose run --env <KEY[=[VAL]]>`
2. Passed from/set in `compose.yaml` service’s configuration, from the [environment key](https://docs.docker.com/compose/compose-file/#environment).
3. Passed from/set in `compose.yaml` service’s configuration, from the [env_file key](https://docs.docker.com/compose/compose-file/#env_file).
4. Passed from/set in Container Image in the [ENV directive](https://docs.docker.com/engine/reference/builder/#env).

#### 参考

[Environment variables in Compose | Docker Documentation](https://docs.docker.com/compose/environment-variables/)

### 其他

```bash
docker stats 9f215ed0b0d3
## make a new image from a container's changes
docker commit 70e5a5f241b8 centos-nginx-01
## create a new docker container with port mapping from 20080(host's port) -> 80(container's port) in backgroud
docker run -d -p 20080:80 centos-nginx-01  
docker ps
docker ps -a
docker run -p 8080:80 --name hello -d hello-world
docker run -it ubuntu bash
docker images
docker build -t getting-started .
docker login -u
docker login -u keepthinker
docker image ls
docker exec 6a9f550510c0 cat /data.txt
docker stop 6a9f550510c0
docker rm 6a9f550510c0
docker rm 466f31895334 83629defd935  96f729a8d7be e8a3800a262b e01445bfd6e1
docker start 82e2e71d7d79
docker attach 82e2e71d7d79
docker rename bb1a263ad46b ubuntu-bash
docker exec -i -t 0ce1dc54c68c sh
docker logs 2b1b7a428627
```

## 为Docker daemon设置代理

```bash
# Create a systemd drop-in directory for the docker service:
mkdir -p /etc/systemd/system/docker.service.d

# Create a file named /etc/systemd/system/docker.service.d/http-proxy.conf that adds the HTTP_PROXY environment variable:
# Multiple environment variables can be set; to set both a non-HTTPS and a HTTPs proxy; 
# 如果代理有异常，可以去除http://和https://再试一下，
[Service]
Environment="HTTP_PROXY=http://proxy.example.com:80"
Environment="HTTPS_PROXY=https://proxy.example.com:443"
Environment="NO_PROXY=localhost,127.0.0.1,docker-registry.example.com,.corp"
例子：
[Service]
Environment="HTTP_PROXY=127.0.0.1:7890" "HTTPS_PROXY=127.0.0.1:7890" "NO_PROXY=localhost,127.0.0.1,registry-1.docker.io"

# Flush changes and restart Docker
systemctl daemon-reload  
systemctl restart docker
```

[Docker网络代理设置_styshoo的博客-CSDN博客_docker代理设置](https://blog.csdn.net/styshoo/article/details/55657714)
[docker-http-proxy](https://docs.docker.com/engine/admin/systemd/#http-proxy)

## 设置镜像源

### 创建daemon.json文件

创建或修改 /etc/docker/daemon.json 文件，修改为如下形式

```json
{
    "registry-mirrors": [
        "http://hub-mirror.c.163.com",
        "https://docker.mirrors.ustc.edu.cn",
        "https://mirror.ccs.tencentyun.com"
    ]
}
```

### 加载重启docker

service docker restart

### 查看是否成功

```bash
# 查看docker一些信息
docker info
```

### 参考

[docker 设置国内镜像源](https://blog.csdn.net/whatday/article/details/86770609)

## Minikube

docker pull anjone/kicbase 

minikube start --vm-driver=docker --base-image="anjone/kicbase" --image-mirror-country='cn' --image-repository='registry.cn-hangzhou.aliyuncs.com/google_containers'

docker   load  -i kicbase.tar 

[minikube start启动集群失败Unable to find image gcr.io/k8s-minikube/kicbase:v0.0.10](https://blog.csdn.net/kelsel/article/details/107728562)

## **对比虚拟机与Docker**

**Docker守护进程**可以直接与**主操作系统**进行通信，为各个**Docker容器**分配资源；它还可以将容器与**主操作系统**隔离，并将各个容器互相隔离。**虚拟机**启动需要数分钟，而**Docker容器**可以在数毫秒内启动。由于没有臃肿的**从操作系统**，Docker可以节省大量的磁盘空间以及其他系统资源。

说了这么多Docker的优势，大家也没有必要完全否定**虚拟机**技术，因为两者有不同的使用场景。**虚拟机**更擅长于彻底隔离整个运行环境。例如，云服务提供商通常采用虚拟机技术隔离不同的用户。而**Docker**通常用于隔离不同的应用，例如**前端**，**后端**以及**数据库**。

**服务器虚拟化** **vs Docker**

**服务器**好比运输码头：拥有场地和各种设备（服务器硬件资源）

**服务器虚拟化**好比作码头上的仓库：拥有独立的空间堆放各种货物或集装箱

(仓库之间完全独立，独立的应用系统和操作系统）

**Docker**比作集装箱：各种货物的打包

(将各种应用程序和他们所依赖的运行环境打包成标准的容器,容器之间隔离)

Docker有着小巧、迁移部署快速、运行高效等特点，但隔离性比服务器虚拟化差：不同的集装箱属于不同的运单（Docker上运行不同的应用实例），相互独立（隔离）。但由同一个库管人员管理（主机操作系统内核），因此通过库管人员可以看到所有集装箱的相关信息（因为共享操作系统内核，因此相关信息会共享）。

服务器虚拟化就好比在码头上（物理主机及虚拟化层），建立了多个独立的“小码头”—仓库（虚拟机）。其拥有完全独立（隔离）的空间，属于不同的客户（虚拟机所有者）。每个仓库有各自的库管人员（当前虚拟机的操作系统内核），无法管理其它仓库。不存在信息共享的情况

因此，我们需要根据不同的应用场景和需求采用不同的方式使用Docker技术或使用服务器虚拟化技术。例如一个典型的Docker应用场景是当主机上的Docker实例属于单一用户的情况下，在保证安全的同时可以充分发挥Docker的技术优势。对于隔离要求较高的环境如混合用户环境，就可以使用服务器虚拟化技术。正则科技提供了丰富的Docker应用实例，满足您的各种应用需求，并且支持在已经安装了自在（Isvara）服务器虚拟化软件的主机上同时使用服务器虚拟化技术和Docker技术提供不同技术场景。

然后Docker并没有和虚拟机一样利用一个独立的Guest OS执行环境的隔离，它利用的是目前当前Linux内核本身支持的容器方式，实现了资源和环境的隔离，简单来说，Docker就是利用Namespace 实现了系统环境的隔离，利用了cgroup实现了资源的限制，利用镜像实例实现跟环境的隔离。

#### **Docker vs. VM – where is the difference?**

Docker is container based technology and containers are just **user space of the operating system**. At the low level, a container is just a set of processes that are isolated from the rest of the system, running from a distinct image that provides all files necessary to support the processes. It is built for running applications. In Docker, **the containers running share the host OS kernel.**

A Virtual Machine, on the other hand, is not based on container technology. They are made up of **user space plus kernel space of an operating system**. Under VMs, server hardware is virtualized. **Each VM has Operating system (OS) & apps. It shares hardware resource from the host.**

VMs & Docker – each comes with benefits and demerits. Under a VM environment, each workload needs a complete OS. But with a container environment, multiple workloads can run with 1 OS. The bigger the OS footprint, the more environment benefits from containers. With this, it brings further benefits like Reduced IT management resources, reduced size of snapshots, quicker spinning up apps, reduced & simplified security updates, less code to transfer, migrate and upload workloads.

#### 参考

[docker容器与虚拟机有什么区别？](https://www.zhihu.com/question/48174633)

[DOCKER VS. VIRTUAL MACHINE: WHERE ARE THE DIFFERENCES?](https://devopscon.io/blog/docker/docker-vs-virtual-machine-where-are-the-differences/[Docker vs. Virtual Machine: Where are the differences? - DevOps Conference](https://devopscon.io/blog/docker/docker-vs-virtual-machine-where-are-the-differences/))

## 关于docker容器启动后修改或添加端口

### 方法一：删除原有容器，重新建新容器

docker rm -f 1e560fca3906

docker run -p 8080:80 --name hello -d hello-world

### 方法二：利用docker commit新构镜像

docker commit -m="has update" -a="runoob" e218edb10161 runoob/ubuntu:v2

docker run -p 8080:80 --name hello -d ubuntu:v2

### 方法三：修改文件端口，重启docker服务

1. 首先停止docker服务，比如用systemctl stop docker。

2. 然后修改位于/var/lib/docker/containers/{containerId}/hostconfig.json文件中的PortBindings，例子："PortBindings:":{"80/tcp":[{"HostIp":"0.0.0.0", "HostPort": "50080"}]}

3. 然后再修改/var/lib/docker/containers/{containerId}/config.v2.json的ExposedPorts。例子："Ports":{"80/tcp":[{"HostIp":"0.0.0.0","HostPort":"50080"}]}

4. 重新启动docker服务。systemctl restart docker。

5. 最后启动容器docker start {containerId}

参考：[关于docker容器启动后修改或添加端口 - 腾讯云开发者社区-腾讯云](https://cloud.tencent.com/developer/article/1833131)

## Windows

```powershell
# 查看docker相关信息的路径
cd \\wsl$\Ubuntu\mnt\wsl\docker-desktop-data\data\docker
```

## docker swarm
Docker Engine 1.12 引入了 Swarm 模式，一个 Swarm 由多个 Docker 主机组成，它们以 Swarm 集群模式运行。Swarm 集群由 Manager 节点（管理者角色，管理成员和委托任务）和 Worker 节点（工作者角色，运行 Swarm 服务）组成。这些 Docker 主机有些是 Manager 节点，有些是 Worker 节点，或者同时扮演这两种角色。

Swarm 创建服务时，需要指定要使用的镜像、在运行的容器中执行的命令、定义其副本的数量、可用的网络和数据卷、将服务公开给外部的端口等等。与独立容器相比，群集服务的主要优势之一是，你可以修改服务的配置，包括它所连接的网络和数据卷等，而不需要手动重启服务。还有就是，如果一个 Worker Node 不可用了，Docker 会调度不可用 Node 的 Task 任务到其他 Nodes 上。

### 参考
[Docker Swarm 集群管理利器核心概念扫盲
](https://www.cnblogs.com/mrhelloworld/p/docker15.html)





