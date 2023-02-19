[TOC]

# DOCKER BASE INFO

## DOCKER

![img](https://img-blog.csdnimg.cn/img_convert/e5eb7c3fbf535e2beaf9b99f98524cd0.png)

### Docker产生背景

> Docker 是云时代的产物，它的诞生是一种必然。
> 对于云计算\云服务的相关概念，本文不会去阐述。不过如果想了解 Docker， 那么必须对云服务的一些运营模式有所了解。

### 云服务的运营模式：

> IaaS（基础设施即服务）：经营的是基础设施，比如阿里云服务器（只安装操作系统）
>
> PaaS（平台即服务）：经营的是平台，比如 MySQL 开发平台（安装在 linux里面现成的平台）、redis 开发平台、nginx。
>
> SaaS（软件即服务）：经营的是软件，比如公司的 OA 系统（部署到远程服务器中的 OA 软件)
>
> Docker 就是伴随着 PaaS 产生的

### docker是什么

> Docker 就是一种虚拟化容器技术。
>
> 通过 Docker 这种虚拟化容器技术，我们可以对物理机的资源进行更加合理有效的利用，可以将一台物理机器虚拟化出很多个拥有完整操作系统，并且相互独立的“虚拟计算机”。
>
> 不过，对于虚拟化技术的理解，我们要更加深入一些！
>
> 那么，什么是虚拟化呢？
>
> 在计算机中，虚拟化（英语：Virtualization）是一种资源管理技术，是将计算机的各种实体资源，如服务器、网络、内存及存储等，予以抽象、转换后呈现出来， 打破实体结构间的不可切割的障碍，使用户可以比原本的组态更好的方式来应 用这些资源。这些资源的新虚拟部份是不受现有资源的架设方式，地域或物理组态所限制。一般所指的虚拟化资源包括计算能力和资料存储。
>
> 在实际的生产环境中，虚拟化技术主要用来解决高性能的物理硬件产能过剩和老的旧的硬件产能过低的重组重用，透明化底层物理硬件，从而最大化的利用物理硬件。

![img](https://img-blog.csdnimg.cn/img_convert/b18e26061b9672f64ae52902347e4876.png)

> 虚拟化技术种类很多，例如：软件虚拟化、硬件虚拟化、内存虚拟化、网络虚拟化、桌面虚拟化、服务虚拟化、虚拟机等等。

**最常用的虚拟化技术有：全虚拟化和操作系统（**OS**）虚拟化。*

> *比如，**VMware workstation* *就是全虚拟化的实现。*

![image-20220718223749332](https://img-blog.csdnimg.cn/img_convert/df72b8f3beba114ef52760c4404fd411.png)

> 基于操作系统创建出一些相互独立的、功能虚拟化技术有多种实现方式，有基于硬件进行虚拟化的技术，而 Docker 只是针对操作系统进行虚拟化，对于硬件资源的使用率更低。

**相对于VMware这种虚拟化技术，Docker拥有着显著的优势：**

![img](https://img-blog.csdnimg.cn/img_convert/5464355a0141a9f8d919da4ec47b94ef.png)

> 启动速度快，Docker 容器启动操作系统在秒级就可以完成，而 VMware 却是达到分钟级。
>
> 系统资源消耗低，一台 Linux 服务器可以运行成千上百个 Docker 容器，而VMware 大概只能同时运行 10 个左右。
>
> 更轻松的迁移和扩展，由于 Docker 容器比 VMware 占用更少的硬盘空间， 在需要搭建几套软件环境的情况下，对安装好的 Docker 容器进行迁移会更快捷，更方便。而且 Docker 容器几乎可以在任意的平台上运行，包括虚拟机、物理机、公有云，私有云，个人电脑等，这种兼容性，可以让用户将一个应用程序从一个平台直接迁移到另一个平台。

### Docker核心概念

> docker 包含四个基本概念：
>
> 镜像（Image）
>
> 容器（Container）
>
> 仓库注册中心（Registry）
>
> 仓库（Repository）
>
> 理解了这四个概念，就理解了 docker 的整个生命周期了！

![在这里插入图片描述](https://img-blog.csdnimg.cn/19c37c15f45f49ddb96e2c262fa1e8c6.png#pic_center)

#### 镜像

> Docker 镜像（Image）就是一个只读的模板。
>
> Docker 镜像可以用来创建 Docker 容器。
>
> Docker 镜像和 Docker 容器的关系，类似于 java 中 class 类与对象之间的关系。
>
> Docker 提供了一个很简单的机制来创建镜像或者更新已有的镜像，用户甚至可以直接从其他人那里下载一个已经做好的镜像来直接使用。

#### 容器
> Docker 利用容器（Container）来运行应用。
>
> 容器是镜像创建的运行实例。它可以被启动、开始、停止、删除。每个容器都是相互隔离的、保证安全的平台。
>
> 可以把容器看成是一个简易版的 Linux 环境（包括 ROOT 用户权限、进程空间、用户空间、网络等）和运行在其中的应用程序。

#### Registry

> Registry 是集中存放镜像文件的场所。
> Repository 是对于其中的镜像进行分类管理。
> 一个 Registry 中会有多个 Repository。
> Registry 分为公有（public）和私有（private）两种形式。
> 最大的公有 Registry 是Docker Hub，存放了数量庞大的镜像供用户下载使用。
> 国内的公开 Registry 包括 USTC、网易云、DaoCloud、AliCloud 等，可以供大陆用户更稳当快捷的访问。
> 用户可以在本地创建一个私有 Registry。
> 用户创建了自己的镜像之后就可以使用 push 命令将它上传的公有 Registry 或者私有 Registry 中，这样下次在另一台机器上使用这个镜像的时候，只需要从Registry 上pull 下来运行就可以了。

#### Repository
> Repository 仓库可看成一个代码控制中心，用来保存镜像。
> 一个 Docker Registry 中可以包含多个仓库（Repository）；每个仓库可以包含多个标签（Tag）；每个标签对应一个镜像。
> 一个 Repository *中会有多个不同tag的Image。
> 比如名称为 centos 的 Repository 仓库下，有 tag 为 6 或者 7 的 Image 镜像。
> 通常，一个仓库会包含同一个软件不同版本的镜像，而标签就常用于对应该软件的各个版本。我们可以通过 <仓库名>:<标签> 的格式来指定具体是这个软件哪个版本的镜像。如果不给出标签，将以 latest 作为默认标签。

## Docker安装

> 官方默认的操作系统是安装到 **Ubuntu**。
>
> 官方网站上有各种环境下的安装指南，比如：CentOS、Ubuntu 和 Debian 系列的安装。
>
> 而我们现在主要讲解的是基于 CentOS 7.x 上面的安装。
>
> 一切完毕之后，只需要执行以下命令即可完成 Docker 的安装：

```shell
删除： yum -y remove docker
安装： yum install -y docker
启动： systemctl start docker
```

## Docker镜像

### 列出镜像

```shell
docker images
```

![img](https://img-blog.csdnimg.cn/img_convert/8a594ccc509c94d03e94bf18abdc527d.jpeg)

> **Repository**：镜像所在仓库名称**Tag**：镜像版本*
>
> **Image ID**：镜像* *ID*
>
> **Created**：镜像创建时间*
>
> **Size**：镜像大小*

### 搜索镜像

```shell
docker search $imagename
```

![img](https://img-blog.csdnimg.cn/img_convert/377b6a86a43d6eaf478b317e20bda168.jpeg)

> **NAME**：仓库名称
>
> **DESCRIPTION**：镜像描述
>
> **STARS**：用户评价，反应一个镜像的受欢迎程度
>
> **OFFICIA**L：是否官方
>
> **AUTOMATED**：自动构建，表示该镜像由 Docker Hub 自动构建流程创建的

### **拉取镜像**

> 我们可以使用命令从一些公用仓库中拉取镜像到本地，下面就列举一些常用的公用仓库拉取镜像的方式！

#### 从docker hub拉取

> Docker Hub 的网址：https://hub.docker.com/

**需求：从\*Docker Hub仓库下载一个** **CentOS 7** **版本的操作系统镜像。**

```shell
docker pull centos:7
```

Docker Hub 是 docker 默认的公用 Registry，不过缺点是国内下载会比较慢。

![img](https://img-blog.csdnimg.cn/img_convert/20a28493d1179d360f59d108b5f92abd.png)

#### 从ustc 拉取（建议使用）

在宿主机器编辑文件（centos7 不支持 vim 命令，但是支持 vi 命令）：

```shell
vim /etc/docker/deamon.json
```

请在该配置文件中加入（没有该文件的话，请先建一个）：

![img](https://img-blog.csdnimg.cn/img_convert/c6deae2a159c53318736fcd9b53f1866.png)

最后，需要重启 docker 服务

```shell
systemctl restart docker
```

之后再使用 pull 命令拉取镜像，这时候是从 ustc 获取镜像，而且速度杠杠的。

```shell
docker pull centos:7
```

![img](https://img-blog.csdnimg.cn/img_convert/05dcd512c623ac4df6eaa8116328e1ce.jpeg)

### 删除镜像

删除指定镜像

```shell
docker rmi repository:tag docker rmi imageId
```

删除所有镜像

```shell
docker rmi ${docker images -q}
```

注意

> ***删除时，如果镜像的image id一致，则需要按照一定顺序进行删除，因为镜像之间有关联

###  导入导出镜像(镜像迁移)

导出镜像：

```shell
docker save repository:tag/imageId > /root/xx.tar.gz
docker save -o mynginx.tar myningx
```

导入镜像：

```shell
docker load < /root/xx.tar.gz
docker load -i mynginx.tar
```

## docker容器

### 创建并运行容器

> 创建容器命令：docker run
>
> 创建容器常用的参数说明：
>
> -i：表示运行容器
>
> -t：表示容器启动后会进入其命令行。加入这两个参数后，容器创建就能登录进去。即分配一个伪终端。
>
> –name :为创建的容器命名。
>
> -v：表示目录映射关系（前者是宿主机目录，后者是映射到宿主机上的目录），可以使用多个－v 做多个目录或文件映射。注意：最好做目录映射，在宿主机上做修改，然后共享到容器上。
>
> -d：在 run 后面加上-d 参数,则会创建一个守护式容器在后台运行（这样创建容器后不会自动登录容器，如果只加-i -t 两个参数，创建后就会自动进去容器）。
>
> -p：表示端口映射，前者是宿主机端口，后者是容器内的映射端口。可以使用多个－p做多个端口映射

以交互方式运行容器：

```shell
docker run -it --name containername imageID /bin/bash
```

以守护进程方式运行容器：

```shell
docker run -di containername imageId
```

*注意：通过* *run* *创建并进入容器之后，如果使用* *exit* *命令退出容器，则容器停止。再次进入该容器，先使用* *start* *启动容器，再使用* *exec/attach* *命令进入容器。*

### 启动容器

```shell
docker start containername id
```

### 进入容器

进入正在运行的容器的命令如下：

```shell
docker exec it containerName/Id /bin/bash
docker attach containerName/Id
```

*两者之间的区别：*

==attach* *进入容器之后，如果使用* *exit* *退出容器，则容器停止。*==

==*exec* *进入容器之后，使用* *exit* *退出容器，容器依然处于运行状态==

### **查看容器**

> *docker ps* ：查看正在运行的容器
>
> docker ps* *-**a*：查看历史运行过的容器
>
> docker ps -l*：查看最近运行过的容器

### **停止容器**

> *docker stop* *容器名称或者容器* *ID*

### **删除容器**

> 删除指定容器：
>
> *docker rm* *容器名称或者容器* *ID*
>
> 删除所有容器：
>
> *docker rm ‘docker ps -a -q’*

### **复制容器**

> ***docker cp\* \*源文件 目标文件\***

```shell
docker cp /root/boot.war	my-centos:/usr/local/
```

*说明：*

> */root/boot.war* *是宿主机器的路径*
>
> *my-centos* *是容器的名称*
>
> */usr/local/**是容器内的路径*
>
> 注意：源文件可以是宿主机器也可以是容器中的文件，同样，目标文件可以是容器也可以是宿主机器的文件。

## Docker应用

### Mysql部署

#### **拉取** **MySQL** 镜像

```shell
docker pull mysql:5.6
```

查看镜像

docker images

#### 创建MySQL容器

```shell
docker run -di --name ztq_mysql -*p* 3306:3306 -e MYSQL_ROOT_PASSWORD=root mysql:5.6
```

-p 代表端口映射，格式为 宿主机映射端口:容器运行端口

-e 代表添加环境变量 MYSQL_ROOT_PASSWORD 是 root 用户的登陆密码

#### 进入MySQL容器登录 MySQL

进入 mysql 容器

```shell
docker exec -it ztq_mysql /bin/bash
```

登录 mysql

```shell
mysql -u root -p
```

#### 远程登录MySQL

我们在我们本机的电脑上去连接虚拟机 Centos 中的 Docker 容器， 这里

![img](https://img-blog.csdnimg.cn/img_convert/f5acb9b757dd7d9cca8d9f66abff78b1.png)

#### 查看容器 IP地址

我们可以通过以下命令查看容器运行的各种数据

```shell
docker inspect mysql
```

![img](https://img-blog.csdnimg.cn/img_convert/787da8c50b0a39e2e1202973c8c56faa.jpeg)

## 其它

### 容器的历史和发展

> 2008年，容器的概念就已经基本定型了，并不是后面Docker造出来的。讲到容器，就不得不讲到它的前生LXC（Linux Container）。LXC是Linux内核提供的容器技术，能提供轻量级的虚拟化能力，能隔离进程和资源。
>
> 2009年，Cloud Foundry（业界第一个开源PaaS云平台）基于LXC（Linux Container）实现了对容器的操作，该项目取名为Warden。
>
> 2010年，dotCloud公司同样基于LXC技术，使用Go语言实现了一款容器引擎，也就是现在的Docker。那时，dotCloud公司还是个小公司，出生卑微的Docker没什么热度，活得相当艰难。
>
> 2013年，dotCloud公司决定将Docker开源。开源后，项目异常火爆，直接驱动dotCloud公司更名为Docker公司。Docker也快速成长，干掉了CoreOS公司的rkt容器和Google的lmctfy容器，直接变成了容器的事实标准。以致于后来人一提到容器就认为是Docker。

### 容器 VS 虚拟机

> 虚拟机：虚拟出一套硬件后，在其上运行一个完整操作系统，在该系统上再运行所需应用进程

![虚拟机底层](https://img-blog.csdnimg.cn/25b3a7a325bd41df81177e61dc4c780f.png?x-oss-process=type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RoaXJ0ZWVuMTIx,size_16,color_FFFFFF,t_70)

> 容器：容器的应用进程则直接运行于宿主机的内核，容器内没有自己的内核，也没有进行硬件虚拟。

![在这里插入图片描述](https://img-blog.csdnimg.cn/31cd2fc323f2415fb4395185bc4bc01a.png?x-oss-process=type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RoaXJ0ZWVuMTIx,size_16,color_FFFFFF,t_70)

容器与虚拟机对比:

| 特性       | 容器                     | 虚拟机                       |
| ---------- | ------------------------ | :--------------------------- |
| 启动速度   | 秒级                     | 分钟级                       |
| 体量       | 极小(MB)                 | 较大(GB)                     |
| 性能       | 近似物理机               | 性能损耗大                   |
| 系统支持量 | 单机支持上千个容器       | 一般支持几十个               |
| 交付/部署  | 开发、测试、生成环境一致 | 易受操作系统、环境变量等限制 |
| 迁移/扩展  | 易迁移、易扩展           | 易受操作系统、资源等限制     |

### 容器解决了什么

> 运行环境一致性：容器打包了程序必需的所有依赖以及操作系统，可创建与其他应用相隔离的环境，彻底解决了环境的一致性问题。
>
> 各环境灵活迁移：Docker镜像可在所有主流 Linux 发行版、Microsoft 平台灵活迁移，极大减轻了开发和部署工作量。
>
> 资源消耗和冲突：MB级别的空间占用、秒级的启动速度可大大减少资源的消耗；同时容器会将应用相互隔离，我们可以为每个应用设置明确的资源限制，不必担心依赖项冲突或资源争用。
>
> 提高生产力：容器不是直接在主机操作系统运行，可减少调试和诊断环境差异所需的时间，提高了开发生产效率。
>
> 版本控制：每个容器的镜像都有版本控制，可以对历史版本进行追踪和差异比较。
>
> 安全隔离：容器会在操作系统级别虚拟化 CPU、内存、存储和网络资源，为开发者提供在逻辑上与其他应用相隔离的沙盒化操作系统接口。

### 容器原理

> 容器技术两大知识点==Cgroups（Linux Control Group）和Linux Namespace==。掌握它们，容器技术就基本了解了。

#### Namespace

> - Namespace重点在“隔离”，是Linux内核用来隔离资源的方式。
> - 每个Namespace下的资源对于其他Namespace都是不透明，不可见的。
> - 简单来说，容器和容器之间不要相互影响，容器和宿主机之间不要相互影响。

![NameSpace](https://img-blog.csdnimg.cn/67c6e1909a8f40ff812586691d36caa0.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmzZG4ubmV0L1RoaXJ0ZWVuMTIx,size_16,color_FFFFFF,t_70)

Linux 的Namespace机制提供了七种不同的命名空间：

|namespace	|系统调用参数	|隔离内容|
| ---------- | ------------------------ | :--------------------------- |
|UTS|CLONE_NEWUTS|主机名和域名|
|IPC|	CLONE_NEWIPC|	信号量、消息队列和共享内存|
|PID|	CLONE_NEWPID|	进程编号|
|Network|	CLONE_NEWNET|	网络设备、网络栈、端口等|
|Mount|	CLONE_NEWNS|	挂载点(文件系统)|
|User|	CLONE_NEWUSER|	用户和用户组|
|Cgroup|	CLONE_NEWCGROUP|	控制进程对系统资源的支配调度|

#### Cgroups

CGroups(Control Groups) 重点在“限制”。限制资源的使用，包括CPU、内存、磁盘的使用，体现出对资源的管理能力。

> 控制组（CGroup） 一个 CGroup 包含一组进程，并可以在这个 CGroup 上增加 Linux Subsystem 的各种参数配置，将一组进程和一组 Subsystem 关联起来。
>
> Subsystem 子系统 是一组资源控制模块，比如 CPU 子系统可以控制 CPU 时间分配，内存子系统可以限制 CGroup 内存使用量。可以通过lssubsys -a命令查看当前内核支持哪些 Subsystem。
>
> Hierarchy 层级树 主要功能是把 CGroup 串成一个树型结构，使 CGruop 可以做到继承，每个 Hierarchy 通过绑定对应的 Subsystem 进行资源调度。
>
> Task 在 CGroups 中，task 就是系统的一个进程。一个任务可以加入某个 CGroup，也可以从某个 CGroup 迁移到另外一个 CGroup。

### Dockerfile、Docker镜像、Docker容器的关系

> - Dockerfile是一个用来构建镜像的文本文件，文本内容包含了一条条构建镜像所需的指令和说明。
> - Docker镜像是由一系列只读的层组成的，Dockerfile中每一个命令都会在已有的只读层上再创建一个新的只读层。
> - 当镜像被创建时会在镜像的最上层添加一层可读写层，也就是容器层，所有对于运行时容器的修改其实是对于镜像读写层的修改。因此，镜像和容器的区别就在于：所有的镜像都是只读的，而容器等于镜像加上一层读写层，也就是说同一个镜像可以对应多个容器。
> - 一句话概括三者关系：Docker镜像由Dockerfile构建，Docker容器是Docker镜像的运行时状态。

### Docker的进程隔离

> 当我们在宿主机运行 Docker，通过docker run或docker start创建新容器进程时，会传入 CLONE_NEWPID 实现进程上的隔离；接着在方法createSpec的setNamespaces中完成除进程命名空间之外与用户、网络、IPC(信号量、消息队列和共享内存) 以及 UTS(主机名和域名) 相关的命名空间的设置。

![Docker实现原理](https://img-blog.csdnimg.cn/e198faf643c7450bb7e0244c263cdf83.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RoaXJ0ZWVuMTIx,size_16,color_FFFFFF,t_70)

### Docker的网络通信

> 当 Docker 容器完成命名空间的设置，其网络也变成了独立的命名空间，与宿主机的网络通信便产生了限制，这就导致外部很难访问到容器内的应用程序服务。因此，Docker 提供了 4 种网络模式，通过–net参数指定。
> host， 与宿主机共用一个Network Namespace，容器不会虚拟出自己的网卡、配置自己的IP等，而是使用宿主机的IP和端口。
>
> container，指定新创建的容器和已经存在的一个容器共享一个Network Namespace，而不是和宿主机共享。
>
> none，容器拥有自己的Network Namespace，但是并不为Docker容器进行任何网络配置。即容器没有网卡、IP、路由等信息，需要手动添加配置。
>
> bridge，Docker默认的网络设置，此模式会为每一个容器分配Network Namespace、设置IP等，并将一个主机上的Docker容器连接到一个虚拟网桥上。

由于Docker常用bridge模式，一下着重介绍该模式：

> 虚拟网桥：当Docker Server启动时，会在主机上创建一个名为docker0的虚拟网桥，此主机上启动的Docker容器都会连接到这个虚拟网桥上。
>
> 分配容器IP：Docker会选择一个与宿主机不同的IP地址和子网分配给docker0，连接docker0的容器都会从该子网中选择一个未占用的IP使用。
>
> 容器间通信：连在同一网桥上的容器可以相互通信，也可以出于安全考虑禁止通信。
>
> 容器与外界通信：IP包首先从容器发往默认网关docker0（IP包到达docker0也就是到达了主机），然后查询主机路由表，发现包应该有eth0发往主机网关，接着IP包会转给eth0由eth0发出。对于外界看来，IP包是由主机eth0发出的，docker容器对外是不可见的。

![bridge模式](https://img-blog.csdnimg.cn/abecd565b3744bc2a09bde31a257e411.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RoaXJ0ZWVuMTIx,size_16,color_FFFFFF,t_70)

### Docker的文件系统隔离

`上述解决了进程和网络等隔离的问题，但是 Docker 容器中的进程仍然能够访问或者修改宿主机器上的其他目录，这是我们不希望看到的。`

> 在新的进程中创建隔离的挂载点命名空间需要在 clone 函数中传入 CLONE_NEWNS，这样子进程就能得到父进程挂载点的拷贝，如果不传入这个参数子进程对文件系统的读写都会同步回父进程以及整个主机的文件系统。
> 当一个容器需要启动时，它一定需要提供一个根文件系统（rootfs），容器需要使用这个文件系统来创建一个新的进程，所有二进制的执行都必须在这个根文件系统中，并建立一些符号链接来保证 IO 不会出现问题。
> 通过 Linux 的chroot命令能够改变当前的系统根目录结构，通过改变当前系统的根目录，我们能够限制用户的权利，在新的根目录下并不能够访问旧系统根目录的结构个文件，也就建立了一个与原系统完全隔离的目录结构。

![docker文件隔离](https://img-blog.csdnimg.cn/d2f6e9e7c5d9413d8df6a70dc6d45741.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RoaXJ0ZWVuMTIx,size_16,color_FFFFFF,t_70)

### Docker的Cgroup

> Docker 安装目录下有一个 docker 目录，当启动一个容器时，就会创建一个与容器标识符相同的 CGroup，大概的层级关系：
>
> 每一个 CGroup 下面都有一个 tasks 文件，其中存储着属于当前控制组的所有进程的 pid，作为负责 cpu 的子系统，cpu.cfs_quota_us 文件中的内容能够对 CPU 的使用作出限制。
>
> 当我们 Docker 关闭掉正在运行的容器时，Docker 的子控制组对应的文件夹也会被 Docker 进程移除。

![Docker Cgroup](https://img-blog.csdnimg.cn/c2f7dde25ae944c68c66468beb0d33fe.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RoaXJ0ZWVuMTIx,size_16,color_FFFFFF,t_70)

### Docker命令大全

- 操作命令实践相对简单，可查看[Docker命令|菜鸟教程](https://www.runoob.com/docker/docker-command-manual.html)

### K8s应运而生

#### K8s是什么

> kubernetes，简称K8s，是用8代替8个字符“ubernete”而成的缩写。
> Kubernetes是Google开源的一个容器编排引擎，它具有完备的集群管理能力，支持自动化部署、服务滚动升级和在线扩容、可扩展的资源自动调度机制、多实例负载均衡等。

#### 为什么使用K8s

> 虽然Docker已经为容器提供了开放标准，但随着容器不断增加也出现了一系列问题：
>
> - 单机支持容器到达上限，怎么办？
> - 分布式环境下容器之间如何通信？
> - 如何协调和管理数量庞大的容器？
> - 容器的运行状况如何监控？
> - 应用升级版本如何做到不中断？
> - 如何做到批量启动容器？

## LINK

https://blog.csdn.net/hanxiaoyong_/article/details/127990519
