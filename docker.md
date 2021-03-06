# 服务容器&集群

### 容器
容器是一组运行在Linux操作系统上并使用命名空间进程进行分隔的进程，有了容器就无需再启动和维护虚拟机。与虚拟机技术相比，容器的最大不同之处在于打包格式和可移植性。构建容器的目的在于为现代基础设施降低占用空间和启动时间、提供重用性、更好地利用服务器资源，并更好地集成到整个开发生态系统中（例如持续集成和交付生命周期）。更多的细节可参见“微服务、容器及云原生架构”一文。

*Docker的吸引力来自于两个方面：快速与可移植性*

- 快速

  普通的虚拟机在每次开机时都需要启动一个完整的新操作系统实例，而Docker的容器能够通过内核共享的方式，共享一套托管操作系统。这意味着，Docker容器的启动和停止不需要几分钟，只要几百毫秒就足够了。
  
  更快的速度就意味着，使用Docker容器创建的软件系统比起使用基于虚拟机的解决方案能够实现更高级别的敏捷性，即使将那些基于虚拟机的解决方案通过基于微服务的架构进行组织也是一样。此外，“容器化”的应用比起虚拟机和裸机的性能更好，在2014年IBM发布的一份研究报告中表明：“在几乎所有情况下，容器都能表现出与虚拟机相等、或者是更好的性能。”

- 可移植性

  在基于虚拟机的解决方案中，应用的可移植性通常来说会受到云提供商所支持的地区的限制，如果是在自托管的环境中运行企业软件，那么可移植性就限制在数据中心内。原因在于，不同的云提供商通常会提供不同格式的虚拟机。如果使用Packer这样的工具，那么在不同的云服务中使用运行相同的虚拟机镜像文件也是可以的，但需要进行许多额外的工作。虽然可行，但它也将用户限制在一个单一的平台中。下文将进一步分析这一点的问题所在。
  
  Docker容器的设计理念是“一次编写，到处运行”，这可以使开发者避免上面这种限制。工程团队与运维团队可以将他们的基础设施扩展到多个云提供商的服务中，只要Docker的守护进程还在运行，就能够保证应用程序的正常运行。这种将应用从云提供商中进行解耦的方式能够给予IT团队更大的自由度，也可以在与各个提供商之间的对冲中，提高软件解决方案的适应性。

#### 无状态的应用程序设计
微服务架构的创建者倾向于在任何可能的情况下使用无状态的服务、而不是有状态的服务。无状态应用程序设计的主要优点在于：**它能够平稳地应对为服务添加或移除某些实例的场景，而无需对应用程序进行重大的变更或进行配置的改动**。比方说，如果服务的负载产生了突发性的增长，可以为服务加入更多无状态的web服务器，而如果某个无状态的服务器挂机了，也可以方便地用另外一台服务器取代它。因此，无状态的服务更容易实现敏捷性和适应性。

### Docker Swarm具有如下基本特性：
- 集群管理集成进Docker Engine
使用内置的集群管理功能，我们可以直接通过Docker CLI命令来创建Swarm集群，然后去部署应用服务，而不再需要其它外部的软件来创建和管理一个Swarm集群。
- 去中心化设计
Swarm集群中包含Manager和Worker两类Node，我们可以直接基于Docker Engine来部署任何类型的Node。而且，在Swarm集群运行期间，我们既可以对其作出任何改变，实现对集群的扩容和缩容等，如添加Manager Node，如删除Worker Node，而做这些操作不需要暂停或重启当前的Swarm集群服务。
- 声明式服务模型（Declarative Service Model）
在我们实现的应用栈中，Docker Engine使用了一种声明的方式，让我们可以定义我们所期望的各种服务的状态，例如，我们创建了一个应用服务栈：一个Web前端服务、一个后端数据库服务、Web前端服务又依赖于一个消息队列服务。
- 服务扩容缩容
对于我们部署的每一个应用服务，我们可以通过命令行的方式，设置启动多少个Docker容器去运行它。已经部署完成的应用，如果有扩容或缩容的需求，只需要通过命令行指定需要几个Docker容器即可，Swarm集群运行时便能自动地、灵活地进行调整。
- 协调预期状态与实际状态的一致性
Swarm集群Manager Node会不断地监控集群的状态，协调集群状态使得我们预期状态和实际状态保持一致。例如我们启动了一个应用服务，指定服务副本为10，则会启动10个Docker容器去运行，如果某个Worker Node上面运行的2个Docker容器挂掉了，则Swarm Manager会选择集群中其它可用的Worker Node，并创建2个服务副本，使实际运行的Docker容器数仍然保持与预期的10个一致。
- 多主机网络
我们可以为待部署应用服务指定一个Overlay网络，当应用服务初始化或者进行更新时，Swarm Manager在给定的Overlay网络中为Docker容器自动地分配IP地址，实际是一个虚拟IP地址（VIP）。
- 服务发现
Swarm Manager会给集群中每一个服务分配一个唯一的DNS名称，对运行中的Docker容器进行负载均衡。我们可以通过Swarm内置的DNS Server，查询Swarm集群中运行的Docker容器状态。
- 负载均衡
在Swarm内部，可以指定如何在各个Node之间分发服务容器（Service Container），实现负载均衡。如果想要使用Swarm集群外部的负载均衡器，可以将服务容器的端口暴露到外部。
- 安全策略
在Swarm集群内部的Node，强制使用基于TLS的双向认证，并且在单个Node上以及在集群中的Node之间，都进行安全的加密通信。我们可以选择使用自签名的根证书，或者使用自定义的根CA（Root CA）证书。
- 滚动更新（Rolling Update）
对于服务需要更新的场景，我们可以在多个Node上进行增量部署更新，Swarm Manager支持通过使用Docker CLI设置一个delay时间间隔，实现多个服务在多个Node上依次进行部署。这样可以非常灵活地控制，如果有一个服务更新失败，则暂停后面的更新操作，重新回滚到更新之前的版本。






  
  



