[TOC]

# Kubernetes

## 简介

### Kubernetes 是什么

> Kubernetes 是一个全新的基于容器技术的分布式架构解决方案，是 Google 开源的一个==容器集群管理系统==，Kubernetes 简称 K8S。
>
> Kubernetes 是一个一站式的完备的分布式系统开发和支撑平台，更是一个开放平台，对现有的编程语言、编程框架、中间件没有任何侵入性。
>
> Kubernetes 提供了完善的管理工具，这些工具涵盖了开发、部署测试、运维监控在内的各个环节。
>
> Kubernetes 具有完备的集群管理能力，包括==多层次的安全防护和准入机制、多租户应用支撑能力、透明的服务注册和服务发现机制、内建智能负载均衡器、强大的故障发现和自我修复能力、服务滚动升级和在线扩容能力、可扩展的资源自动调度机制、多粒度的资源配额管理能力。==

Kubernetes 官方文档：https://kubernetes.io/zh/

### Kubernetes 特性

> ① 自我修复
>
> 在节点故障时，重新启动失败的容器，替换和重新部署，保证预期的副本数量；杀死健康检查失败的容器，并且在未准备好之前不会处理用户的请求，确保线上服务不中断。
>
> ② 弹性伸缩
>
> 使用命令、UI或者基于CPU使用情况自动快速扩容和缩容应用程序实例，保证应用业务高峰并发时的高可用性；业务低峰时回收资源，以最小成本运行服务。
>
> ③ 自动部署和回滚
>
> K8S采用滚动更新策略更新应用，一次更新一个Pod，而不是同时删除所有Pod，如果更新过程中出现问题，将回滚更改，确保升级不影响业务。
>
> ④ 服务发现和负载均衡
>
> K8S为多个容器提供一个统一访问入口（内部IP地址和一个DNS名称），并且负载均衡关联的所有容器，使得用户无需考虑容器IP问题。
>
> ⑤ 机密和配置管理
>
> 管理机密数据和应用程序配置，而不需要把敏感数据暴露在镜像里，提高敏感数据安全性。并可以将一些常用的配置存储在K8S中，方便应用程序使用。
>
> ⑥ 存储编排
>
> 挂载外部存储系统，无论是来自本地存储，公有云，还是网络存储，都作为集群资源的一部分使用，极大提高存储使用灵活性。
>
> ⑦ 批处理
>
> 提供一次性任务，定时任务；满足批量数据处理和分析的场景。

## 集群架构与组件

Kubernetes 集群架构以及相关的核心组件如下图所示：一个 Kubernetes 集群一般包含一个 Master 节点和多个 Node 节点，一个节点可以看成是一台物理机或虚拟机。

![img](https://img2018.cnblogs.com/blog/856154/201910/856154-20191023003358108-1816205812.png)

### Master

Master 是 K8S 的集群控制节点，每个 K8S 集群里需要有一个 Master 节点来负责整个集群的管理和控制，基本上 K8S 所有的控制命令都是发给它，它来负责具体的执行过程。Master 节点通常会占据一个独立的服务器，因为它太重要了，如果它不可用，那么所有的控制命令都将失效。

Master 节点上运行着以下关键组件：

> ① **kube-apiserver**
>
> 是集群的统一入口，各组件协调者，以 HTTP Rest 提供接口服务，所有对象资源的增、删、改、查和监听操作都交给 apiserver 处理后再提交给 Etcd 存储。
>
> ② **kube-controller-manager**
>
> 是 K8S 里所有资源对象的自动化控制中心，处理集群中常规后台任务，一个资源对应一个控制器，而 controller-manager 就是负责管理这些控制器的。
>
> ③ **kube-scheduler**
>
> 根据调度算法为新创建的 Pod 选择一个 Node 节点，可以任意部署，可以部署在同一个节点上，也可以部署在不同的节点上。
>
> ④ **etcd**
>
> 是一个分布式的，一致的 key-value 存储，主要用途是共享配置和服务发现，保存集群状态数据，比如 Pod、Service 等对象信息。

### Node

除了 Master，K8S 集群中的其它机器被称为 Node 节点，Node 节点是 K8S 集群中的工作负载节点，每个 Node 都会被 Master 分配一些工作负载，当某个 Node 宕机时，其上的工作负载会被 Master 自动转移到其它节点上去。

每个 Node 节点上都运行着以下关键组件：

> ① **kubelet**
>
> kubelet 是 Master 在 Node 节点上的 Agent(代理)，与 Master 密切协作，管理本机运行容器的生命周期，负责 Pod 对应的容器的创建、启停等任务，实现集群管理的基本功能。
>
> ② **kube-proxy**
>
> 在 Node 节点上实现 Pod 网络代理，实现 Kubernetes Service 的通信，维护网络规则和四层负载均衡工作。
>
> ③ **docker engine**
>
> Docker 引擎，负责本机的容器创建和管理工作。
>
> Node 节点可以在运行期间动态增加到 K8S 集群中，前提是这个节点上已经正确安装、配置和启动了上述关键组件。在默认情况下 kubelet 会向 Master 注册自己，一旦 Node 被纳入集群管理范围，kubelet 就会定时向 Master 节点汇报自身的情况，例如操作系统、Docker 版本、机器的 CPU 和内存情况，以及之前有哪些 Pod 在运行等，这样 Master 可以获知每个 Node 的资源使用情况，并实现高效均衡的资源调度策略。而某个 Node 超过指定时间不上报信息时，会被 Master 判定为“失联”，Node 的状态被标记为不可用（Not Ready），随后 Master 会触发“工作负载大转移”的自动流程。

## 核心概念

### **Pod**

> Pod 是 K8S 中最重要也是最基本的概念，Pod 是最小的部署单元，是一组容器的集合。每个 Pod 都由一个特殊的根容器 ==Pause 容器==，以及一个或多个紧密相关的用户业务容器组成。
>
> Pause 容器作为 Pod 的根容器，以它的状态代表整个容器组的状态。K8S 为每个 Pod 都分配了唯一的 IP 地址，称之为 Pod IP。Pod 里的多个业务容器共享 Pause 容器的IP，共享 Pause 容器挂载的 Volume。

### **Label**

> 标签，附加到某个资源上，用于关联对象、查询和筛选。一个 Label 是一个 key=value 的键值对，key 与 value 由用户自己指定。Label 可以附加到各种资源上，一个资源对象可以定义任意数量的 Label，同一个 Label 也可以被添加到任意数量的资源上。
>
> 我们可以通过给指定的资源对象捆绑一个或多个不同的 Label 来实现多维度的资源分组管理功能，以便于灵活、方便地进行资源分配、调度、配置、部署等工作。
>
> K8S 通过 Label Selector（标签选择器）来查询和筛选拥有某些 Label 的资源对象。Label Selector 有基于等式（ name=label1 ）和基于集合（ name in (label1, label2) ）的两种方式。

### **ReplicaSet（RC）**

> ReplicaSet 用来确保预期的 Pod 副本数量，如果有过多的 Pod 副本在运行，系统就会停掉一些 Pod，否则系统就会再自动创建一些 Pod。
>
> 我们很少单独使用 ReplicaSet，它主要被 Deployment 这个更高层的资源对象使用，从而形成一整套 Pod 创建、删除、更新的编排机制。

### **Deployment**

> Deployment 用于部署无状态应用，Deployment 为 Pod 和 ReplicaSet 提供声明式更新，只需要在 Deployment 描述想要的目标状态，Deployment 就会将 Pod 和 ReplicaSet 的实际状态改变到目标状态。

### **Horizontal Pod Autoscaler（HPA）**

> HPA 为 Pod 横向自动扩容，也是 K8S 的一种资源对象。HPA 通过追踪分析 RC 的所有目标 Pod 的负载变化情况，来确定是否需要针对性调整目标 Pod 的副本数量。

### **Service**

> Service 定义了一个服务的访问入口，通过 Label Selector 与 Pod 副本集群之间“无缝对接”，定义了一组 Pod 的访问策略，防止 Pod 失联。
>
> 创建 Service 时，K8S会自动为它分配一个全局唯一的虚拟 IP 地址，即 ==Cluster IP。服务发现就是通过 Service 的 Name 和 Service 的 ClusterIP 地址做一个 DNS 域名映射来解决的。==

### Namespace

> 命名空间，Namespace 多用于实现多租户的资源隔离。Namespace 通过将集群内部的资源对象“分配”到不同的Namespace中，形成逻辑上分组的不同项目、小组或用户组。
>
> K8S 集群在启动后，会创建一个名为 default 的 Namespace，如果不特别指明 Namespace，创建的 Pod、RC、Service 都将被创建到 default 下。
>
> 当我们给每个租户创建一个 Namespace 来实现多租户的资源隔离时，还可以结合 K8S 的资源配额管理，限定不同租户能占用的资源，例如 CPU 使用量、内存使用量等。

## 集群搭建 —— 平台规划

### 生产环境 K8S 平台规划

K8S 环境有两种架构方式，单 Master 集群和多 Master 集群，将先搭建起单 Master 集群，再扩展为多 Master 集群。开发、测试环境可以部署单 Master 集群，生产环境为了保证高可用需部署多 Master 集群。

**① 单 Master 集群架构**

单 Master 集群架构相比于多 Master 集群架构无法保证集群的高可用，因为 master 节点一旦宕机就无法进行集群的管理工作了。单 master 集群主要包含一台 Master 节点，及多个 Node 工作节点、多个 Etcd 数据库节点。

Etcd 是 K8S 集群的数据库，可以安装在任何地方，也可以与 Master 节点在同一台机器上，只要 K8S 能连通 Etcd。

![img](https://img2018.cnblogs.com/blog/856154/201910/856154-20191029230911133-1562198790.png)

**② 多 Master 集群架构**

多 Master 集群能保证集群的高可用，相比单 Master 架构，需要一个额外的负载均衡器来负载多个 Master 节点，Node 节点从连接 Master 改成连接 LB 负载均衡器。

![img](https://img2018.cnblogs.com/blog/856154/201910/856154-20191029231927413-1474291721.png)

**③ 集群规划**

为了测试，我在本地使用 VMware 创建了几个虚拟机（可通过克隆快速创建虚拟机），一个虚拟机代表一台独立的服务器。

![img](https://img2018.cnblogs.com/blog/856154/201910/856154-20191031002248686-44613079.png)

K8S 集群规划如下：

生产环境建议至少两台 Master 节点，LB 主备各一个节点；至少两台以上 Node 节点，根据实际运行的容器数量调整；Etcd 数据库可直接部署在 Master 和 Node 的节点，机器比较充足的话，可以部署在单独的节点上。

![img](https://img2018.cnblogs.com/blog/856154/201910/856154-20191031002330707-1803081307.png)

**④ 服务器硬件配置推荐**

测试环境与生产环境服务器配置推荐如下，本地虚拟机的配置将按照本地测试环境的配置来创建虚拟机。

![img](https://img2018.cnblogs.com/blog/856154/201910/856154-20191030010523449-1281148439.png)

### 操作系统初始化

接下来将基于二进制包的方式，手动部署每个组件，来组成 K8S 高可用集群。通过手动部署每个组件，一步步熟悉每个组件的配置、组件之间的通信等，深层次的理解和掌握 K8S。

首先做的是每台服务器的配置初始化，依次按如下步骤初始化。

① 关闭防火墙

```shell
# systemctl stop firewalld
# systemctl disable firewalld
```

② 关闭 selinux

```shell
# #临时生效
# setenforce 0
# #永久生效
# sed -i 's/enforcing/disabled/' /etc/selinux/config
```

③ 关闭 swap

```shell
# #临时关闭
# swapoff -a
# # 永久生效
# vim /etc/fstab# #将 [UUID=5b59fd54-eaad-41d6-90b2-ce28ac65dd81 swap                    swap    defaults        0 0] 这一行注释掉
```

④ 添加 hosts

```shell
# vim /etc/hosts

192.168.31.24 k8s-master-1
192.168.31.26 k8s-master-2
192.168.31.35 k8s-node-1
192.168.31.71 k8s-node-2
192.168.31.178 k8s-lb-master
192.168.31.224 k8s-lb-backup
```

⑤ 同步系统时间

各个节点之间需保持时间一致，因为自签证书是根据时间校验证书有效性，如果时间不一致，将校验不通过。

```shell
# #联网情况可使用如下命令
# ntpdate time.windows.com

# #如果不能联外网可使用 date 命令设置时间
```

## 集群搭建 —— 部署Etcd集群

### 自签证书

K8S 集群安装配置过程中，会使用各种证书，目的是为了加强集群安全性。K8S 提供了基于 CA 签名的双向数字证书认证方式和简单的基于 http base 或 token 的认证方式，其中 CA 证书方式的安全性最高。每个K8S集群都有一个集群根证书颁发机构（CA），集群中的组件通常使用CA来验证API server的证书，由API服务器验证kubelet客户端证书等。

证书生成操作可以在master节点上执行，证书只需要创建一次，以后在向集群中添加新节点时只要将证书拷贝到新节点上，并做一定的配置即可。下面就在 **k8s-master-1** 节点上来创建证书，详细的介绍也可以参考官方文档：[分发自签名-CA-证书](https://kubernetes.io/zh/docs/concepts/cluster-administration/certificates/#分发自签名-ca-证书)

① K8S 证书

如下是 K8S 各个组件需要使用的证书

![img](https://img2018.cnblogs.com/blog/856154/201911/856154-20191112143250504-2083667283.png)

② 准备 cfssl 工具

我是使用 cfssl 工具来生成证书，首先下载 cfssl 工具。依次执行如下命令：

```
# curl -L https://pkg.cfssl.org/R1.2/cfssl_linux-amd64 -o /usr/local/bin/cfssl
# curl -L https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64 -o /usr/local/bin/cfssljson
# curl -L https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64 -o /usr/local/bin/cfssl-certinfo

# chmod +x /usr/local/bin/cfssl*
```

#### 自签 Etcd SSL 证书

我们首先为 **etcd** 签发一套SSL证书，通过如下命令创建几个目录，/k8s/etcd/ssl 用于存放 etcd 自签证书，/k8s/etcd/cfg 用于存放 etcd 配置文件，/k8s/etcd/bin 用于存放 etcd 执行程序。

```shell
# cd /
# mkdir -p /k8s/etcd/{ssl,cfg,bin}
```

进入 etcd 目录：

```shell
# cd /k8s/etcd/ssl
```

① 创建 CA 配置文件：**ca-config.json**

执行如下命令创建 ca-config.json

```shell
cat > ca-config.json <<EOF
{
  "signing": {
    "default": {
      "expiry": "87600h"
    },
    "profiles": {
      "etcd": {
        "usages": [
            "signing",
            "key encipherment",
            "server auth",
            "client auth"
        ],
        "expiry": "87600h"
      }
    }
  }
}
EOF
```

**说明：**

- signing：表示该证书可用于签名其它证书；生成的 ca.pem 证书中 CA=TRUE；
- profiles：可以定义多个 profiles，分别指定不同的过期时间、使用场景等参数；后续在签名证书时使用某个 profile；
- expiry：证书过期时间
- server auth：表示client可以用该 CA 对server提供的证书进行验证；
- client auth：表示server可以用该CA对client提供的证书进行验证；

② 创建 CA 证书签名请求文件：**ca-csr.json**

执行如下命令创建 ca-csr.json：

```shell
cat > ca-csr.json <<EOF
{
  "CN": "etcd",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "Shanghai",
      "L": "Shanghai",
      "O": "etcd",
      "OU": "System"
    }
  ],
    "ca": {
       "expiry": "87600h"
    }
}
EOF
```

**说明：**

- CN：Common Name，kube-apiserver 从证书中提取该字段作为请求的用户名 (User Name)；浏览器使用该字段验证网站是否合法；
- key：加密算法
- C：国家
- ST：地区
- L：城市
- O：组织，kube-apiserver 从证书中提取该字段作为请求用户所属的组 (Group)；
- OU：组织单位

③ 生成 CA 证书和私钥

```shell
# cfssl gencert -initca ca-csr.json | cfssljson -bare ca
```

![img](https://img2018.cnblogs.com/blog/856154/201911/856154-20191114232231356-1578817713.png)

**说明：**

- **ca-key.pem**：CA 私钥
- **ca.pem**：CA 数字证书

④ 创建证书签名请求文件：**etcd-csr.json**

执行如下命令创建 etcd-csr.json：

```shell
cat > etcd-csr.json <<EOF
{
    "CN": "etcd",
    "hosts": [
      "192.168.31.24",
      "192.168.31.35",
      "192.168.31.71"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "ST": "BeiJing",
            "L": "BeiJing",
            "O": "etcd",
            "OU": "System"
        }
    ]
}
EOF
```

**说明：**

- **hosts**：需要指定授权使用该证书的 IP 或域名列表，这里配置所有 etcd 的IP地址。
- **key**：加密算法及长度

⑤ 为 etcd 生成证书和私钥

```shell
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=etcd etcd-csr.json | cfssljson -bare etcd
```

**说明：**

- -ca：指定 CA 数字证书
- -ca-key：指定 CA 私钥
- -config：CA 配置文件
- -profile：指定环境
- -bare：指定证书名前缀

![img](https://img2018.cnblogs.com/blog/856154/201911/856154-20191114232316392-395705331.png)

**说明：**

- **ca-key.pem**：etcd 私钥
- **ca.pem**：etcd 数字证书

证书生成完成，后面部署 Etcd 时主要会用到如下几个证书：

![img](https://img2018.cnblogs.com/blog/856154/201911/856154-20191114232356282-2007896330.png)

### Etcd 数据库集群部署

etcd 集群采用主从架构模式（一主多从）部署，集群通过选举产生 leader，因此需要部署奇数个节点（3/5/7）才能正常工作。etcd使用raft一致性算法保证每个节点的一致性。

![img](https://img2018.cnblogs.com/blog/856154/201912/856154-20191203223130198-1515895263.png)

① 下载 etcd

从 github 上下载合适版本的 etcd

```shell
# cd /k8s/etcd
# wget https://github.com/etcd-io/etcd/releases/download/v3.2.28/etcd-v3.2.28-linux-amd64.tar.gz
```

解压：

```shell
# tar zxf etcd-v3.2.28-linux-amd64.tar.gz
```

将 etcd 复制到 /usr/local/bin 下：

```shell
# cp etcd-v3.2.28-linux-amd64/{etcd,etcdctl} /k8s/etcd/bin
# rm -rf etcd-v3.2.28-linux-amd64*
```

② 创建 etcd 配置文件：**etcd.conf**

```shell
cat > /k8s/etcd/cfg/etcd.conf <<EOF 
# [member]
ETCD_NAME=etcd-1
ETCD_DATA_DIR=/k8s/data/default.etcd
ETCD_LISTEN_PEER_URLS=https://192.168.31.24:2380
ETCD_LISTEN_CLIENT_URLS=https://192.168.31.24:2379

# [cluster]
ETCD_INITIAL_ADVERTISE_PEER_URLS=https://192.168.31.24:2380
ETCD_ADVERTISE_CLIENT_URLS=https://192.168.31.24:2379
ETCD_INITIAL_CLUSTER=etcd-1=https://192.168.31.24:2380,etcd-2=https://192.168.31.35:2380,etcd-3=https://192.168.31.71:2380
ETCD_INITIAL_CLUSTER_TOKEN=etcd-cluster
ETCD_INITIAL_CLUSTER_STATE=new

# [security]
ETCD_CERT_FILE=/k8s/etcd/ssl/etcd.pem
ETCD_KEY_FILE=/k8s/etcd/ssl/etcd-key.pem
ETCD_TRUSTED_CA_FILE=/k8s/etcd/ssl/ca.pem
ETCD_PEER_CERT_FILE=/k8s/etcd/ssl/etcd.pem
ETCD_PEER_KEY_FILE=/k8s/etcd/ssl/etcd-key.pem
ETCD_PEER_TRUSTED_CA_FILE=/k8s/etcd/ssl/ca.pem
EOF
```

**说明：**

- ETCD_NAME：etcd在集群中的唯一名称
- ETCD_DATA_DIR：etcd数据存放目录
- ETCD_LISTEN_PEER_URLS：etcd集群间通讯的地址，设置为本机IP
- ETCD_LISTEN_CLIENT_URLS：客户端访问的地址，设置为本机IP
- 
- ETCD_INITIAL_ADVERTISE_PEER_URLS：初始集群通告地址，集群内部通讯地址，设置为本机IP
- ETCD_ADVERTISE_CLIENT_URLS：客户端通告地址，设置为本机IP
- ETCD_INITIAL_CLUSTER：集群节点地址，以 key=value 的形式添加各个 etcd 的地址
- ETCD_INITIAL_CLUSTER_TOKEN：集群令牌，用于集群间做简单的认证
- ETCD_INITIAL_CLUSTER_STATE：集群状态
- 
- ETCD_CERT_FILE：客户端 etcd 数字证书路径
- ETCD_KEY_FILE：客户端 etcd 私钥路径
- ETCD_TRUSTED_CA_FILE：客户端 CA 证书路径
- ETCD_PEER_CERT_FILE：集群间通讯etcd数字证书路径
- ETCD_PEER_KEY_FILE：集群间通讯etcd私钥路径
- ETCD_PEER_TRUSTED_CA_FILE：集群间通讯CA证书路径

③ 创建 etcd 服务：**etcd.service**

通过EnvironmentFile指定 **etcd.conf** 作为环境配置文件

```shell
cat > /k8s/etcd/etcd.service <<'EOF'
[Unit]
Description=Etcd Server
After=network.target
After=network-online.target
Wants=network-online.target

[Service]
Type=notify
EnvironmentFile=/k8s/etcd/cfg/etcd.conf
WorkingDirectory=${ETCD_DATA_DIR}

ExecStart=/k8s/etcd/bin/etcd \
  --name=${ETCD_NAME} \
  --data-dir=${ETCD_DATA_DIR} \
  --listen-peer-urls=${ETCD_LISTEN_PEER_URLS} \
  --listen-client-urls=${ETCD_LISTEN_CLIENT_URLS},http://127.0.0.1:2379 \
  --initial-advertise-peer-urls=${ETCD_INITIAL_ADVERTISE_PEER_URLS} \
  --advertise-client-urls=${ETCD_ADVERTISE_CLIENT_URLS} \
  --initial-cluster=${ETCD_INITIAL_CLUSTER} \
  --initial-cluster-token=${ETCD_INITIAL_CLUSTER_TOKEN} \
  --initial-cluster-state=${ETCD_INITIAL_CLUSTER_STATE} \
  --cert-file=${ETCD_CERT_FILE} \
  --key-file=${ETCD_KEY_FILE} \
  --trusted-ca-file=${ETCD_TRUSTED_CA_FILE} \
  --peer-cert-file=${ETCD_PEER_CERT_FILE} \
  --peer-key-file=${ETCD_PEER_KEY_FILE} \
  --peer-trusted-ca-file=${ETCD_PEER_TRUSTED_CA_FILE}

Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
EOF
```

etcd.service 更多的配置以及说明可以通过如下命令查看：

```shell
# /k8s/etcd/bin/etcd --help
```

④ 将 etcd 目录拷贝到另外两个节点

```shell
# scp -r /k8s root@k8s-node-1:/k8s
# scp -r /k8s root@k8s-node-2:/k8s
```

![img](https://img2018.cnblogs.com/blog/856154/201911/856154-20191115222519900-1199048557.png)

⑤ 修改两个节点配置文件

修改 k8s-node-1 节点的 /k8s/etcd/cfg/etcd.conf：

```shell
# [member]
ETCD_NAME=etcd-2
ETCD_LISTEN_PEER_URLS=https://192.168.31.35:2380
ETCD_LISTEN_CLIENT_URLS=https://192.168.31.35:2379

# [cluster]
ETCD_INITIAL_ADVERTISE_PEER_URLS=https://192.168.31.35:2380
ETCD_ADVERTISE_CLIENT_URLS=https://192.168.31.35:2379
```

修改 k8s-node-2 节点的 /k8s/etcd/cfg/etcd.conf：

```shell
# [member]
ETCD_NAME=etcd-2
ETCD_LISTEN_PEER_URLS=https://192.168.31.71:2380
ETCD_LISTEN_CLIENT_URLS=https://192.168.31.71:2379

# [cluster]
ETCD_INITIAL_ADVERTISE_PEER_URLS=https://192.168.31.71:2380
ETCD_ADVERTISE_CLIENT_URLS=https://192.168.31.71:2379
```

⑥ 启动 etcd 服务

首先在三个节点将 **etcd.service** 拷贝到 /usr/lib/systemd/system/ 下

```shell
# cp /k8s/etcd/etcd.service /usr/lib/systemd/system/
# systemctl daemon-reload
```

在三个节点启动 etcd 服务

```shell
# systemctl start etcd
```

设置开机启动

```shell
# systemctl enable etcd
```

查看 etcd 的日志

```shell
# tail -f /var/log/messages
```

![img](https://img2018.cnblogs.com/blog/856154/201911/856154-20191115232442732-2092355582.png)

注意：如果日志中出现连接异常信息，请确认所有节点防火墙是否开放2379,2380端口，或者直接关闭防火墙。

查看 etcd 集群状态

```shell
/k8s/etcd/bin/etcdctl \
    --ca-file=/k8s/etcd/ssl/ca.pem \
    --cert-file=/k8s/etcd/ssl/etcd.pem \
    --key-file=/k8s/etcd/ssl/etcd-key.pem \
    --endpoints=https://192.168.31.24:2379,https://192.168.31.35:2379,https://192.168.31.71:2379 \
cluster-health
```

![img](https://img2018.cnblogs.com/blog/856154/201911/856154-20191115235034288-1984020155.png)

## 集群搭建 —— 部署Master组件

### 自签 ApiServer SSL 证书

K8S 集群中所有资源的访问和变更都是通过 kube-apiserver 的 REST API 来实现的，首先在 master 节点上部署 kube-apiserver 组件。

我们首先为 **apiserver** 签发一套SSL证书，过程与 etcd 自签SSL证书类似。通过如下命令创建几个目录，ssl 用于存放自签证书，cfg 用于存放配置文件，bin 用于存放执行程序，logs 存放日志文件。

```shell
# cd /
# mkdir -p /k8s/kubernetes/{ssl,cfg,bin,logs}
```

进入 kubernetes 目录：

```shell
# cd /k8s/kubernetes/ssl
```

① 创建 CA 配置文件：**ca-config.json**

执行如下命令创建 ca-config.json

```shell
cat > ca-config.json <<EOF
{
  "signing": {
    "default": {
      "expiry": "87600h"
    },
    "profiles": {
      "kubernetes": {
        "usages": [
            "signing",
            "key encipherment",
            "server auth",
            "client auth"
        ],
        "expiry": "87600h"
      }
    }
  }
}
EOF
```

② 创建 CA 证书签名请求文件：**ca-csr.json**

执行如下命令创建 ca-csr.json：

```shell
cat > ca-csr.json <<EOF
{
  "CN": "kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "Shanghai",
      "L": "Shanghai",
      "O": "kubernetes",
      "OU": "System"
    }
  ],
    "ca": {
       "expiry": "87600h"
    }
}
EOF
```

③ 生成 CA 证书和私钥

```shell
# cfssl gencert -initca ca-csr.json | cfssljson -bare ca
```

④ 创建证书签名请求文件：**kubernetes-csr.json**

执行如下命令创建 kubernetes-csr.json：

```shell
cat > kubernetes-csr.json <<EOF
{
    "CN": "kubernetes",
    "hosts": [
      "127.0.0.1",
      "10.0.0.1",
      "192.168.31.24",
      "192.168.31.26",
      "192.168.31.35",
      "192.168.31.71",
      "192.168.31.26",
      "192.168.31.26",
      "kubernetes",
      "kubernetes.default",
      "kubernetes.default.svc",
      "kubernetes.default.svc.cluster",
      "kubernetes.default.svc.cluster.local"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "ST": "BeiJing",
            "L": "BeiJing",
            "O": "kubernetes",
            "OU": "System"
        }
    ]
}
EOF
```

**说明：**

- **hosts**：指定会直接访问 apiserver 的IP列表，一般需指定 etcd 集群、kubernetes master 集群的主机 IP 和 kubernetes 服务的服务 IP，Node 的IP一般不需要加入。

⑤ 为 kubernetes 生成证书和私钥

```shell
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kubernetes-csr.json | cfssljson -bare kubernetes
```

### 部署 kube-apiserver 组件

① 下载二进制包

通过 kubernetes Github 下载安装用的二进制包，我这里使用 **[v1.16.2](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.16.md#downloads-for-v1163)** 版本，server 二进制包已经包含了 master、node 上的各个组件，下载 server 二进制包即可。

考虑到网络问题，可以从百度网盘下载已经准备好的二进制包：链接: [下载离线安装包](https://pan.baidu.com/s/1LAaBajkPIWTU1Gd4Ez9ztw)。

将下载好的 kubernetes-v1.16.2-server-linux-amd64.tar.gz 上传到 /usr/local/src下，并解压：

```shell
# tar -zxf kubernetes-v1.16.2-server-linux-amd64.tar.gz
```

先将 master 节点上部署的组件拷贝到 /k8s/kubernetes/bin 目录下：

```shell
# cp -p /usr/local/src/kubernetes/server/bin/{kube-apiserver,kube-controller-manager,kube-scheduler} /k8s/kubernetes/bin/

# cp -p /usr/local/src/kubernetes/server/bin/kubectl /usr/local/bin/
```

② 创建 Node 令牌文件：**token.csv**

Master apiserver 启用 TLS 认证后，Node节点 kubelet 组件想要加入集群，必须使用CA签发的有效证书才能与apiserver通信，当Node节点很多时，签署证书是一件很繁琐的事情，因此有了 TLS Bootstrap 机制，kubelet 会以一个低权限用户自动向 apiserver 申请证书，kubelet 的证书由 apiserver 动态签署。因此先为 apiserver 生成一个令牌文件，令牌之后会在 Node 中用到。

生成 token，一个随机字符串，可使用如下命令生成 token：apiserver 配置的 token 必须与 Node 节点 bootstrap.kubeconfig 配置保持一致。

```shell
# head -c 16 /dev/urandom | od -An -t x | tr -d ' '
```

![img](https://img2018.cnblogs.com/blog/856154/201911/856154-20191120142640146-1408433933.png)

创建 token.csv，格式：token，用户，UID，用户组

```shell
cat > /k8s/kubernetes/cfg/token.csv <<'EOF'
bfa3cb7f6f21f87e5c0e5f25e6cfedad,kubelet-bootstrap,10001,"system:node-bootstrapper"
EOF
```

③ 创建 kube-apiserver 配置文件：**kube-apiserver.conf**

kube-apiserver 有很多配置项，可以参考官方文档查看每个配置项的用途：[kube-apiserver](https://kubernetes.io/zh/docs/reference/command-line-tools-reference/kube-apiserver/)

注意：踩的一个坑，“\” 后面不要有空格，不要有多余的换行，否则启动失败。

```shell
cat > /k8s/kubernetes/cfg/kube-apiserver.conf <<'EOF'
KUBE_APISERVER_OPTS="--etcd-servers=https://192.168.31.24:2379,https://192.168.31.35:2379,https://192.168.31.71:2379 \
  --bind-address=192.168.31.24 \
  --secure-port=6443 \
  --advertise-address=192.168.31.24 \
  --allow-privileged=true \
  --service-cluster-ip-range=10.0.0.0/24 \
  --service-node-port-range=30000-32767 \
  --enable-admission-plugins=NamespaceLifecycle,LimitRanger,ServiceAccount,ResourceQuota,NodeRestriction \
  --authorization-mode=RBAC,Node \
  --enable-bootstrap-token-auth=true \
  --token-auth-file=/k8s/kubernetes/cfg/token.csv \
  --kubelet-client-certificate=/k8s/kubernetes/ssl/kubernetes.pem \
  --kubelet-client-key=/k8s/kubernetes/ssl/kubernetes-key.pem \
  --tls-cert-file=/k8s/kubernetes/ssl/kubernetes.pem \
  --tls-private-key-file=/k8s/kubernetes/ssl/kubernetes-key.pem \
  --client-ca-file=/k8s/kubernetes/ssl/ca.pem \
  --service-account-key-file=/k8s/kubernetes/ssl/ca-key.pem \
  --etcd-cafile=/k8s/etcd/ssl/ca.pem \
  --etcd-certfile=/k8s/etcd/ssl/etcd.pem \
  --etcd-keyfile=/k8s/etcd/ssl/etcd-key.pem \
  --v=2 \
  --logtostderr=false \
  --log-dir=/k8s/kubernetes/logs \
  --audit-log-maxage=30 \
  --audit-log-maxbackup=3 \
  --audit-log-maxsize=100 \
  --audit-log-path=/k8s/kubernetes/logs/k8s-audit.log"
EOF
```

**重点配置说明：**

- --etcd-servers：etcd 集群地址
- --bind-address：apiserver 监听的地址，一般配主机IP
- --secure-port：监听的端口
- --advertise-address：集群通告地址，其它Node节点通过这个地址连接 apiserver，不配置则使用 --bind-address
- --service-cluster-ip-range：Service 的 虚拟IP范围，以CIDR格式标识，该IP范围不能与物理机的真实IP段有重合。
- --service-node-port-range：Service 可映射的物理机端口范围，默认30000-32767
- --admission-control：集群的准入控制设置，各控制模块以插件的形式依次生效，启用RBAC授权和节点自管理
- --authorization-mode：授权模式，包括：AlwaysAllow，AlwaysDeny，ABAC(基于属性的访问控制)，Webhook，RBAC(基于角色的访问控制)，Node(专门授权由 kubelet 发出的API请求)。（默认值"AlwaysAllow"）。
- --enable-bootstrap-token-auth：启用TLS bootstrap功能
- --token-auth-file：这个文件将被用于通过令牌认证来保护API服务的安全端口。
- --v：指定日志级别，0~8，越大日志越详细

④ 创建 apiserver 服务：**kube-apiserver.service**

```shell
cat > /usr/lib/systemd/system/kube-apiserver.service <<'EOF'
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/GoogleCloudPlatform/kubernetes
After=network.target

[Service]
EnvironmentFile=-/k8s/kubernetes/cfg/kube-apiserver.conf
ExecStart=/k8s/kubernetes/bin/kube-apiserver $KUBE_APISERVER_OPTS
Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
EOF
```

⑤ 启动 kube-apiserver 组件

启动组件

```shell
# systemctl daemon-reload
# systemctl start kube-apiserver
# systemctl enable kube-apiserver
```

检查启动状态

```shell
# systemctl status kube-apiserver.service
```

查看启动日志

```shell
# less /k8s/kubernetes/logs/kube-apiserver.INFO
```

⑥ 将 kubelet-bootstrap 用户绑定到系统集群角色，之后便于 Node 使用token请求证书

```shell
kubectl create clusterrolebinding kubelet-bootstrap \
  --clusterrole=system:node-bootstrapper \
  --user=kubelet-bootstrap
```

### 部署 kube-controller-manager 组件

① 创建 kube-controller-manager 配置文件：**kube-controller-manager.conf**

详细的配置可参考官方文档：[kube-controller-manager](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-controller-manager/)

```shell
cat > /k8s/kubernetes/cfg/kube-controller-manager.conf <<'EOF'
KUBE_CONTROLLER_MANAGER_OPTS="--leader-elect=true \
  --master=127.0.0.1:8080 \
  --address=127.0.0.1 \
  --allocate-node-cidrs=true \
  --cluster-cidr=10.244.0.0/16 \
  --service-cluster-ip-range=10.0.0.0/24 \
  --cluster-signing-cert-file=/k8s/kubernetes/ssl/ca.pem \
  --cluster-signing-key-file=/k8s/kubernetes/ssl/ca-key.pem \
  --root-ca-file=/k8s/kubernetes/ssl/ca.pem \
  --service-account-private-key-file=/k8s/kubernetes/ssl/ca-key.pem \
  --experimental-cluster-signing-duration=87600h0m0s \
  --v=2 \
  --logtostderr=false \
  --log-dir=/k8s/kubernetes/logs"
EOF
```

**重点配置说明：**

- --leader-elect：当该组件启动多个时，自动选举，默认true
- --master：连接本地apiserver，apiserver 默认会监听本地8080端口
- --allocate-node-cidrs：是否分配和设置Pod的CDIR
- --service-cluster-ip-range：Service 集群IP段

② 创建 kube-controller-manager 服务：**kube-controller-manager.service**

```shell
cat > /usr/lib/systemd/system/kube-controller-manager.service <<'EOF'
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/GoogleCloudPlatform/kubernetes
After=network.target

[Service]
EnvironmentFile=/k8s/kubernetes/cfg/kube-controller-manager.conf
ExecStart=/k8s/kubernetes/bin/kube-controller-manager $KUBE_CONTROLLER_MANAGER_OPTS
Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
EOF
```

③ 启动 kube-controller-manager 组件

启动组件

```shell
# systemctl daemon-reload
# systemctl start kube-controller-manager
# systemctl enable kube-controller-manager
```

检查启动状态

```shell
# systemctl status kube-controller-manager.service
```

查看启动日志

```shell
# less /k8s/kubernetes/logs/kube-controller-manager.INFO
```

### 部署 kube-scheduler 组件

① 创建 kube-scheduler 配置文件：**kube-scheduler.conf**

```shell
cat > /k8s/kubernetes/cfg/kube-scheduler.conf <<'EOF'
KUBE_SCHEDULER_OPTS="--leader-elect=true \
  --master=127.0.0.1:8080 \
  --address=127.0.0.1 \
  --v=2 \
  --logtostderr=false \
  --log-dir=/k8s/kubernetes/logs"
EOF
```

② 创建 kube-scheduler 服务：**kube-scheduler.service**

```shell
cat > /usr/lib/systemd/system/kube-scheduler.service <<'EOF'
[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/GoogleCloudPlatform/kubernetes
After=network.target

[Service]
EnvironmentFile=/k8s/kubernetes/cfg/kube-scheduler.conf
ExecStart=/k8s/kubernetes/bin/kube-scheduler $KUBE_SCHEDULER_OPTS
Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
EOF
```

③ 启动 kube-scheduler 组件

启动组件

```shell
# systemctl daemon-reload
# systemctl start kube-scheduler
# systemctl enable kube-scheduler
```

查看启动状态

```shell
# systemctl status kube-scheduler.service
```

查看启动日志

```shell
# less /k8s/kubernetes/logs/kube-scheduler.INFO
less /k8s/kubernetes/logs/kube-scheduler.INFO
```

### 查看集群状态

① 查看组件状态

```shell
# kubectl get cs
```

![img](https://img2018.cnblogs.com/blog/856154/201911/856154-20191120235101404-1573446834.png)

## 集群搭建 —— 部署Node组件

### 安装 Docker

CentOS 安装参考官方文档：https://docs.docker.com/install/linux/docker-ce/centos/

① 卸载旧版本

```shell
# yum remove docker docker-common docker-selinux
```

② 安装依赖包

```shell
# yum install -y yum-utils device-mapper-persistent-data lvm2
```

③ 安装 Docker 软件包源

```shell
# yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

④ 安装 Docker CE

```shell
# yum install docker-ce
```

⑤ 启动 Docker 服务

```shell
# systemctl start docker
```

⑥ 设置开机启动

```shell
# systemctl enable docker
```

⑦ 验证安装是否成功

```shell
# docker -v
# docker info
```

![img](https://img2018.cnblogs.com/blog/856154/201908/856154-20190806005734464-186989792.png)

### Node 节点证书

① 创建 Node 节点的证书签名请求文件：**kube-proxy-csr.json**

首先在 k8s-master-1 节点上，通过颁发的 CA 证书先创建好 Node 节点要使用的证书，先创建证书签名请求文件：kube-proxy-csr.json：

```shell
cat > kube-proxy-csr.json <<EOF
{
    "CN": "system:kube-proxy",
    "hosts": [],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "ST": "BeiJing",
            "L": "BeiJing",
            "O": "kubernetes",
            "OU": "System"
        }
    ]
}
EOF
```

② 为 kube-proxy 生成证书和私钥

```shell
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kube-proxy-csr.json | cfssljson -bare kube-proxy
```

![img](https://img2018.cnblogs.com/blog/856154/201911/856154-20191127220706167-1455473162.png)

③ node 节点创建工作目录

在 k8s-node-1 节点上创建 k8s 目录

```shell
# mkdir -p /k8s/kubernetes/{bin,cfg,logs,ssl}
```

④ 将 k8s-master-1 节点的文件拷贝到 node 节点

将 kubelet、kube-proxy 拷贝到 node 节点上：

```shell
# scp -r /usr/local/src/kubernetes/server/bin/{kubelet,kube-proxy} root@k8s-node-1:/k8s/kubernetes/bin/
```

将证书拷贝到 k8s-node-1 节点上：

```shell
# scp -r /k8s/kubernetes/ssl/{ca.pem,kube-proxy.pem,kube-proxy-key.pem} root@k8s-node-1:/k8s/kubernetes/ssl/
```

### 安装 kubelet

① 创建请求证书的配置文件：**bootstrap.kubeconfig**

bootstrap.kubeconfig 将用于向 apiserver 请求证书，apiserver 会验证 token、证书 是否有效，验证通过则自动颁发证书。

```shell
cat > /k8s/kubernetes/cfg/bootstrap.kubeconfig <<'EOF'
apiVersion: v1
clusters:
- cluster: 
    certificate-authority: /k8s/kubernetes/ssl/ca.pem
    server: https://192.168.31.24:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubelet-bootstrap
  name: default
current-context: default
kind: Config
preferences: {}
users:
- name: kubelet-bootstrap
  user:
    token: bfa3cb7f6f21f87e5c0e5f25e6cfedad
EOF
```

**说明：**

- certificate-authority：CA 证书
- server：master 地址
- token：master 上 token.csv 中配置的 token

② 创建 kubelet 配置文件：**kubelet-config.yml**

为了安全性，kubelet 禁止匿名访问，必须授权才可以，通过 kubelet-config.yml 授权 apiserver 访问 kubelet。

```shell
cat > /k8s/kubernetes/cfg/kubelet-config.yml <<'EOF'
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
address: 0.0.0.0
port: 10250
readOnlyPort: 10255
cgroupDriver: cgroupfs
clusterDNS:
- 10.0.0.2 
clusterDomain: cluster.local
failSwapOn: false
authentication:
  anonymous:
    enabled: false
  webhook:
    cacheTTL: 2m0s
    enabled: true
  x509: 
    clientCAFile: /k8s/kubernetes/ssl/ca.pem
authorization:
  mode: Webhook
  webhook:
    cacheAuthroizedTTL: 5m0s
    cacheUnauthorizedTTL: 30s
evictionHard:
  imagefs.available: 15%
  memory.available: 100Mi
  nodefs.available: 10%
  nodefs.inodesFree: 5%
maxOpenFiles: 100000
maxPods: 110
EOF
```

**说明：**

- address：kubelet 监听地址
- port：kubelet 的端口
- cgroupDriver：cgroup 驱动，与 docker 的 cgroup 驱动一致
- authentication：访问 kubelet 的授权信息
- authorization：认证相关信息
- evictionHard：垃圾回收策略
- maxPods：最大pod数

③ 创建 kubelet 服务配置文件：**kubelet.conf**

```shell
cat > /k8s/kubernetes/cfg/kubelet.conf <<'EOF'
KUBELET_OPTS="--hostname-override=k8s-node-1 \
  --network-plugin=cni \
  --cni-bin-dir=/opt/cni/bin \
  --cni-conf-dir=/etc/cni/net.d \
  --cgroups-per-qos=false \
  --enforce-node-allocatable="" \
  --kubeconfig=/k8s/kubernetes/cfg/kubelet.kubeconfig \
  --bootstrap-kubeconfig=/k8s/kubernetes/cfg/bootstrap.kubeconfig \
  --config=/k8s/kubernetes/cfg/kubelet-config.yml \
  --cert-dir=/k8s/kubernetes/ssl \
  --pod-infra-container-image=kubernetes/pause:latest \
  --v=2 \
  --logtostderr=false \
  --log-dir=/k8s/kubernetes/logs"
EOF
```

**说明：**

- --hostname-override：当前节点注册到K8S中显示的名称，默认为主机 hostname
- --network-plugin：启用 CNI 网络插件
- --cni-bin-dir：CNI 插件可执行文件位置，默认在 /opt/cni/bin 下
- --cni-conf-dir：CNI 插件配置文件位置，默认在 /etc/cni/net.d 下
- --cgroups-per-qos：必须加上这个参数和--enforce-node-allocatable，否则报错 [Failed to start ContainerManager failed to initialize top level QOS containers.......]
- --kubeconfig：会自动生成 kubelet.kubeconfig，用于连接 apiserver
- --bootstrap-kubeconfig：指定 bootstrap.kubeconfig 文件
- --config：kubelet 配置文件
- --cert-dir：证书目录
- --pod-infra-container-image：管理Pod网络的镜像，基础的 Pause 容器，默认是 k8s.gcr.io/pause:3.1

④ 创建 kubelet 服务：**kubelet.service**

```shell
cat > /usr/lib/systemd/system/kubelet.service <<'EOF'
[Unit]
Description=Kubernetes Kubelet
After=docker.service
Before=docker.service

[Service]
EnvironmentFile=/k8s/kubernetes/cfg/kubelet.conf
ExecStart=/k8s/kubernetes/bin/kubelet $KUBELET_OPTS
Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
EOF
```

⑤ 启动 kubelet

```shell
# systemctl daemon-reload
# systemctl start kubelet
```

开机启动：

```shell
# systemctl enable kubelet
```

查看启动日志： 

```shell
# tail -f /k8s/kubernetes/logs/kubelet.INFO
```

⑥ master 给 node 授权

kubelet 启动后，还没加入到集群中，会向 apiserver 请求证书，需手动在 k8s-master-1 上对 node 授权。

通过如下命令查看是否有新的客户端请求颁发证书：

```shell
# kubectl get csr
```

![img](https://img2018.cnblogs.com/blog/856154/201911/856154-20191127220530728-425743917.png)

给客户端颁发证书，允许客户端加入集群：

```shell
# kubectl certificate approve node-csr-FoPLmv3Sr2XcYvNAineE6RpdARf2eKQzJsQyfhk-xf8
```

![img](https://img2018.cnblogs.com/blog/856154/201911/856154-20191127220827044-1194859709.png)

⑦ 授权成功

查看 node 是否加入集群(此时的 node 还处于未就绪的状态，因为还没有安装 CNI 组件)：

```shell
# kubectl get node
```

![img](https://img2018.cnblogs.com/blog/856154/201911/856154-20191127220909837-1664629101.png)

颁发证书后，可以在 /k8s/kubenetes/ssl 下看到 master 为 kubelet 颁发的证书：

![img](https://img2018.cnblogs.com/blog/856154/201911/856154-20191127221220075-1579437290.png)

在 /k8s/kubenetes/cfg 下可以看到自动生成的 kubelet.kubeconfig 配置文件：

![img](https://img2018.cnblogs.com/blog/856154/201911/856154-20191127221332515-613675599.png)

### 安装 kube-proxy

① 创建 kube-proxy 连接 apiserver 的配置文件：**kube-proxy.kubeconfig**

```shell
cat > /k8s/kubernetes/cfg/kube-proxy.kubeconfig <<'EOF'
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /k8s/kubernetes/ssl/ca.pem
    server: https://192.168.31.24:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kube-proxy
  name: default
current-context: default
kind: Config
preferences: {}
users:
- name: kube-proxy
  user:
    client-certificate: /k8s/kubernetes/ssl/kube-proxy.pem
    client-key: /k8s/kubernetes/ssl/kube-proxy-key.pem
EOF
```

② 创建 kube-proxy 配置文件：**kube-proxy-config.yml**

```shell
cat > /k8s/kubernetes/cfg/kube-proxy-config.yml <<'EOF'
kind: KubeProxyConfiguration
apiVersion: kubeproxy.config.k8s.io/v1alpha1
address: 0.0.0.0
metrisBindAddress: 0.0.0.0:10249
clientConnection:
  kubeconfig: /k8s/kubernetes/cfg/kube-proxy.kubeconfig
hostnameOverride: k8s-node-1
clusterCIDR: 10.0.0.0/24
mode: ipvs
ipvs:
  scheduler: "rr"
iptables:
  masqueradeAll: true
EOF
```

**说明：**

- metrisBindAddress：采集指标暴露的地址端口，便于监控系统，采集数据
- clusterCIDR：集群 Service 网段

③ 创建 kube-proxy 配置文件：**kube-proxy.conf**

```shell
cat > /k8s/kubernetes/cfg/kube-proxy.conf <<'EOF'
KUBE_PROXY_OPTS="--config=/k8s/kubernetes/cfg/kube-proxy-config.yml \
  --v=2 \
  --logtostderr=false \
  --log-dir=/k8s/kubernetes/logs"
EOF
```

④ 创建 kube-proxy 服务：**kube-proxy.service**

```shell
cat > /usr/lib/systemd/system/kube-proxy.service <<'EOF'
[Unit]
Description=Kubernetes Proxy
After=network.target

[Service]
EnvironmentFile=/k8s/kubernetes/cfg/kube-proxy.conf
ExecStart=/k8s/kubernetes/bin/kube-proxy $KUBE_PROXY_OPTS
Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
EOF
```

⑤ 启动 kube-proxy

启动服务：

```shell
# systemctl daemon-reload
# systemctl start kube-proxy
```

开机启动：

```shell
# systemctl enable kube-proxy
```

查看启动日志： 

```shell
# tail -f /k8s/kubernetes/logs/kube-proxy.INFO
```

### 部署其它Node节点

部署其它 Node 节点基于与上述流程一致，只需将配置文件中 k8s-node-1 改为 k8s-node-2 即可。

```
# kubectl get node -o wide
```

![img](https://img2018.cnblogs.com/blog/856154/201911/856154-20191128224015295-979089341.png)

### 部署K8S容器集群网络(Flannel)

① K8S 集群网络

Kubernetes 项目并没有使用 Docker 的网络模型，kubernetes 是通过一个 CNI 接口维护一个单独的网桥来代替 docker0，这个网桥默认叫 **cni0**。

CNI（Container Network Interface）是CNCF旗下的一个项目，由一组用于配置 Linux 容器的网络接口的规范和库组成，同时还包含了一些插件。CNI仅关心容器创建时的网络分配，和当容器被删除时释放网络资源。

Flannel 是 CNI 的一个插件，可以看做是 CNI 接口的一种实现。Flannel 是针对 Kubernetes 设计的一个网络规划服务，它的功能是让集群中的不同节点主机创建的Docker容器都具有全集群唯一的虚拟IP地址，并让属于不同节点上的容器能够直接通过内网IP通信。

Flannel 网络架构请参考：[flannel 网络架构](https://ggaaooppeenngg.github.io/zh-CN/2017/09/21/flannel-网络架构/)

② 创建 CNI 工作目录

通过给 kubelet 传递 **--network-plugin=cni** 命令行选项来启用 CNI 插件。 kubelet 从 **--cni-conf-dir** （默认是 /etc/cni/net.d）读取配置文件并使用该文件中的 CNI 配置来设置每个 pod 的网络。CNI 配置文件必须与 CNI 规约匹配，并且配置引用的任何所需的 CNI 插件都必须存在于 **--cni-bin-dir**（默认是 /opt/cni/bin）指定的目录。

由于前面部署 kubelet 服务时，指定了 **--cni-conf-dir=/etc/cni/net.d**，**--cni-bin-dir=/opt/cni/bin**，因此首先在node节点上创建这两个目录：

```shell
# mkdir -p /opt/cni/bin /etc/cni/net.d
```

③ 装 CNI 插件

可以从 github 上下载 CNI 插件：[下载 CNI 插件](https://github.com/containernetworking/plugins/releases) 。

解压到 /opt/cni/bin：

```shell
# tar zxf cni-plugins-linux-amd64-v0.8.2.tgz -C /opt/cni/bin/
```

![img](https://img2018.cnblogs.com/blog/856154/201911/856154-20191128220554099-1684002659.png)

④ 部署 Flannel

可通过此地址下载 flannel 配置文件：[下载 kube-flannel.yml](https://raw.githubusercontent.com/coreos/flannel/2140ac876ef134e0ed5af15c65e414cf26827915/Documentation/kube-flannel.yml)

注意如下配置：Network 的地址需与 **kube-controller-manager.conf** 中的 **--cluster-cidr=10.244.0.0/16** 保持一致。

![img](https://img2018.cnblogs.com/blog/856154/201911/856154-20191128112822714-1964053859.png)

在 k8s-master-1 节点上部署 Flannel：

```shell
# kubectl apply -f kube-flannel.yml
```

![img](https://img2018.cnblogs.com/blog/856154/201911/856154-20191128230506906-1309290806.png)

⑤ 检查部署状态

Flannel 会在 Node 上起一个 Flannel 的 Pod，可以查看 pod 的状态看 flannel 是否启动成功：

```shell
# kubectl get pods -n kube-system -o wide
```

![img](https://img2018.cnblogs.com/blog/856154/201912/856154-20191205005818391-377110510.png)

 Flannel 部署成功后，就可以看 Node 是否就绪：

```shell
# kubectl get nodes -o wide
```

![img](https://img2018.cnblogs.com/blog/856154/201912/856154-20191205010005973-37990797.png)

在 Node 上查看网络配置，可以看到多了一个 **flannel.1** 的虚拟网卡，这块网卡用于接收 Pod 的流量并转发出去。

![img](https://img2018.cnblogs.com/blog/856154/201912/856154-20191205010303481-754515627.png)

⑥ 测试创建 Pod

例如创建一个 Nginx 服务：

```shell
# kubectl create deployment web --image=nginx
```

![img](https://img2018.cnblogs.com/blog/856154/201912/856154-20191205010658719-540734499.png)

查看 Pod 状态：

![img](https://img2018.cnblogs.com/blog/856154/201912/856154-20191205010735690-1937177430.png)

在对应的节点上可以看到部署的容器：

![img](https://img2018.cnblogs.com/blog/856154/201912/856154-20191205012234493-297995248.png)

容器创建成功后，再在 Node 上查看网络配置，又多了一块 cni0 的虚拟网卡，cni0 用于 pod 本地通信使用。

![img](https://img2018.cnblogs.com/blog/856154/201912/856154-20191205010844816-242752920.png)

暴露端口并访问 Nginx：

```shell
# kubectl expose deployment web --port=80 --type=NodePort
```

![img](https://img2018.cnblogs.com/blog/856154/201912/856154-20191205012049946-519639325.png)

### 部署内部 DNS 服务

在Kubernetes集群推荐使用Service Name作为服务的访问地址，因此需要一个Kubernetes集群范围的DNS服务实现从Service Name到Cluster IP的解析，这就是Kubernetes基于DNS的服务发现功能。

① 部署 CoreDNS

下载CoreDNS配置文件：[coredns.yaml](https://raw.githubusercontent.com/kubernetes/kubernetes/master/cluster/addons/dns/coredns/coredns.yaml.base)

注意如下 clusterIP 一定要与 **kube-config.yaml** 中的 **clusterDNS** 保持一致

![img](https://img2018.cnblogs.com/blog/856154/201912/856154-20191206013327734-494461951.png)

部署 CoreDNS：

```shell
$ kubectl apply -f coredns.yaml
```

![img](https://img2018.cnblogs.com/blog/856154/201912/856154-20191208235827818-71864912.png)

② 验证 CoreDNS

创建 busybox 服务：

```shell
# cat > busybox.yaml <<'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: busybox
  namespace: default
spec:
  dnsPolicy: ClusterFirst
  containers:
  - name: busybox
    image: busybox:1.28.4
    command:
      - sleep
      - "3600"
    imagePullPolicy: IfNotPresent
  restartPolicy: Always
EOF# kubectl apply -f busybox.yaml
```

验证是否安装成功：

![img](https://img2018.cnblogs.com/blog/856154/201912/856154-20191209011552538-431087460.png)

## 集群搭建 —— 部署 Dashboard

K8S 提供了一个 Web 版 Dashboard，用户可以用 dashboard 部署容器化的应用、监控应用的状态，能够创建和修改各种 K8S 资源，比如 Deployment、Job、DaemonSet 等。用户可以 Scale Up/Down Deployment、执行 Rolling Update、重启某个 Pod 或者通过向导部署新的应用。Dashboard 能显示集群中各种资源的状态以及日志信息。Kubernetes Dashboard 提供了 kubectl 的绝大部分功能。

### 部署 K8S Dashboard

通过此地址下载 dashboard yaml文件：[kubernetes-dashboard.yaml](http://mirror.faasx.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml)

下载下来之后，需更新如下内容：通过 Node 暴露端口访问 dashboard

![img](https://img2018.cnblogs.com/blog/856154/201912/856154-20191206010304862-1366726165.png)

部署 dashboard：

```shell
# kubectl apply -f kubernetes-dashboard.yaml
```

查看是否部署成功：

![img](https://img2018.cnblogs.com/blog/856154/201912/856154-20191206010403304-236700234.png)

通过 https 访问 dashboard：

![img](https://img2018.cnblogs.com/blog/856154/201912/856154-20191206010519105-236777362.png)

### 登录授权

Dashboard ==支持 Kubeconfig 和 Token 两种认证方式==，为了简化配置，我们通过配置文件为 Dashboard 默认用户赋予 admin 权限。

```shell
cat > kubernetes-adminuser.yaml <<'EOF'
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kube-system
EOF
```

授权：

```shell
# kubectl apply -f kubernetes-adminuser.yaml
```

获取登录的 token：

```shell
# kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk ' {print $1}')
```

![img](https://img2018.cnblogs.com/blog/856154/201912/856154-20191206011211859-592094825.png)

通过token登录进 dashboard，就可以查看集群的信息：

![img](https://img2018.cnblogs.com/blog/856154/201912/856154-20191206011456607-1992406368.png)

## 集群搭建 —— 多 Master 部署

### 部署Master2组件

① 将 k8s-master-1 上相关文件拷贝到 k8s-master-2 上

创建k8s工作目录：

```shell
# mkdir -p /k8s/kubernetes
# mkdir -p /k8s/etcd
```

拷贝 k8s 配置文件、执行文件、证书：

```shell
# scp -r /k8s/kubernetes/{cfg,ssl,bin} root@k8s-master-2:/k8s/kubernetes
# cp /k8s/kubernetes/bin/kubectl /usr/local/bin/
```

拷贝 etcd 证书：

```shell
# scp -r /k8s/etcd/ssl root@k8s-master-2:/k8s/etcd
```

拷贝 k8s 服务的service文件：

```shell
# scp /usr/lib/systemd/system/kube-* root@k8s-master-2:/usr/lib/systemd/system
```

② 修改 k8s-master-2 上的配置文件

修改 kube-apiserver.conf，修改IP为本机IP

 

![img](https://img2018.cnblogs.com/blog/856154/201912/856154-20191210231658832-199791846.png)

③ 启动 k8s-master-2 组件

重新加载配置：

```shell
# systemctl daemon-reload
```

启动 kube-apiserver：

```shell
# systemctl start kube-apiserver
# systemctl enable kube-apiserver
```

启动 kube-controller-manager：

```shell
# systemctl start kube-controller-manager
# systemctl enable kube-controller-manager
```

部署 kube-scheduler：

```shell
# systemctl start kube-scheduler
# systemctl enable kube-scheduler
```

④ 验证

![img](https://img2018.cnblogs.com/blog/856154/201912/856154-20191210233157623-672470319.png)

### 部署 Nginx 负载均衡

为了保证 k8s master 的高可用，将使用 k8s-lb-master 和 k8s-lb-backup 这两台机器来部署负载均衡。这里使用 nginx 做负载均衡器，下面分别在 k8s-lb-master 和 k8s-lb-backup 这两台机器上部署 nginx。

① gcc等环境安装，后续有些软件安装需要这些基础环境

```shell
# gcc安装：
# yum install gcc-c++
# PCRE pcre-devel 安装：
# yum install -y pcre pcre-devel
# zlib 安装：
# yum install -y zlib zlib-devel
#OpenSSL 安装：
# yum install -y openssl openssl-devel
```

② 安装nginx

```shell
# rpm -ivh https://nginx.org/packages/rhel/7/x86_64/RPMS/nginx-1.16.1-1.el7.ngx.x86_64.rpm
```

③ apiserver 负载配置

```shell
# vim /etc/nginx/nginx.conf
```

增加如下配置：

```shell
stream {
    log_format main '$remote_addr $upstream_addr - [$time_local] $status $upstream_bytes_sent';
    access_log /var/log/nginx/k8s-access.log main;

    upstream k8s-apiserver {
        server 192.168.31.24:6443;
        server 192.168.31.26:6443;
    }

    server {
        listen 6443;
        proxy_pass k8s-apiserver;
    }
}
```

④ 启动 nginx

```shell
# systemctl start nginx
# systemctl enable nginx
```

### 部署 KeepAlive

为了保证 nginx 的高可用，还需要部署 keepalive，keepalive 主要负责 nginx 的健康检查和故障转移。

① 分别在 k8s-lb-master 和 k8s-lb-backup 这两台机器上安装 keepalive

```shell
# yum install keepalived -y
```

② master 启动 keepalived

修改 k8s-lb-master keepalived 配置文件

```shell
cat > /etc/keepalived/keepalived.conf <<'EOF'
global_defs {
   notification_email {
     acassen@firewall.loc
     failover@firewall.loc
     sysadmin@firewall.loc
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id NGINX_MASTER
}

vrrp_script check_nginx {
   script "/etc/keepalived/check_nginx.sh" 
}

# vrrp实例
vrrp_instance VI_1 {
    state MASTER 
    interface ens33 
    virtual_router_id 51 
    priority 100
    advert_int 1 
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.31.100/24 
    }
    track_script {
        check_nginx
    }
}
EOF
```

**配置说明：**

- vrrp_script：用于健康检查nginx状态，如果nginx没有正常工作，就会进行故障漂移，使备节点接管VIP，这里通过 shell 脚本来检查nginx状态
- state：keepalived 角色，主节点为 MASTER，备节点为 BACKUP
- interface：接口，配置本地网卡名，keepalived 会将虚拟IP绑定到这个网卡上
- virtual_router_id：#VRRP 路由ID实例，每个实例是唯一的
- priority：优先级，备服务器设置90
- advert_int：指定VRRP心跳包通告间隔时间，默认1秒
- virtual_ipaddress：VIP，要与当前机器在同一网段，keepalived 会在网卡上附加这个IP，之后通过这个IP来访问Nginx，当nginx不可用时，会将此虚拟IP漂移到备节点上。

增加 check_nginx.sh 脚本，通过此脚本判断 nginx 是否正常：

```shell
cat > /etc/keepalived/check_nginx.sh <<'EOF'
#!/bin/bash
count=$(ps -ef | grep nginx | egrep -cv "grep|$$")
if [ "$count" -eq 0 ];then
    exit 1;
else 
    exit 0;
fi
EOF
```

增加可执行权限：

```shell
# chmod +x /etc/keepalived/check_nginx.sh
```

启动 keepalived：

```shell
# systemctl start keepalived
# systemctl enable keepalived
```

③ backup 启动 keepalived

修改 k8s-lb-backup keepalived 配置文件

```shell
cat > /etc/keepalived/keepalived.conf <<'EOF'
global_defs {
   notification_email {
     acassen@firewall.loc
     failover@firewall.loc
     sysadmin@firewall.loc
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id NGINX_BACKUP
}

vrrp_script check_nginx {
   script "/etc/keepalived/check_nginx.sh"
}

# vrrp实例
vrrp_instance VI_1 {
    state BACKUP
    interface eno16777736
    virtual_router_id 51
    priority 90
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.31.100/24
    }
    track_script {
        check_nginx
    }
}
EOF
```

增加 check_nginx.sh 脚本：

```shell
cat > /etc/keepalived/check_nginx.sh <<'EOF'
#!/bin/bash
count=$(ps -ef | grep nginx | egrep -cv "grep|$$")
if [ "$count" -eq 0 ];then
    exit 1;
else 
    exit 0;
fi
EOF
```

增加可执行权限：

```shell
# chmod +x /etc/keepalived/check_nginx.sh
```

启动 keepalived：

```shell
# systemctl start keepalived
# systemctl enable keepalived
```

④ 验证负载均衡

keepalived 已经将VIP附加到MASTER所在的网卡上

![img](https://img2018.cnblogs.com/blog/856154/202001/856154-20200105231349220-1660151152.png)

BACKUP节点上并没有

![img](https://img2018.cnblogs.com/blog/856154/202001/856154-20200105232120068-658127475.png)

关闭 k8s-lb-master 上的nginx，可看到VIP已经不在了

![img](https://img2018.cnblogs.com/blog/856154/202001/856154-20200105233741081-433366894.png)

可以看到已经漂移到备节点上了，如果再重启 MASTER 上的 Ngnix，VIP又会漂移到主节点上。

![img](https://img2018.cnblogs.com/blog/856154/202001/856154-20200105233815959-403369443.png)

访问虚拟IP还是可以访问的

![img](https://img2018.cnblogs.com/blog/856154/202001/856154-20200105233850964-1133520064.png)

4、Node节点连接VIP

① 修改 node 节点的配置文件中连接 k8s-master 的IP为VIP

![img](https://img2018.cnblogs.com/blog/856154/202001/856154-20200105234648106-1656311814.png)

② 重启 kubelet 和 kube-proxy

```shell
# systemctl restart kubelet
# systemctl restart kube-proxy
```

③ 在 node 节点上访问VIP调用 apiserver 验证

Authorization 的token 是前面生成 token.csv 中的令牌。

![img](https://img2018.cnblogs.com/blog/856154/202001/856154-20200106001143565-1533967509.png)

 至此，通过二进制方式安装 K8S 集群就算完成了！！