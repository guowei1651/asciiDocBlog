[DevOps实践](https://www.jianshu.com/c/f8fa98feb686)系列文章，请参见连接。

## 概述：
现阶段服务运行环境有很多种方式，可以从不同的角度划分。比较原始的有直接在硬件的操作系统上运行，稍高级一点的需要在硬件系统上虚拟化操作系统再运行系统服务。还有更高级的是在操作系统上做系统隔离然后再运行，在向上可以使用无服务器架构运行服务。

这里介绍使用Swarm进行服务运行环境搭建，以及使用过程中的一些注意事项。下图为Swarm帮我们解决的问题。
![Swarm示意图](https://upload-images.jianshu.io/upload_images/2454595-db2559324f156428.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 技术选型：
业界在开发SaaS化服务时，有很多经验可以借鉴。阿里的大中台服务、HeroKu的十二因子、OpenStack的服务群组、NetFlix的地域IDC、微服务的设计规范等等，这些帮我们在不同的层面上制定相关的要求与解决方案。
在这里大概的总结一下，一个网络服务产品服务在运行环境上的一些大概要求。
|编号|要求|说明|
|:-:|-|-|
|1|所有地方统一的运行环境|1. 生产环境和开发人员的开发环境时一样的<br>2. 生产环境和测试环境时一样的<br>3. 最小运行环境必须可以在开发人员环境上启动并运行|
|2|可以针对每一个服务进行资源限制|1. 可以根据硬件的情况进行服务的资源的划分<br>2. 可以根据服务的负载情况限制服务的资源|
|3|方便的运行态管理|1. 支持日志<br>2. 支持运行时数据获取<br>3. 动态的配置数据下发<br>4. 快速启动与优雅关闭功能需要支持 |
|4|简单的部署，升级管理|1. 需要支持蓝绿发布<br>2. 需要可以根据部署情况，切换流量到不同的服务商|
|5|可以适应大范围的扩缩容|1. 可以根据服务的负载情况进行动态的伸缩工作<br>2. 最好能够自动的伸，手动的缩|
|6|方便的运行版本，回滚管理|1. 部署过程中或部署完成后发现版本问题，可以方便的回滚<br>2. 有自己的发行版管理方法，可以选择确定版本运行|
|7|最好支持服务发现|1. 各种形式的服务发现工作|

运行环境的要求的策略是尽量少的操作就可以完成服务的监控与上线操作，并且是越少越好。所以，下面的技术选择过程中也是会按照这个大原则并根据上面的要求进行技术的选择。

运行环境技术近些年发展出来很多种。所以，这里简单的说明一下各种技术的对比。
![各种运行环境示意](https://upload-images.jianshu.io/upload_images/2454595-a60af4ec87eee8f6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

使用IaaS系统搭建服务需要进行中间件与运行时，数据持久化的控制。用PaaS做会方便很多用，只需要开发上层应用即可，并对上层应用的部署进行管理。用FaaS的话只需要写脚本，讲脚本上传后就可以运行在任何地方。FaaS把底层的所有内容都屏蔽掉了，不需要进行相关的调试。

从某种角度来看FaaS是最好的选择，它可以满足所有的设计原则。可以满足无状态服务，前后端分离，Restful接口，弹性伸缩，异步任务，延时任务，自动恢复等等特点。不过现阶段，没有成熟的IDE可以支持FaaS的开发，所以从某种角度限制了FaaS的发展。这里就可以排除FaaS的技术选择。

这里就剩下IaaS和PaaS的选择，选择这两个可以使用两条原则：1. 如需要最大程度的免除运维操作，可以使用PaaS，2. 比较可控性与费用。如果使用IaaS自行搭建服务比PaaS的可控性和费用要好，那就可以选择IaaS服务。

在不同的项目或产品上对服务的关注点不一样，所以这里需要针对不同的项目进行分析。我这里选择的是IaaS和PaaS中间的一档，我自己搭建一些PaaS提供的服务，使用另外一些PaaS提供的服务。这样组合形成我的服务运行环境。

本文中会使用Swarm搭建的PaaS运行环境。用Docker的环境隔离能力和Swarm的服务调度能力完成运行环境搭建。

## 环境搭建：
搭建过程非常简单，可以参照[Swarm Mode](https://docs.docker.com/glossary/?term=swarm%20mode)官方文档进行搭建。

因为这里使用的是IaaS云进行环境搭建，所以，我不指望Swarm帮我去管理整个IDC。如果需要使用容器化技术管理整个IDC请参考CNCF或者Mesos的DC/OS的支持。

### Swarm概念介绍：
**网络端口：**

1. Swarm使用TCP port 2377作为集群管理通信使用。
2. TCP/UDP 7946端口用于节点间通信的。
3. UDP 4789端口用于Ingress。

在搭建环境时需要将这几个端口从防火墙上放开。

**Node:**
一个Docker Daemon进程。可以加入到Swarm集群中，可以接受Swarm任务调度的一个节点。

**Service:**
一个服务，一个服务可以有多份部署，多份运行的进程。可以指定服务的运行副本数，指定一个服务运行多少个副本，运行在哪个节点。

**Stack:**
一群服务可以组成一个栈。提供完整的一套服务。Swarm Stack和Kubernetes的配置文件的格式都是使用yml文件。这是近些年来业界比较流行的一切即代码的一个实践。这里就是[基础设施即代码](http://www.ituring.com.cn/article/179897)，形成代码后就可以对环境进行版本化的管理。

**Task:**
一个运行中的容器。

从上可以看出Swarm的概念很简单，使用过Docker的人很容易上手。不像Kubernetes一上来就整了很多概念，安装过程又复杂。从这里可以看出Swarm的血刺成本比Kubernetes低很多，所以，可以使用Swarm作为上手云基础设施搭建的练手过程，然后再去搭建Kubernetes集群会容易很多。

## 注意事项：
### 1. 服务集群
Swarm在做服务部署和服务管理时，基本上都是对无状态服务的支持。可以使用volume做有状态服务的状态持久化动作，但是没有对分布式文件系统进行支持。所以，在Swarm上最好的做法是指把无状态服务添加到Swarm进行管理。

在Swarm中对服务调度策略的开放度不是很高，现在Swarm只支持两种[调度策略](https://docs.docker.com/compose/compose-file/#mode)：1.Replicated；2. Global。Replicated是指定副本数，由Swarm进行那个节点运行服务的调度。Global表示在Swarm中所有节点上都运行一个服务。

上面说到服务都是由Swarm进行调度的，所以，这里必须由调度负责帮助进行服务的扩缩容操作，自动恢复操作。在Swarm中可以使用docker stack deploy进行扩缩容。自动恢复也是配合调度策略的，在一个Node上如果服务连续的出现几次宕机之后，服务可能被Swarm调度到另外的Node上。

在Swarm没有完成的版本升级与蓝绿部署，流量切换策略等。所以，需要自行通过Swarm提供的工具进行升级的整体工作。

### 2. 负载均衡
在Swarm中提供了集群方式发布应用，那它是怎么做负载均衡的？

负载均衡的服务可能需要部署在不同的Node上，所以，Swarm扩展了原来Docker的网络，创建了Overlay网络。Overlay网络可以进行多物理主机间的Docker和容器的通信。

使用Overlay满足了多主机通信后，使用内嵌的DNS服务进行服务的发现操作。这样就可以将多台主机连接到一个Swarm集群中，并保证能够进行通信与交互。

Swarm使用内嵌的Ingress进行负载均衡的支持。

我使用了一段时间的Swarm网络和Ingress之后发现一个问题。在服务重新部署或新加入Docker Overlay网络的时原先的服务没有办法路由到新的服务上，这个具体原因不太清楚。我怀疑是Swarm Overlay和内嵌DNS的更新过程没有做完整，所以，这里才会发生老服务ping新服务不通的问题。

因为Swarm内嵌的Ingress没有暴露出配置接口，所以对Swarm的负载均衡方式的可控性不是很高。

如果进行四层的负载均衡，Ingress的处理能力具体如何不太清晰。在进行7层负载均衡时的策略与控制方法也没有看到。所以，从可控制角度看Swarm内嵌的Ingress不太可用。

### 3. 持续交付
减轻了环境管理在持续交付过程中的工作量。原先在不同的服务器上运行，并监控的服务。现在都交由Swarm完成。

### 4. 运行状态
可以看到服务宕机状态，其他状态只能从服务中暴露出接口进行管理。

## 总结:
现阶段的Swarm在很多反面适应做小系统的控制。在很多方面的可控性和可用性提高后，应该是一个不错的服务管理工具。

在这里留几个问题：
1. 如何进行有状态服务的Swarm部署？
2. 怎么进行服务的自动动态增加部署？
3. 蓝绿发布怎么进行流量的切换，与服务启动？升级过程中的流量切换也有一样问题？
4. 服务的状态监控怎么控制服务假死情况？
5. 在使用Dubbo进行服务部署时，需要进行特殊的集群配置吗？（dubbo协议，不是Restful协议）
6. 十二因子中的日志事件流的方式的支持？（可以使用网络日志方式）
7. 配置发现方式在Swarm中的处理方式？

接下来的文章中介绍DevOps上遇到这些问题应该怎样解决？

## 参考：
[Docker Reference Architecture: Running Docker Enterprise Edition at Scale](https://success.docker.com/article/running-docker-ee-at-scale)
[Docker Reference Architecture: Docker Logging Design and Best Practices](https://success.docker.com/article/logging-best-practices)
[Docker Reference Architecture: Universal Control Plane 3.0 Service Discovery and Load Balancing](https://success.docker.com/article/ucp-service-discovery)
[Docker Reference Architecture: Designing Scalable, Portable Docker Container Networks](https://success.docker.com/article/networking#swarmnativeservicediscovery)
[Docker Swarm和Kubernetes在大规模集群中的性能比较](http://www.dockone.io/article/1145)
[【Docker】 Swarm简单介绍](https://www.cnblogs.com/franknihao/p/8490416.html)
[Stack Overflow：我们是如何做监控的](https://mp.weixin.qq.com/s/iiO1EiHXWmOAbQSYqxLQ3A)
