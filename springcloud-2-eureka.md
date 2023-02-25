[TOC]

# EUREKA

## 概述

Eureka是Netflix组件的一个子模块，也是核心模块之一。云端服务发现，一个基于 REST 的服务，用于定位服务，以实现云端中间层服务发现和故障转移（来源springcloud中文网的介绍：https://www.springcloud.cc/）。下图总结了Eureka服务端（以下简称服务端）与Eureka客户端（以下简称客户端）之间协同工作的流程：

![img](https://www.freesion.com/images/706/7baac0ff453a0f71a7168bc39fcd5a4a.png)

流程说明：

- Eureka客户端（以下简称客户端）启动后，定时向Eureka服务端（以下简称服务端）注册自己的服务信息（服务名、IP、端口等）；
- 客户端启动后，定时拉取服务端以保存的服务注册信息；
- 拉取服务端保存的服务注册信息后，就可调用消费其他服务提供者提供的服务。

虽然流程比较简单，但是在这样的简单流程下，Eureak究竟做了哪些工作，我们会有如下问题：

1. 客户端启动时如何注册到服务端？
2. 服务端如何保存客户端服务信息？
3. 客户端如何拉取服务端已保存的服务信息（是需要使用的时候再去拉取，还是先拉取保存本地，使用的时候直接从本地获取）？
4. 如何构建高可用的Eureka集群？
5. 心跳和服务剔除机制是什么？
6. Eureka自我保护机制是什么？

要彻底搞清楚Eureka的工作流程，必须需要弄清楚这些问题.

## 客户端启动时如何注册到服务端

**源码分析**

Eureka客户端在启动后，会创建一些定时任务，其中就有一个任务heartbeatExecutor就是就是处理心跳的线程池，部分源码（源码位置：com.netflix.discovery.DiscoveryClient）如下：

```java
heartbeatExecutor = new ThreadPoolExecutor(
        1, clientConfig.getHeartbeatExecutorThreadPoolSize(), 0, TimeUnit.SECONDS,
        new SynchronousQueue<Runnable>(),
        new ThreadFactoryBuilder()
                .setNameFormat("DiscoveryClient-HeartbeatExecutor-%d")
                .setDaemon(true)
                .build()
);  // use direct handoff
 
...此处省略其他代码
 
 //finally, init the schedule tasks (e.g. cluster resolvers, heartbeat, instanceInfo replicator, fetch
initScheduledTasks();
```

查看方法initScheduledTasks以及注释，可知该方法是初始化所有的任务（schedule tasks）。

```java
/**
 * Initializes all scheduled tasks.
 */
private void initScheduledTasks() {
  ...
  // Heartbeat timer
  scheduler.schedule(
        new TimedSupervisorTask(
                "heartbeat",
                scheduler,
                heartbeatExecutor,
                renewalIntervalInSecs,
                TimeUnit.SECONDS,
                expBackOffBound,
                new HeartbeatThread()
        ),
        renewalIntervalInSecs, TimeUnit.SECONDS);
   ...   
}
```

在上述方法中，第15行创建了一个线程HeartbeatThread，该线程就是处理心跳任务：

```java
/**
 * The heartbeat task that renews the lease in the given intervals.
 */
private class HeartbeatThread implements Runnable {
 
    public void run() {
        if (renew()) {
            lastSuccessfulHeartbeatTimestamp = System.currentTimeMillis();
        }
    }
}
 
/**
 * Renew with the eureka service by making the appropriate REST call
 */
boolean renew() {
    EurekaHttpResponse<InstanceInfo> httpResponse;
    try {
        httpResponse = eurekaTransport.registrationClient.sendHeartBeat(instanceInfo.getAppName(), instanceInfo.getId(), instanceInfo, null);
        logger.debug(PREFIX + "{} - Heartbeat status: {}", appPathIdentifier, httpResponse.getStatusCode());
        if (httpResponse.getStatusCode() == Status.NOT_FOUND.getStatusCode()) {
            REREGISTER_COUNTER.increment();
            logger.info(PREFIX + "{} - Re-registering apps/{}", appPathIdentifier, instanceInfo.getAppName());
            long timestamp = instanceInfo.setIsDirtyWithTime();
            boolean success = register();
            if (success) {
                instanceInfo.unsetIsDirty(timestamp);
            }
            return success;
        }
        return httpResponse.getStatusCode() == Status.OK.getStatusCode();
    } catch (Throwable e) {
        logger.error(PREFIX + "{} - was unable to send heartbeat!", appPathIdentifier, e);
        return false;
    }
}
```

在renew方法中，首先会发送一个心跳数据到服务端，服务端返回一个状态码，如果是NOT_FOUND（即404），表示Eureka服务端不存在该客户端的服务信息，那么就会向服务端发起注册请求（上面代码25行调用register方法）：

```java
boolean register() throws Throwable {
    logger.info(PREFIX + "{}: registering service...", appPathIdentifier);
    EurekaHttpResponse<Void> httpResponse;
    try {
        httpResponse = eurekaTransport.registrationClient.register(instanceInfo);
    } catch (Exception e) {
        logger.warn(PREFIX + "{} - registration failed {}", appPathIdentifier, e.getMessage(), e);
        throw e;
    }
    if (logger.isInfoEnabled()) {
        logger.info(PREFIX + "{} - registration status: {}", appPathIdentifier, httpResponse.getStatusCode());
    }
    return httpResponse.getStatusCode() == Status.NO_CONTENT.getStatusCode();
}
```

在register方法中，向服务端的注册信息instanceInfo，它是com.netflix.appinfo.InstanceInfo，包括服务名、ip、端口、唯一实例ID等信息。

> Eureka客户端在启动时，首先会创建一个心跳的定时任务，定时向服务端发送心跳信息，服务端会对客户端心跳做出响应，如果响应状态码为404时，表示服务端没有该客户端的服务信息，那么客户端则会向服务端发送注册请求，注册信息包括服务名、ip、端口、唯一实例ID等信息。

## 服务端如何保存客户端服务信息

**源码分析**

服务端注册源码（com.netflix.eureka.registry.PeerAwareInstanceRegistryImpl.class的方法register）如下：

```java
@Override
public void register(final InstanceInfo info, final boolean isReplication) {
    int leaseDuration = Lease.DEFAULT_DURATION_IN_SECS;
    if (info.getLeaseInfo() != null && info.getLeaseInfo().getDurationInSecs() > 0) {
        leaseDuration = info.getLeaseInfo().getDurationInSecs();
    }
    super.register(info, leaseDuration, isReplication);
    replicateToPeers(Action.Register, info.getAppName(), info.getId(), info, null, isReplication);
}
```

第7行调用了父类（com.netflix.eureka.registry.AbstractInstanceRegistry）register方法，源码如下

```java
public abstract class AbstractInstanceRegistry implements InstanceRegistry {
  ...
  private final ConcurrentHashMap<String, Map<String, Lease<InstanceInfo>>> registry
        = new ConcurrentHashMap<String, Map<String, Lease<InstanceInfo>>>();
  ...
  public void register(InstanceInfo registrant, int leaseDuration, boolean isReplication) {
    try {
        read.lock();
        Map<String, Lease<InstanceInfo>> gMap = registry.get(registrant.getAppName());
        REGISTER.increment(isReplication);
        if (gMap == null) {
            final ConcurrentHashMap<String, Lease<InstanceInfo>> gNewMap = new ConcurrentHashMap<String, Lease<InstanceInfo>>();
            gMap = registry.putIfAbsent(registrant.getAppName(), gNewMap);
            if (gMap == null) {
                gMap = gNewMap;
            }
        }
    ...
  }
}
```

在register方法中，我们可以看到将服务实例信息InstanceInfo注册到了register变量中，它其实就是一个ConcurrentHashMap。

> 客户端通过Jersey框架（亚马逊的一个http框架）将服务实例信息发送到服务端，服务端将客户端信息放在一个ConcurrentHashMap对象中。

## 客户端如何拉取服务端已保存的服务信息

在知道客户端是如何拉取服务端信息的同时，也需要清楚以下问题：

> *是需要时才去服务端拉取，还是先拉取到本地，需要用的时候直接从本地获取？*

**源码分析**

我们回到问题1的1.1节的initScheduledTasks方法中：

```java
private void initScheduledTasks() {
    if (clientConfig.shouldFetchRegistry()) {
        // registry cache refresh timer
        int registryFetchIntervalSeconds = clientConfig.getRegistryFetchIntervalSeconds();
        int expBackOffBound = clientConfig.getCacheRefreshExecutorExponentialBackOffBound();
        scheduler.schedule(
                new TimedSupervisorTask(
                        "cacheRefresh",
                        scheduler,
                        cacheRefreshExecutor,
                        registryFetchIntervalSeconds,
                        TimeUnit.SECONDS,
                        expBackOffBound,
                        new CacheRefreshThread()
                ),
                registryFetchIntervalSeconds, TimeUnit.SECONDS);
    }
    ...
}
```

> 上述代码中初始化了一个刷新缓存的定时任务，我们看到第14行的新建了一个线程CacheRefreshThread（源码不再列出），既是用来定时刷新服务端已保存的服务信息。

> 通过3.1节源码总结：客户端拉取服务端服务信息是通过一个定时任务定时拉取的，每次拉取后刷新本地已保存的信息，需要使用时直接从本地直接获取。

## 如何构建高可用的EUREKA集群

首先，搭建一个高可用的Eureka集群，只需要在每个注册中心（服务端）通过配置：

```java
eureka.client.service-url.defaultZone
```

指定其他服务端的地址，多个使用逗号隔开，如：

```yml
eureka.client.service-url.defaultZone=http://localhost:10000/eureka/,http://localhost:10001/eureka/,http://localhost:10002/eureka/
```

在eureka的高可用状态下，这些==注册中心是对等==的，他们会互相将注册在自己的实例同步给其他的注册中心，同样是通过问题1的方式将注册在自己上的实例注册到其他注册中心去。

那么问题来了，一旦 其中一个eureka收到一个客户端注册实例时，既然eureka注册中心将注册在自己的实例同步到其他注册中心中的方式和客户端注册的方式相同，那么在接收的eureka注册中心一端，会不会再同步回给注册中心（或者其他注册中心），从而导致死循环。

**源码解析**

我们回到2.1节的PeerAwareInstanceRegistryImpl类的register方法，在该方法中的最后一行：

```java
replicateToPeers(Action.Register, info.getAppName(), info.getId(), info, null, isReplication);
```

replicateToPeers方法字面意思是同步或者复制到同事（即其他对等的注册中心），最后一个参数为isReplication，是一个boolean值，表示是否同步（复制），如果是客户端注册的，那么为false,如果是其他注册中心同步的则为true，replicateToPeers方法中，如果isReplication=false时，将会发起同步（第19行）：

```java
private void replicateToPeers(Action action, String appName, String id,
                              InstanceInfo info /* optional */,
                              InstanceStatus newStatus /* optional */, boolean isReplication) {
    Stopwatch tracer = action.getTimer().start();
    try {
        if (isReplication) {
            numberOfReplicationsLastMin.increment();
        }
        // If it is a replication already, do not replicate again as this will create a poison replication
        if (peerEurekaNodes == Collections.EMPTY_LIST || isReplication) {
            return;
        }
 
        for (final PeerEurekaNode node : peerEurekaNodes.getPeerEurekaNodes()) {
            // If the url represents this host, do not replicate to yourself.
            if (peerEurekaNodes.isThisMyUrl(node.getServiceUrl())) {
                continue;
            }
            replicateInstanceActionsToPeers(action, appName, id, info, newStatus, node);
        }
    } finally {
        tracer.stop();
    }
}
```

> - 搭建高可用的Eureka集群，只需要在注册中心的配置文件中配置其他注册中心的地址：
>
> - 注册中心收到注册信息后会判断是否是其他注册中心同步的信息还是客户端注册的信息，如果是客户端注册的信息，那么他将会将该客户端信息同步到其他注册中心去；否则收到信息后不作任何操作。通过此机制避免集群中信息同步的死循环。

## 心跳和服务剔除机制是什么

**源码分析**

在eureka源码中，有个evict（剔除，驱逐，源码位置：com.netflix.eureka.registry.AbstractInstanceRegistry，代码清单5.1）的方法：

```java
public void evict(long additionalLeaseMs) {
    logger.debug("Running the evict task");
 
    if (!isLeaseExpirationEnabled()) {
        logger.debug("DS: lease expiration is currently disabled.");
        return;
    }
 
    // We collect first all expired items, to evict them in random order. For large eviction sets,
    // if we do not that, we might wipe out whole apps before self preservation kicks in. By randomizing it,
    // the impact should be evenly distributed across all applications.
    List<Lease<InstanceInfo>> expiredLeases = new ArrayList<>();
    for (Entry<String, Map<String, Lease<InstanceInfo>>> groupEntry : registry.entrySet()) {
        Map<String, Lease<InstanceInfo>> leaseMap = groupEntry.getValue();
        if (leaseMap != null) {
            for (Entry<String, Lease<InstanceInfo>> leaseEntry : leaseMap.entrySet()) {
                Lease<InstanceInfo> lease = leaseEntry.getValue();
                if (lease.isExpired(additionalLeaseMs) && lease.getHolder() != null) {
                    expiredLeases.add(lease);
                }
            }
        }
    }
 
    // To compensate for GC pauses or drifting local time, we need to use current registry size as a base for
    // triggering self-preservation. Without that we would wipe out full registry.
    int registrySize = (int) getLocalRegistrySize();
    int registrySizeThreshold = (int) (registrySize * serverConfig.getRenewalPercentThreshold());
    int evictionLimit = registrySize - registrySizeThreshold;
 
    int toEvict = Math.min(expiredLeases.size(), evictionLimit);
    if (toEvict > 0) {
        logger.info("Evicting {} items (expired={}, evictionLimit={})", toEvict, expiredLeases.size(), evictionLimit);
 
        Random random = new Random(System.currentTimeMillis());
        for (int i = 0; i < toEvict; i++) {
            // Pick a random item (Knuth shuffle algorithm)
            int next = i + random.nextInt(expiredLeases.size() - i);
            Collections.swap(expiredLeases, i, next);
            Lease<InstanceInfo> lease = expiredLeases.get(i);
 
            String appName = lease.getHolder().getAppName();
            String id = lease.getHolder().getId();
            EXPIRED.increment();
            logger.warn("DS: Registry: expired lease for {}/{}", appName, id);
            internalCancel(appName, id, false);
        }
    }
}
```

在上述代码第4行，做了isLeaseExpirationEnabled（字面意思：是否启用租约到期，即是否开启了服务过期超时机制，开启之后就会将过期的服务进行剔除）的if判断，源码（com.netflix.eureka.registry

.PeerAwareInstanceRegistryImpl实现类中，代码清单5.2）如下：

```java
@Override
public boolean isLeaseExpirationEnabled() {
    if (!isSelfPreservationModeEnabled()) {
        // The self preservation mode is disabled, hence allowing the instances to expire.
        return true;
    }
    return numberOfRenewsPerMinThreshold > 0 && getNumOfRenewsInLastMin() > numberOfRenewsPerMinThreshold;
}
```

同样在上述方法开始的第3行也做了isSelfPreservationModeEnabled方法的判断，该方法是判断是否开启了自我保护机制（有关自我保护机制有关说明在第6节），接下来看到第4行的注释翻译如下：

> *自保存模式被禁用，因此允许实例过期*

也就是说如果关闭了自我保护机制，那么直接就允许实例过期，也就是说可以将过期的服务实例剔除。那如果开启了自我保护机制，会做如下判断（代码清单5.3）：

```java
numberOfRenewsPerMinThreshold > 0 && getNumOfRenewsInLastMin() > numberOfRenewsPerMinThreshold
```

getNumOfRenewsInLastMin即最后一分钟接收到的心跳总数，numberOfRenewsPerMinThreshold 表示收到一分钟内收到服务心跳数临界值（后简称临界值），也就是说当临界值大于0，且最后一分钟接收到的心跳总数大于临界值时，允许实例过期，他的计算方式源码如下（代码清单5.4）：

```java
protected void updateRenewsPerMinThreshold() {
    this.numberOfRenewsPerMinThreshold = (int) (this.expectedNumberOfClientsSendingRenews
            * (60.0 / serverConfig.getExpectedClientRenewalIntervalSeconds())
            * serverConfig.getRenewalPercentThreshold());
}
```

其中：

- this.expectedNumberOfClientsSendingRenews：接收到的客户端数量
- serverConfig.getExpectedClientRenewalIntervalSeconds()：客户端发送心跳时间的间隔，默认是30秒
- serverConfig.getRenewalPercentThreshold()：一个百分比率阈值，默认是0.85，可以通过配置修改

从上述代码的计算方法可以看出：

> *分钟内收到服务心跳数临界值 = 客户端数量 \* （60/心跳时间间隔） \* 比率*

带入默认值：

```
一分钟内收到服务心跳数临界值 = 客户端数量 * （60/30） * 0.85
                          = 客户端数量 * 1.7
```

所以假如有总共有10个客户端，那么表示一分钟至少需要收到17次心跳。

所以代码清单5.3的解析就是，如果开启只我保护机制，那么一分钟内收到的心跳数大于一分钟内收到服务心跳数临界值时，则启用租约到期机制，即服务剔除机制。

那么最终回到代码清单5.1的第4行的if判断，即如果没有启用服务剔除机制（即开启了自我保护机制或者一分钟收到的心跳数小于临界值），那么直接return结束，不做任何操作。否则代码继续运行，从代码的第9行注释到最后，可以看出先跳出已过期的服务实例，然后通过随机数的方式将这些已过期的实例进行剔除。

> 心跳机制：
>
> - 客户端启动后，就会启动一个定时任务，定时向服务端发送心跳数据，告知服务端自己还活着，默认的心跳时间间隔是30秒。
>
> 服务剔除机制：
>
> - 如果开启了自我保护机制，那么所有的服务，包括长时间没有收到心跳的服务（即已过期的服务）都不会被剔除；
> - 如果未开启自我保护机制，那么将判断最后一分钟收到的心跳数与一分钟收到心跳数临界值（计算方法参考5.1节）比较，如果前者大于后者，且后者大于0的话，则启用服务剔除机制；
> - 一旦服务剔除机制开启，则Eureka服务端并不会直接剔除所有已过期的服务，而是通过随机数的方式进行剔除，避免自我保护开启之前将所有的服务（包括正常的服务）给剔除。

## EUREKA自我保护机制是什么

由于在第5节中已经提到了有关Eureka自我保护机制的用途以及它在服务剔除机制中起到的作用，这里不再结合源码分析，这里分析Eureka为什么要采用自我保护机制。

在分布式系统的CAP理论中，Eureka采用的AP，也就是Eureak保证了服务的可用性（A），而舍弃了数据的一致性（C）。当网络发生分区时，客户端和服务端的通讯将会终止，那么服务端在一定的时间内将收不到大部分的客户端的一个心跳，如果这个时候将这些收不到心跳的服务剔除，那可能会将可用的客户端剔除了，这就不符合AP理论。