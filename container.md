title: 容器化技术演进-docker
speaker: 愚安
transition: zoomin
theme: colors

[slide]

# DOCKER
## 容器化技术演进
<img src="https://www.docker.com/sites/default/files/island_1.png" />

### [愚安](github.com/marshalYuan)@dianwoba.com

[slide]

#谁有mysql的安装包，Mac版的？

[subslide]

#写代码累了，玩会儿Hadoop？ {:&.moveIn}
[/subslide]

[note]
docker run -it —rm sequenceiq/hadoop-docker:2.7.0 /etc/bootstrap.sh -bash

cd $HADOOP_PREFIX

bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.0.jar grep input output 'dfs[a-z.]+'

bin/hdfs dfs -cat output/*

[/note]

[slide]

##Docker? 什么鬼...

![magic](http://media.giphy.com/media/ujUdrdpX7Ok5W/giphy.gif)


[slide]

![www.docker.io](https://www.docker.com/sites/all/themes/docker/assets/images/logo.png)

#Docker为何，焉有卵用？ {:&.flexbox.vleft}
##Build, Ship, Run

[subslide]

> An open platform for distributed applications for developers and sysadmins

======

* `Build` Docker allows you to compose your application from microservices, without worrying about inconsistencies between development and production environments, and without locking into any platform or language.  {:&.rollIn}
* `Ship` Docker lets you design the entire cycle of application development, testing and distribution, and manage it with a consistent user interface.
* `Run` Docker offers you the ability to deploy scalable services, securely and reliably, on a wide variety of platforms.

[/subslide]

[slide]

#Docker为何，焉有卵用？ {:&.flexbox.vleft}

> Docker 是 PaaS 提供商 dotCloud 开源的一个基于 LXC 的高级容器引擎，源代码托管在 Github 上, 基于go语言并遵从Apache2.0协议开源。 {:&.fadeIn}

[slide]
#What's LXC?

> LXC为Linux Container的简写。Linux Container容器是一种内核虚拟化技术，可以提供轻量级的虚拟化，以便隔离进程和资源，而且不需要提供指令解释机制以及全虚拟化的其他复杂性。相当于C++中的NameSpace。容器有效地将由单个操作系统管理的资源划分到孤立的组中，以更好地在孤立的组之间平衡有冲突的资源使用需求。

[slide]
#VMs VS Containers
<img src="http://pointful.github.io/docker-intro/docker-img/containers-vs-vms.png" style="background-color:#FFFFFF" />

[slide]

#隔离性: Linux Namespace(ns) {:&.flexbox.vleft}

>每个用户实例之间相互隔离, 互不影响。 一般的硬件虚拟化方法给出的方法是VM，而LXC给出的方法是container，更细一点讲就是kernel namespace。其中pid、net、ipc、mnt、uts、user等namespace将container的进程、网络、消息、文件系统、UTS("UNIX Time-sharing System")和用户空间隔离开。

* 1) pid namespace {:&.moveIn.flexbox.vleft}
* 2) net namespace
* 3) ipc namespace
* 4) mnt namespace
* 5) uts namespace
* 6) user namespace

[note]
1) pid namespace
不同用户的进程就是通过pid namespace隔离开的，且不同 namespace 中可以有相同pid。所有的LXC进程在docker中的父进程为docker进程，每个lxc进程具有不同的namespace。同时由于允许嵌套，因此可以很方便的实现 Docker in Docker。

2) net namespace
有了 pid namespace, 每个namespace中的pid能够相互隔离，但是网络端口还是共享host的端口。网络隔离是通过net namespace实现的， 每个net namespace有独立的 network devices, IP addresses, IP routing tables, /proc/net 目录。这样每个container的网络就能隔离开来。docker默认采用veth的方式将container中的虚拟网卡同host上的一个docker bridge: docker0连接在一起。

3) ipc namespace
container中进程交互还是采用linux常见的进程间交互方法(interprocess communication - IPC), 包括常见的信号量、消息队列和共享内存。然而同 VM 不同的是，container 的进程间交互实际上还是host上具有相同pid namespace中的进程间交互，因此需要在IPC资源申请时加入namespace信息 - 每个IPC资源有一个唯一的 32 位 ID。

4) mnt namespace
类似chroot，将一个进程放到一个特定的目录执行。mnt namespace允许不同namespace的进程看到的文件结构不同，这样每个 namespace 中的进程所看到的文件目录就被隔离开了。同chroot不同，每个namespace中的container在/proc/mounts的信息只包含所在namespace的mount point。

5) uts namespace
UTS("UNIX Time-sharing System") namespace允许每个container拥有独立的hostname和domain name, 使其在网络上可以被视作一个独立的节点而非Host上的一个进程。

6) user namespace
每个container可以有不同的 user 和 group id, 也就是说可以在container内部用container内部的用户执行程序而非Host上的用户。
[/note]


[slide]

#可配额/可度量 - Control Groups (cgroups) {:&.flexbox.vleft}

>cgroups 实现了对资源的配额和度量。 cgroups 的使用非常简单，提供类似文件的接口，在 /cgroup目录下新建一个文件夹即可新建一个group，在此文件夹中新建task文件，并将pid写入该文件，即可实现对该进程的资源控制。groups可以限制blkio、cpu、cpuacct、cpuset、devices、freezer、memory、net_cls、ns九大子系统的资源，以下是每个子系统的详细说明：

* blkio 这个子系统设置限制每个块设备的输入输出控制。例如:磁盘，光盘以及usb等等。 {:&.moveIn.flexbox.vleft} 
* cpu 这个子系统使用调度程序为cgroup任务提供cpu的访问。
* cpuacct 产生cgroup任务的cpu资源报告。
* cpuset 如果是多核心的cpu，这个子系统会为cgroup任务分配单独的cpu和内存。
* devices 允许或拒绝cgroup任务对设备的访问。
* freezer 暂停和恢复cgroup任务。
* memory 设置每个cgroup的内存限制以及产生内存资源报告。
* net_cls 标记每个网络包以供cgroup方便使用。
* ns 名称空间子系统

[slide]
<style>

</style>
<div>
<div style="width:50%" class="pull-left"><img src="https://docs.docker.com/images/docker-friends.png"   /></div>
<div style="width:50%" class="pull-left"><img src="https://upload.wikimedia.org/wikipedia/commons/thumb/0/09/Docker-linux-interfaces.svg/800px-Docker-linux-interfaces.svg.png"  /></div>
	

</div>

[slide]

#非linux用户

#boot2docker || kitematic {:&.rollIn}

### 老司机微软?

### 容器化技术：Hyper-V，SoftGrid

### Nono Server --> Windows Docker

[slide]

# Container 从何而来？

[slide]

#docker images:search,pull,push


<img src="http://pointful.github.io/docker-intro/docker-img/changes-and-updates.png" style="background-color:#FFFFFF" />

###Docker Hub

[slide]
#Docker 镜像

[subslide]
典型的启动Linux运行需要两个FS: bootfs + rootfs: {:&.flexbox.vleft}

![linux image](http://cdn3.infoqstatic.com/statics_s1_20150819-0313/resource/articles/analysis-of-docker-file-system-aufs-and-devicemapper/zh/resources/0328090.png)

=====

对于不同的linux发行版, bootfs基本是一致的, 但rootfs会有差别, 因此不同的发行版可以公用bootfs 如下图: {:&.flexbox.vleft}

![linux](http://cdn3.infoqstatic.com/statics_s1_20150819-0313/resource/articles/docker-core-technology-preview/zh/resources/0731015.png)

=====

典型的Linux在启动后，首先将 rootfs 设置为 readonly, 进行一系列检查, 然后将其切换为 "readwrite" 供用户使用。在Docker中，初始化时也是将 rootfs 以readonly方式加载并检查，然而接下来利用 union mount 的方式将一个 readwrite 文件系统挂载在 readonly 的rootfs之上，并且允许再次将下层的 FS(file system) 设定为readonly 并且向上叠加, 这样一组readonly和一个writeable的结构构成一个container的运行时态, 每一个FS被称作一个FS层。如下图: {:&.flexbox.vleft}


![docker container](http://cdn3.infoqstatic.com/statics_s1_20150819-0313/resource/articles/docker-core-technology-preview/zh/resources/0731016.png)

======

得益于AUFS的特性, 每一个对readonly层文件/目录的修改都只会存在于上层的writeable层中。这样由于不存在竞争, 多个container可以共享readonly的FS层。 所以Docker将readonly的FS层称作 "image" - 对于container而言整个rootfs都是read-write的，但事实上所有的修改都写入最上层的writeable层中, image不保存用户状态，只用于模板、新建和复制使用。 {:&.flexbox.vleft}

![docker image](http://cdn3.infoqstatic.com/statics_s1_20150819-0313/resource/articles/docker-core-technology-preview/zh/resources/0731017.png)

======

上层的image依赖下层的image，因此Docker中把下层的image称作父image，没有父image的image称作base image。因此想要从一个image启动一个container，Docker会先加载这个image和依赖的父images以及base image，用户的进程运行在writeable的layer中。所有parent image中的数据信息以及 ID、网络和lxc管理的资源限制等具体container的配置，构成一个Docker概念上的container。如下图: {:&.flexbox.vleft}

![container](http://cdn3.infoqstatic.com/statics_s1_20150819-0313/resource/articles/docker-core-technology-preview/zh/resources/0731018.png)

[/subslide]



[slide]
#docker 命令
* help
* insepect
* run

* net
* volume


[slide]
#build 与 Dockerfile

[dockerfile](http://github.com/tenstartups/openresty-docker)

[slide]

#Docker Toolbox
![toolbox](http://7qnbk5.com1.z0.glb.clouddn.com/img/docker%20toolbox.png)

[slide]
#docker-compose:让我们再懒一点

```yaml
wiki2:
    image: 'nickstenning/mediawiki'
    ports:
        - "8880:80"
    links:
        - db:database
    volumes:
        - /data/wiki2:/data

db:
    image: "mysql"
    expose:
        - "3306"
    environment:
        - MYSQL_ROOT_PASSWORD=defaultpass
```

[slide]

#Docker是唯一的么？

---

###Docker vs Rocket

[slide]
[subslide]

##当我们在谈论Docker时我们在谈论什么？

=======

![zhuangbi](http://img5.duitang.com/uploads/item/201407/24/20140724235249_uU2Gv.thumb.700_0.jpeg) 

=======

#那我得往高大上说

* 虚拟化 VS 容器化 {:&.bounceIn.vleft.flexbox} 
* PasS 
* 云计算

======

* 空间 {:&.bounceIn.vleft.flexbox} 
* 应用引擎（GAE, SAE, BAE ...）
* PaaS（Cloudfoundry, openShift ...）
* 虚拟主机（ESC，EC2，digital ocean ...）
* Docker



[/subslide]

[slide]

#Docker解决了什么问题？

[slide]

##遇到了什么问题？

---

<img src="http://pointful.github.io/docker-intro/docker-img/the-challenge.png" style="background-color:#FFFFFF"/>

[slide]
##魔鬼矩阵
---

<img src="http://pointful.github.io/docker-intro/docker-img/the-matrix-from-hell.png" style="background-color:#FFFFFF"/>

[slide]
##60's 运输的魔鬼矩阵
---

<img src="http://pointful.github.io/docker-intro/docker-img/also-a-matrix-from-hell.png" style="background-color:#FFFFFF"/>

[slide]
##解决方案：统一集装箱(container)
---

<img src="http://pointful.github.io/docker-intro/docker-img/intermodal-shipping-container.png" style="background-color:#FFFFFF"/>

[slide]
##Docker:代码集装箱
---

<img src="http://pointful.github.io/docker-intro/docker-img/shipping-container-for-code.png" style="background-color:#FFFFFF"/>

[slide]
##Containers With Dockers
---

<img src="http://pointful.github.io/docker-intro/docker-img/eliminates-matrix-from-hell.png" style="background-color:#FFFFFF"/>

[slide]

##踏上巨鲸
###我们的征途是星辰大海
![onepeace](http://img4.duitang.com/uploads/item/201406/20/20140620172457_iVWjs.jpeg)
<p>Run everywhere,Run anything</p>



[slide]
<h1>
<img src="http://www.infoq.com/resource/articles/what-is-coreos/zh/smallimage/coreos_meitu_1.jpg" class="pull-left" />
<p>CoreOS::为云计算诞生的Linux</p>
</h1>
![coreos](http://cdn2.infoqstatic.com/statics_s1_20150819-0313/resource/articles/what-is-coreos/zh/resources/0919000.png)

---

* CoreOS 没有提供包管理工具
* CoreOS 采用双系统分区 (dual root partition) 设计
* CoreOS 使用 Systemd 取代 SysV 作为系统和服务的管理工具

[slide]

#CoreOS们

![tinyOS](http://dockerone.com/uploads/article/20150527/ec46998a23d029ec8ec77127bd1d3dd6.jpg)

![tinyOS](http://dockerone.com/uploads/article/20150512/3bc44b4b9583893642bfd0dc957f03d4.png)



[slide]

# Kubernets

![kubernets](http://7qnbk5.com1.z0.glb.clouddn.com/img/kubernets.jpg)

[slide]

[![tectonic](https://coreos.com/assets/images/media/tectonic-launch.png)](https://tectonic.com/)

![tectonic](https://tectonic.com/assets/images/screenshots/replication-controller-list.png) {:&.moveIn}


[slide]
# 目前应用

[subslide]

* 微博平台Docker集群的规模情况： {:&.vleft.flexbox}

	* Docker集群规模达到1000+节点
	* QPS峰值达到800K/s
	* 4个9的服务SLA达到150ms
	* 共覆盖23个核心服务
	* 春晚共调度近300节点完成动态扩容

======

* 微博平台Docker集群的技术栈： {:&.vleft.flexbox}

	* 宿主机：CentOS 6.5
	* Docker：1.3.2
	* Registry：docker-registry 0.9.1版本
	* 组网：host模式
	* 监控：cAdvisor + Elasticsearch + Kibana + Graphite
	* 文件系统：devicemapper
	* 镜像发布：Jenkins Container
	* 容器：容器即服务，服务即容器
	* 日志：volume挂载
	* 生命周期管理：自研，类似Compose
	* 服务发现：自研，类似Kubernetes的Pods和Service

[/subslide]




[slide]

# Demo 时间 {:&.moveIn}

## 短链接服务的持续集成 

[slide]
![naruki](http://img1.gamersky.com/image2014/12/20141129hzy_3/gamersky_17small_34_2014122103017B.jpg)

# 口寄せの術--docker転生 {:&.vleft.flexbox}


* nginx(openresty) {:&.bounceIn.vleft.flexbox}
* redis
* 再来点什么呢？
* zookeeper
* kafka
* storm

[slide]

```
zookeeper:
  image: jplock/zookeeper:3.4.6
  ports: 
    - "2181:2181"
nimbus:
  image: wurstmeister/storm-nimbus:0.9.2
  ports:
    - "3773:3773"
    - "3772:3772"
    - "6627:6627"
  links: 
    - zookeeper:zk
supervisor:
  image: wurstmeister/storm-supervisor:0.9.2
  ports:
    - "8000:8000"
  links: 
    - nimbus:nimbus
    - zookeeper:zk
ui:
  image: wurstmeister/storm-ui:0.9.2
  ports:
    - "8080:8080"
  links: 
    - nimbus:nimbus
    - zookeeper:zk
kafka1:
  image: wurstmeister/kafka:0.8.1
  ports:
    - "9092:9092"
  links:
    - zookeeper:zk
  environment:
    BROKER_ID: 1
    HOST_IP: 192.168.99.102
    PORT: 9092
kafka2:
  image: wurstmeister/kafka:0.8.1
  ports:
    - "9093:9093"
  links:
    - zookeeper:zk
  environment:
    BROKER_ID: 2
    HOST_IP: 192.168.99.102
    PORT: 9093
openfire:
  image: mdouglas/openfire
  ports:
  - "5222:5222"
  - "5223:5223"
  - "9091:9091"
  - "9090:9090"
```


