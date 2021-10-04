## docker command
```shell
## to kill a docker container process
docker kill 9f215ed0b0d3
docker pause 9f215ed0b0d3
docker unpause 9f215ed0b0d3
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
```



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

#### 参考

[docker容器与虚拟机有什么区别？](https://www.zhihu.com/question/48174633)
