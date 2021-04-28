# Kubernetes-Concepts and principles

# Kubernetes 核心概念

## 简介

Container 除了能够表示 “容器” 之外，还有 “集装箱” 的意思；而 Kubernetes 对应的中文翻译是 “舵手” 或者 “飞行员”，所以很容易领会到 Kubernetes 命名的寓意：运送集装箱的一个轮船，来帮助我们管理这些集装箱

更具体地来说：Kubernetes 是一个自动化的容器编排平台，它负责应用的部署、应用的弹性以及应用的管理

## 功能

Kubernetes 有以下几个核心功能

- 服务发现与负载均衡
- 调度，容器的自动装箱 Scheduling

    把一个容器放到一个集群的某一个 Node 上，Kubernetes 会帮助我们去做存储的编排，让存储的声明周期与容器的生命周期能有一个连接

- 容器的自动恢复

    在一个集群中，经常会出现宿主机的问题或者说是 OS 的问题，导致容器本身的不可用，Kubernetes 会自动地对这些不可用的容器进行恢复

- 应用的自动发布与应用的回滚
- 水平伸缩

    Kubernetes 通过业务负载检查的能力，监测业务上所承担的负载，对业务进行扩容或缩容

## 架构

Kubernetes 是一个比较典型的二层架构和 server-client 架构，

由负责管理和与用户交互的 Master，和负责实际运行业务的 Node 组成

Kubernetes 的架构示意图如下

![./img/Untitled.png](./img/Untitled%2000.png)

### Master

Master 作为中央的管控节点，会去与实际运行业务负载的 Node 进行连接

所有 UI 、clients 等 user 侧的组件，只会和 Master 进行连接，把希望的状态或者想执行的命令下发给 Master，Master 会把这些命令或者状态下发给相应的节点，进行最终的执行

Kubernetes 的 Master 包含四个主要的组件：API Server、Controller、Scheduler 以及 etcd

![./img/Untitled%201.png](./img/Untitled%2001.png)

- API Server

    处理 API 操作，Kubernetes 中所有的组件都会和 API Server 进行连接

    组件与组件之间一般不进行独立的连接，都依赖于 API Server 进行消息的传送

- Controller

    控制器，用来完成对集群状态的一些管理

    比如，自动对容器进行修复、自动进行水平扩张

- Scheduler

    调度器，用于完成调度的操作

    比如，把一个用户提交的 Container，依据它对 CPU、对 memory 请求大小，找一台合适的节点，进行放置

- etcd

    一个分布式的一个存储系统，API Server 中所需要的这些原信息都被放置在 etcd 中

    etcd 本身是一个高可用系统，通过 etcd 保证整个 Kubernetes 的 Master 组件的高可用性

### Node

Node 是真正运行业务负载的，每个业务负载会以 Pod 的形式运行

Kubernetes 的 Node 并不会直接和 user 进行 interaction，它的 interaction 只会通过 Master

而 User 是通过 Master 向节点下发这些信息的，Kubernetes 每个 Node 上都会运行以下四个组件

![./img/Untitled%2002.png](./img/Untitled%2002.png)

- Kubelet

    一个 Pod 中运行的一个或者多个容器，而真正去运行这些 Pod 的组件的是叫做 kubelet

    它通过 API Server 接收到所需要 Pod 运行的状态，然后提交到 Container Runtime 中

- Storage Pulgin / Network Plugin

    Kubernetes 并不会直接进行网络存储的操作，他们会靠 Storage Plugin 或者是 Network Plugin 来进行操作

    用户或云厂商都会去写相应的 Storage Plugin ****或者 Network Plugin，去完成存储操作或网络操作

- Kube-proxy

    在 Kubernetes 环境中，真正完成 service 组网的组件的是 Kube-proxy

    它是利用了 iptable 的能力来进行组建 Kubernetes 的 Network，就是 cluster network

## 组件

### API Server

kube-apiserver 是 Kubernetes 最重要的核心组件之一，提供了 Kubernetes 各类资源对象的增删改查及 watch 等 HTTP Rest 接口，是整个系统的数据总线和数据中心，主要提供以下的功能

- 提供集群管理的 REST API 接口，包括认证授权、数据校验以及集群状态变更等
- 提供其他模块之间的数据交互和通信的枢纽（其他模块通过 API Server 查询或修改数据，只有 API Server 才直接操作 etcd）

API Server作为整个Kubernetes集群的核心组件，让所有资源可被描述和配置；这里的资源包括了类似网络、存储、Pod这样的基础资源也包括了replication controller、deployment这样的管理对象

API Server某种程度上来说更像是包含了一定逻辑的对象数据库；接口上更加丰富、自带GC、支持对象间的复杂逻辑；当然API Server本身是无状态的,数据都是存储在etcd当中

API Server提供集群管理的REST API接口，支持增删改查和patch、监听的操作，其他组件通过和API Server的接口获取资源配置和状态，以实现各种资源处理逻辑

![./img/Untitled%203.png](./img/Untitled%2003.png)

- Scheme：定义了资源序列化和反序列化的方法以及资源类型和版本的对应关系；这里我们可以理解成一张映射表
- Storage：是对资源的完整封装，实现了资源创建、删除、watch等所有操作
- APIGroupInfo：是在同一个Group下的所有资源的集合

### Kubelet

每个Node节点上都运行一个 Kubelet 服务进程，默认监听 10250 端口，接收并执行 Master 发来的指令，管理 Pod 及 Pod 中的容器

每个 Kubelet 进程会在 API Server 上注册所在Node节点的信息，定期向 Master 节点汇报该节点的资源使用情况，并通过 cAdvisor 监控节点和容器的资源

- Kubelet 主要负责同容器运行时（比如 Docker）打交道，而这个交互所依赖的，是一个称作 CRI（Container Runtime Interface）的远程调用接口，这个接口定义了容器运行时的各项核心操作，比如：启动一个容器需要的所有参数

    而具体的容器运行时，则一般通过 OCI 这个容器运行时规范同底层的 Linux 操作系统进行交互，即：把 CRI 请求翻译成对 Linux 操作系统的调用（操作 Linux Namespace 和 Cgroups 等）

    此外，Kubelet 还通过 gRPC 协议同一个叫作 Device Plugin 的插件进行交互，这个插件是Kubernetes 项目用来管理 GPU 等宿主机物理设备的主要组件，也是基于 Kubernetes 项目进行机器学习训练、高性能作业支持等工作必须关注的功能

- 而kubelet 的另一个重要功能，则是调用网络插件和存储插件为容器配置网络和持久化存储

    这两个插件与 Kubelet  进行交互的接口，分别是 CNI（Container Networking Interface）和CSI（Container Storage Interface）

#### Kubelet 创建容器

Kubelet 创建容器有以下几步

1. 请求创建容器，kubelet → docker-shim

    Kubelet 通过容器运行时容器接口 CRI 调用 docker-shim 请求创建容器

2. 请求转发，docker-shim → docker daemon

    docker-shim 将接受的请求转发到 docker daemon 请求创建容器

3. 请求转发，docker daemon → containerd

    docker daemon 请求 containerd 创建容器

4. 创建容器管理进程，containerd → containerd-shim

    containerd 收到请求之后，创建容器的管理进程 containerd-shim 用于收集状态、维持 stdin 等工作

5. 启动容器，containerd-shim → runC

    containerd-shim 调用 runC 创建容器，runC 在启动完成后退出

    containerd-shim 成为容器进程的父进程, 负责收集容器进程的状态, 上报给 containerd，并在容器中 pid 为 1 的进程退出后接管容器中的子进程进行清理

---

# Kubernetes 设计

## List-Watch

在 Kubernetes 中只有 API Server 会与 etcd 直接交互，其他组件（kubelet、kube-controller-manager、kube-scheduler）获取资源信息都需要通过 API Server

其实 API Server 和组件之间是一种 发布-订阅者模式，当组件需要监控资源时会向 API Server 发送 Watch 请求，并且这个请求是可以带过滤条件的；即 API Server 根据条件过滤数据，只在组件需要部分的数据变更时，发送给组件

List-Watch 由两部分组成，分别是 List 和 Watch

- List

    调用资源的 list API 罗列资源，获取全量数据，基于HTTP短链接实现

    List  查询当前的资源及其对应的状态（即期望的状态），客户端通过拿期望的状态和实际的状态进行对比，纠正状态不一致的资源（用于辅助修正 Watch 的数据）

- Watch：

    调用资源的watch API监听资源变更事件，获取增量数据，基于HTTP 长链接实现

    Watch 和 API Server 保持一个长链接，接收资源的状态变更事件并做相应处理

List  和 Watch 一起保证了消息的可靠性，避免因消息丢失而造成状态不一致场景，如果仅调用Watch 若某个时间点连接中断，就有可能导致消息丢失，所以需要通过 List 解决消息丢失问题

虽然仅仅通过轮询 List，也能达到同步资源状态的效果，但是轮询的存在 开销大，实时性不足的问题

## 关于 Pod

Pod 是可以在 Kubernetes 中创建和管理的、最小的可部署的计算单元

Pod 中封装着应用的容器（有的情况下是好几个容器），存储、独立的网络 IP，管理容器如何运行的策略选项，Pod 代表着部署的一个单位：kubernetes 中应用的一个实例，可能由一个或者多个容器组合在一起共享资源

### 单进程模型

容器的单进程模型，是指容器没有管理多个进程的能力，并不是指容器里只能运行一个进程

因为容器中 pid 为 1 的进程对应应用本身，这个进程就代表着容器的状态

如果在一个已经启动的容器中再去运行另外一个进程，那么当这个进程意外退出时是不会对容器状态产生影响的，我们是无法直接观测到这个进程意外退出了的；并且这个进程意外退出后的垃圾收集或者后续的异常处理是没有人来做的

### 为什么需要 Pod

在 Kubernetes 里面 pod 对应的是 Linux 中进程组的概念

pod 这个概念主要用于处理容器之间的超亲密关系

这里的超亲密关系大概存在以下几种情况

- 文件交换

    两个进程之间会对同一个文件进行操作，比如，一个写日志，一个读日志

- 本地通信

    进程之间需要通过 localhost，或者说是本地的 Socket 进行通信

- 频繁的 RPC 调用

    两个容器或者是微服务之间，需要发生非常频繁的 RPC 调用，出于性能的考虑，也希望它们是超亲密关系

- 共享 Namespace

    最简单常见的一个例子，就是我有一个容器需要加入另一个容器的 Network Namespace，这样我就能看到另一个容器的网络设备，和它的网络信息

可以简单理解为，一个应用存在不同的进程，受限于容器的 **单进程模型**，需要每个进程有独立的容器，而这些不同的进程因为调用或者性能考虑，要求他们”运行在一起“，于是产生了 pod 的概念，即一个 pod 对应一个完整的应用

### pod 的实现原理

pod 是一个逻辑的概念，是一组共享资源（network、volume ...）的容器集合

pod 通过中间容器 Infra 实现资源的共享，Infra 永远是 pod 中第一个被创建的容器，用户定义的其他容器通过 join namespace 的方式和 Infra 关联在一起

![./img/Untitled%2004.png](./img/Untitled%2004.png)

Infra 占用极少的资源（100~200 KB 左右），它使用由汇编语言编写的特殊的镜像 `k8s.gcr.io/pause` ，并且永远处于暂停状态

当 Infra 创建了 network namespace 后，用户容器便可以加入，所以对于 pod 中的用户容器来说：它们之间可以使用 [localhost](http://localhost) 通信；可以看到和 Infra 相同的网络设备，共享所有网络资源

所以，一个 pod 只对应一个 IP 地址（该 network namespace 对应的 IP 地址），pod 的生命周期也和   Infra 一致，与用户容器无关

### Pod 的生命周期

一个 Pod 的创建过程如下图所示

![./img/Untitled%2005.png](./img/Untitled%2005.png)

- init Container

    init C 在 Pod 启动时，在主容器启动前执行，做初始化工作

    初始化容器，可以有一个或多个，如果有多个则按照定义的顺序依次执行，并且前一个必需执行成功后一个才可以执行，所有的初始化容器执行成功后，普通应用容器才可以执行

    当初始化容器执行失败时，如果 `restart policy` 是 `OnFailure` 或者 `Always`，那么会重复执行失败的初始化容器一直到成功，如果 `restart policy` 是 `Never`，则不会重启失败的初始化容器

    如果初始化容器执行成功，那么无论 `restart policy` 是什么，也不会再次被重启

    只有所有的执行完后，主容器才启动，所以在它没有运行完成之前 pod 一定处于未就绪状态

    由于一个 Pod 里的存储卷是共享的，所以 Init C 里产生的数据可以被主容器使用

- Container hook
    1. Post start hook

        容器启动后钩子，该钩子在容器被创建后立刻触发，通知容器它已经被创建

        如果该钩子对应的 hook handler 执行失败，则该容器会被杀死，并根据该容器的重启策略决定是否要重启该容器

        这个钩子不需要传递任何参数

    2. Pre stop hook

        容器结束前钩子，该钩子在容器被删除前触发，其所对应的 hook handler 必须在删除该容器的请求发送给 Docker daemon 之前完成

        在该钩子对应的 hook handler 完成后不论执行的结果如何，Docker daemon 会发送一个 SGTERN 信号量给 Docker daemon 来删除该容器

        这个钩子不需要传递任何参数

- Container probe
    1. Liveness probe

        存活性探测，指示容器是否正在运行

        如果存活性探测失败，则 kubelet 会杀死容器，并且容器将受到其重启策略的影响

        如果容器不提供存活探针，则默认状态为 Success

    2. Readiness probe

        就绪性探测，指示容器是否准备好服务请求

        如果就绪性探测失败，端点控制器将从与 Pod 匹配的所有 Service 的端点中删除该 Pod 的 IP 地址

        初始延迟之前的就绪状态默认为 Failure，如果容器不提供就绪探针，则默认状态为 Success

### Pod 的阶段

Pod 的阶段（Phase）是 Pod 在其生命周期中所处位置的简单宏观概述

 该阶段并不是对容器或 Pod 状态的综合汇总，也不是为了成为完整的状态机

- Running

    Pod 已经被绑定到了一个节点，所有容器已被创建

    至少一个容器正在运行，或者正在启动或重新启动

- Successed

    所有容器成功终止，也不会重启

- Failed

    Pod 中的所有容器都已终止，并且至少有一个容器是因为失败终止

    也就是说，容器以非 0 状态退出或者被系统终止

- Unknown

    由于一些原因，Pod 的状态无法获取，通常是与 Pod 通信时出错导致的

### Pod 初始化

第一次启动 Pod 时，Kubelet 会通过 `Remote Procedure Command` 协议调用 RunPodSandbox

`sandbox` 用于描述一组容器，例如在 Kubernetes 中它表示的是 Pod

`sandbox` 是一个很宽泛的概念，所以对于其他没有使用容器的运行时仍然是有意义的（比如在一个基于 `hypervisor` 的运行时中，sandbox 可能指的就是虚拟机）

我们的例子中使用的容器运行时是 Docker，创建 sandbox 时首先创建的是 `pause` 容器

pause 容器作为同一个 Pod 中所有其他容器的基础容器，它为 Pod 中的每个业务容器提供了大量的 Pod 级别资源，这些资源都是 Linux 命名空间（包括网络命名空间，IPC 命名空间和 PID 命名空间）

pause 容器提供了一种方法来管理所有这些命名空间并允许业务容器共享它们，在同一个网络命名空间中的好处是：同一个 Pod 中的容器可以使用 `localhost` 来相互通信

pause 容器的第二个功能与 PID 命名空间的工作方式相关，在 PID 命名空间中，进程之间形成一个树状结构，一旦某个子进程由于父进程的错误而变成了“孤儿进程”，其便会被 `init` 进程进行收养并最终回收资源

一旦创建好了 pause 容器，下面就会开始检查磁盘状态然后开始启动业务容器

### 容器状态

一旦调度器将 Pod 分派给某个节点，kubelet 就通过 容器运行时 开始为 Pod 创建容器

容器的状态有三种：Waiting（等待）、Running（运行中）和 Terminated（已终止）。

要检查 Pod 中容器的状态，你可以使用 `kubectl describe pod <pod_name>`

每种状态都有特定的含义：

- Waiting
如果容器并不处在 Running 或 Terminated 状态之一，它就处在 Waiting 状态

    处于 Waiting 状态的容器仍在运行它完成启动所需要的操作：例如，从某个容器镜像 仓库拉取容器镜像，或者向容器应用 Secret 数据等等

- Running
Running 状态表明容器正在执行状态并且没有问题发生

    如果配置了 postStart 回调，那么该回调已经执行且已完成

- Terminated
处于 Terminated 状态的容器已经开始执行并且或者正常结束或者因为某些原因失败

---

# 控制器 Controller

---

# Kubernetes 的存储

## Volume

默认情况下容器的数据都是非持久化的，在容器消亡以后数据也跟着丢失，所以 Docker 提供了 Volume 机制以便将数据持久化存储

类似的，Kubernetes 提供了更强大的 Volume 机制和丰富的插件，解决了容器数据持久化和容器间共享数据的问题

与 Docker 不同，Kubernetes Volume 的生命周期与 Pod 绑定

- 容器挂掉后 Kubelet 再次重启容器时，Volume 的数据依然还在
- 而 Pod 删除时，Volume 才会清理。数据是否丢失取决于具体的 Volume 类型，比如 emptyDir 的数据会丢失，而 PV 的数据则不会丢

## Persistent volumes

既然已经有了 Pod Volumes，为什么又要引入 PV 呢？

我们知道 pod 中声明的 volume 生命周期与 pod 是相同的，以下有几种常见的场景：

- 场景一

    pod 重建销毁，如用 Deployment 管理的 pod，在做镜像升级的过程中，会产生新的 pod并且删除旧的 pod ，那新旧 pod 之间如何复用数据？

- 场景二

    宿主机宕机的时候，要把上面的 pod 迁移，这个时候 StatefulSet 管理的 pod，其实已经实现了带卷迁移的语义

- 场景三

    多个 pod 之间，如果想要共享数据，应该如何去声明呢？

    我们知道，同一个 pod 中多个容器想共享数据，可以借助 Pod Volumes 来解决

    当多个 pod 想共享数据时，Pod Volumes 就很难去表达这种语义

- 场景四

    如果要想对数据卷做一些功能扩展性，如：snapshot、resize 这些功能，又应该如何去做呢

以上场景中，通过 Pod Volumes 很难准确地表达它的复用/共享语义，对它的扩展也比较困难

因此 K8s 中又引入了 Persistent Volumes ****概念，它可以将存储和计算分离，通过不同的组件来管理存储资源和计算资源，然后解耦 pod 和 Volume 之间生命周期的关联

这样，当把 pod 删除之后，它使用的PV仍然存在，还可以被新建的 pod 复用

## Persistent volumes claim

用户通过 PVC 使用 PV，来简化 k8s 用户对存储的使用方式，做到职责分离

通常用户在使用存储的时候，只用声明所需的存储大小以及访问模式

访问模式是什么？其实就是：我要使用的存储是可以被多个 node 共享还是只能单 node 独占访问（注意是 node level 而不是 pod level）？只读还是读写访问？用户只用关心这些东西，与存储相关的实现细节是不需要关心的

通过 PVC 和 PV 的概念，将用户需求和实现细节解耦开，用户只用通过 PVC 声明自己的存储需求

PV 是有集群管理员和存储相关团队来统一运维和管控，这样的话，就简化了用户使用存储的方式

可以看到，PV 和 PVC 的设计其实有点像面向对象的接口与实现的关系，用户在使用功能时，只需关心用户接口，不需关心它内部复杂的实现细节

## Static Volume Provisioning

静态分配，由集群管理员事先去规划这个集群中的用户会怎样使用存储，它会先预分配一些存储，也就是预先创建一些 PV；然后用户在提交自己的存储需求（也就是 PVC）的时候，K8s 内部相关组件会帮助它把 PVC 和 PV 做绑定；之后用户再通过 pod 去使用存储的时候，就可以通过 PVC 找到相应的 PV，它就可以使用了

不足，由于需要集群管理员预分配，所以很难预测用户真实需求的。举一个最简单的例子：如果用户需要的是 20G，然而集群管理员在分配的时候可能有 80G 、100G 的，但没有 20G 的，这样就很难满足用户的真实需求，也会造成资源浪费

---

# Kubernetes 的网络

## 单机模式下的容器网络

为了实现容器间的网络通信，需要两个东西，一个是网桥 docker0（Docker 会默认在宿主机创建），另一个是虚拟设备 Veth Pair

Veth Pair 以两张虚拟网卡的形式出现，在 Docker 默认的网络模式 Bridge 下，容器中会存在 Veth Pair 的其中一张网卡 eth0，而在宿主机的 docker0 上会存在另一张，在 docker0 上的这张网卡会将通过他接受到的数据全部转发给 docker0

如果此时在宿主机上有两个 Bridge 模式的容器记作 A 和 B，他们之间的网络时互通的，比如在 A 中 ping B，那么 A 的请求会通过容器中的 eth0 转发到宿主机中另一端 Veth Pair 虚拟网卡所在的 docker0 上，然后会根据数据的 MAC 地址到 docker0 中容器 B 在宿主机 Veth Pair 的一端虚拟网卡转发到 Veth Pair 的另一端虚拟网卡，也就是 B 中的 eth0，B 中处理请求再返回 A 中就完成了这一次通信

![./img/Untitled%2006.png](./img/Untitled%2006.png)

## CNI 网络插件

TODO

## Flannel

Flannel 是用于解决容器“跨主通信”（容器跨主机通信）问题的

Flannel 有三种后端实现，分别是 UDP、VXLAN 和 host-gw

### UDP

UDP 模式是最直接、最容易理解也是性能最差的实现

假设现在有两个主机 Node1 和 Node2，Node1 中的容器 container-1（记作 A）想和 Node2 中的容器 container-2（记作 B）通信，即从 100.96.1.2 到 100.96.2.3 的通信

首先 IP 包通过容器 A 的网关到宿主机的 docker0，但是 docker0 不能匹配到目的地址，所以转向下一条路由规则到达 flannel0（Tunnel 设备，用于在操作系统内核和用户应用程序之间传递 IP 包），然后 flannel0 会把这个 IP 包转给宿主机的 flanneld 进程，flanneld 进程会根据目的 IP 匹配的“子网”（一台宿主机上的容器属于同一子网）去 etcd 中查询目的 IP 对应的宿主机 IP，然后就是宿主机之间的通信了

这种通信方式要求 docker0 网桥的地址范围必须是 Flannel 为宿主机分配的子网，只需要在启动 dockerd 的时候添加参数 `-bip=$FLANNEL_SUBNET` 即可；另外需要 flanneld 进程常驻后台，以解析宿主机发送或者接受到的 UDP 包

![./img/Untitled%2013.png](./img/Untitled%2013.png)

而这种方式性能差的原因就在于使用了 flannel0 这个 TUN 设备，需要在用户态和内核态中进行三次数据传递（简单理解为用户进程和系统内核的交互），而这个切换的消耗是比较高的

![./img/Untitled%2007.png](./img/Untitled%2007.png)

### VXLAN

VXLAN 使用和 UDP 中 flanneld 功能类似的 VTEP 设备，来完成目的地址的包装和解包，但是 VTEP 是在内核中进行操作的，所以性能比 UDP 要高

比如 10.1.15.2 要向 10.1.16.3 发送 IP 包，那么 IP 包会先经过 docker0 转发给 flannel.1，flannel.1 会根据目的地址去寻找目的 VTEP 设备 MAC 地址，进行第一次封装（得到内部数据帧，... VTEP MAC | DEST CONTAINER IP），再根据 VTEP 的 MAC 地址查询对应主机的 IP 地址，进行第二次封装（得到外部数据帧，即目的容器的宿主机信息，DEST MAC | DEST IP ...），然后只要将这个封装好的数据帧（VTEP MAC | DEST CONTAINER IP | DEST MAC | DEST IP）通过宿主机的 eth0 发送出去到目的容器的宿主机，进行相应的解包就完成了一次通信

在这种模式下 flanneld 会维护一份路由规则，在新节点加入 Flannel 网络时在所有节点上添加路由规则，描述目标网段（10.1.16.0/24）、经由网关（10.1.16.0）和 使用的设备（flannel.1），可以使用 `route -n` 查看相应的路由规则

flanneld 会维护 Flannel 网络中 IP 和 VTEP 设备 MAC 地址的对应关系，即 flannel.1 第一次封装查询的数据，可以使用 `ip neigh show dev flannel.1` 查看这个对应关系

flanneld 还会维护 VTEP 设备 MAC 地址 和 其宿主机的对应关系，即 flannel.1 第二次封装查询的数据，可以使用 `bridge fdb show flannel.1` 查看这个对应关系

![./img/Untitled%2008.png](./img/Untitled%2008.png)

### host-gw

host-gw 模式通过 flanneld 维护路由表信息，将每个 Flannel 子网的下一个转发地址设置为该子网对应宿主机的 IP 地址

比如 Infra-container-1 和 Infra-container-2 通信，请求通过 cni0 网桥（功能和 docker0 类似），查询路由表根据目的 IP（10.244.1.3）对应的子网找到通往下一站需要使用的网卡和 IP 地址（该子网对应的宿主机 IP 地址），然后通过这个网卡 eth0 到达 Infra-container-2 的宿主机 10.168.0.3，再次查询路由表，同样根据目的 IP 找到网桥 cni0，通过网桥到达目的容器，完成通信

flanneld 需要通过 API Server 来 Watch etcd 中维护 Flannel 子网和主机的信息，并实时更新路由表

![./img/Untitled%2009.png](./img/Untitled%2009.png)

## Calico

Calico 提供的网络解决方案，与 Flannel 的 host-gw 模式类似，也是通过在路由表中添加目的网段使用的网桥设备以及目的网段对应宿主机 IP 的一组规则，不同之处在于路由信息的维护方式和 Calico 不在宿主机创建网桥设备

Calico 项目的架构由三个部分组成

1. Calico 的 CNI 插件

    Calico 与 Kubernetes 对接的部分

    由于 Calico 没有使用 CNI 的网桥模式，Calico 的 CNI 插件还需要在宿主机上为每个容器的 Veth Pair 设备配置一条路由规则，用于接收传入的 IP 包

2. Felix

    它是一个 DaemonSet，负责在宿主机上插入路由规则（即：写入 Linux 内核的 FIB转发信息库），以及维护 Calico 所需的网络设备等工作

3. BIRD

    它就是 BGP 的客户端，专门负责在集群里分发路由规则信息

### BGP

在宿主机可以相互通信的情况下使用该模式，暂且称之为 BGP 模式；该模式下 CNI 插件会为容器配置 Veth Pair 设备，一端在容器一端在宿主机，相关信息通过路由表维护

假如 container 1 要和 container 4 进行通信，请求通过 Veth Pair 由容器到达宿主机，根据目的容器 IP 去路由表查找需要使用的网络设备和目的容器宿主机 IP，通过 eth0 转发到目的容器宿主机，再次通过路由表查询下一个网络设备 calib5863f3 转发到目的容器，完成通信

![./img/Untitled%2010.png](./img/Untitled%2010.png)

### IPIP

如果宿主机不能够相互通信，则需要为宿主机维护和其他宿主机之间的路由规则，即启用 IPIP 模式

![./img/Untitled%2011.png](./img/Untitled%2011.png)

---

# 服务发现 Service

## Service 简介

Kubernetes 之所以需要 Service 有以下两方面原因

1. Pod 的 IP 不是固定的
Pod 生命周期是短暂的；在 Pod 的生命周期过程中，比如它创建或销毁，它的 IP 地址都会发生变化，这样就不能使用传统的部署方式，不能指定 IP 去访问指定的应用
2. 一组 Pod 实例之间存在负载均衡的需求

    这些 Pod 组需要提供一个统一的访问入口，以及控制流量负载均衡到这个组里面

## Service 声明

一个 Service 可以使用如下格式进行声明，其中比较重要的信息有三部分，分别是 Service 的元信息 `metadata`、Pod 选择器 `spec.selector` 和 协议及端口配置 `spec.ports`

```yaml
apiVersion: v1
kind: Service
metadata:
 name: hostnames
spec:
 selector:
   app: hostnames
 ports:
 - name: default
   protocol: TCP
   port: 80
   targetPort: 9376
```

在上面的这个 Service 的资源清单中，便使用了 hostnames 这个 Service 的 `80` 端口代理了 `app=hostnames` 的 Pod 的 `9376` 端口

### Endpoint

被 `spec.selector` 选中的 Pod 就称为这个 Service 的 Endpoint

可以使用 `kubectl discribe service <service_name>` 或者 `kubectl get endpoints <service_name>` 进行查看

需要注意的是，只有处于 Running 状态，且 readinessProbe 检查通过的 Pod，才会出现在Service 的 Endpoints 列表里

并且，当某一个 Pod 出现问题时，Kubernetes 会自动把它从 Service 里摘除掉

## Service 工作原理

与其说是 Service 工作原理，不如说是 kube-proxy 的工作原理，因为 Service 的功能是由 kube-proxy 实现的，kube-proxy 共有三种模式 userspace、iptables、ipvs，这里简单介绍后两种

### iptables mode

在这种模式下，kube-proxy 的主要工作是监听 Pod 的变化并在宿主机生成并维护一组 iptables 规则，以创建一个 Service 为例

- 将创建 Service 的请求提交给 Kubernetes 之后，kube-proxy 就会在 iptables 中添加规则，将到这个 Service 的请求，也就是到这个 Service 对应 vip 的这个 Service 暴露的端口的请求，转发到另外一条 iptables 链
- 这条 iptables 链是一组规则的集合，其中每条规则都是一个 DNAT 规则，每条规则对应这个 Service 的 Endpoint 中的一个被代理 Pod；DNAT 的作用就是在路由之前修改 IP 包的目的地址和端口，也就是修改为被代理 Pod 的 IP 和 port

这样访问 Service vip 的请求就转化为了一个访问对应 Pod 的请求

可以看出这种模式需要在宿主机上设置相当多的 iptables 规则，并且 kube-proxy 还需要在控制循环里不断地刷新这些规则来确保它们始终是正确的

所以在宿主机上有大量 Pod 的时候，成百上千条 iptables 规则不断地被刷新，会大量占用该宿主机的 CPU 资源，所以基于 iptables 的 Service 是制约 Kubernetes 项目承载更多量级的 Pod 的主要障碍

### IPVS mode

IPVS 模式的工作原理，其实跟 iptables 模式类似，不过它把对 “规则” 的处理放到了内核态，从而降低了维护这些规则的代价，也提高了性能

- 在创建 Service 之后，kube-proxy 会在宿主机上创建一个虚拟网卡，并为它分配 Service vip 作为 IP 地址
- 然后 kube-proxy 就会通过 Linux 的 IPVS 模块，为这个 IP 地址设置三个 IPVS 虚拟主机，并设置这三个虚拟主机之间使用轮询模式来作为负载均衡策略

不过，IPVS 模块只负责上述的负载均衡和代理功能，而一个完整的 Service 流程正常工作所需要的包过滤、SNAT 等操作，还是要靠 iptables 来实现，但是这些辅助性的iptables 规则数量有限，也不会随着 Pod 数量的增加而增加

---

# 资源调度

## 默认调度器

### 职责

默认调度器的主要职责就是为新创建出的 Pod 寻找合适的节点

选择合适节点这一过程包括两步，一是从节点中挑选出可以运行该 Pod 的节点（Predicates），二是从这些节点中选出最适合运行的节点（Priorities）

### 工作原理

调度过程大致可以分为两个部分，两个相互独立的控制循环

第一个控制循环，Informer Path

用于监听 etcd 中 Kubernetes 对象的变化，将待调度对象加入队列，并更新调度器缓存；比如一个待调度 Pod 被创建出来后，Pod Informer 的 Handler 会把这个 Pod 加入待调度的队列，并更新缓存（在调度器中通过缓存查询数据，不访问 API Server ）

对于这种将集群信息尽量保存到缓存中，即 **Cache 化**，并使用缓存中的数据进行处理 ，目的在于提高 Predicated 和 Priority 的效率

第二个控制循环，Scheduling Path

实际负责调度 Pod 的主循环，从待调度队列中列出 Pod ，通过 Predicates 过滤出一组节点，再通过 Priorities 对节点从 0~10 进行打分，以得分最高的为本次调度结果，更新缓存中 Pod 和 Node 的信息完成绑定（Bind，将 Pod 对象的 nodeName 字段的值，修改为该节点），然后调度器才会真正地向 API Server 发请求，进行真正的 Bind，更新 etcd 中的数据

对于只更新缓存中的信息（“虚假的“ API Server 信息）的操作称为 **Assume**；由于这种设计，Pod 在真正运行之前还进行二次验证 Admit 来确保该 Pod 确实能运行在该节点上

保证调度高性能除去 Cache 化 和 Assume 还有另外一个设计，就是 **无锁化**，在对节点进行过滤和打分时，会使用类似 MapReduce 的方式，以节点为粒度，启动多个 Goroutine 进行计算，最后进行汇总

![./img/Untitled%2012.png](./img/Untitled%2012.png)

## 调度策略

### Predicates

在该步骤中，会根据调度策略，对集群中的节点进行过滤，选出一系列可以运行待调度 Pod 的节点

Predicates 一共有以下几种默认的调度策略

- GeneralPredicates

    基础掉调度策略，比如用于检查 CPU、Memory 是否够用，Port 是否冲突等基本的过滤条件，也是 Scheduling Path 中用于二次验证 Admit 调用的确认规则；该策略包含以下等检查项：

    PodFitsResources，检查 Pod 的 `resources.requests` 字段，CPU、Memory 是否够用

    PodFitsHost，检查宿主机的名字是否跟 Pod 的 spec.nodeName 一致

    PodMatchNodeSelector，检查 Pod 的 nodeSelector 或者 nodeAffinity 指定的节点，是否与待考察节点匹配

- VolumePredicates

    与容器持久化 Volume 相关的调度策略；包含以下等检查项：

    NoDiskConflict，检查多个 Pod 声明挂载的持久化 Volume 是否有冲突

    MaxPDVolumeCountPredicate， 检查一个节点上某种类型的持久化 Volume是不是已经超过了一定数目

    VolumeBindingPredicate，检查该 Pod 对应的 PV 的 nodeAffinity 字段，是否跟某个节点的标签相匹配

- NodePredicates

    考察待调度 Pod 是否满足 Node 本身的某些条件；该策略包含以下等检查项：

    PodToleratesNodeTaints，检查 Node 的 “污点” 机制；只有当 Pod 的 Toleration 字段与 Node 的 Taint 字段能够匹配的时候，这个 Pod 才能被调度到该节点上

    NodeMemoryPressurePredicate，检查当前节点的内存是不是已经不够充足，如果不充足那么待调度 Pod 就不能被调度到该节点上

- PodPredicates

    这一组规则，跟 GeneralPredicates 大多数是重合的，比较特别的是 PodAffinityPredicate，检查 Pod 和 Node 之间的亲密关系和反亲密关系，即 affinity 和 anti-affinity

    即检查 Pod 的 `affinity` 字段，可以在该字段下添加子项 `topologyKey` 指定亲密关系的作用域

### Priority

该步骤用于给 Predicates 筛选出的 Node 进行打分，有以下一些规则

- LeastRequestedPriority

    常用的规则，来选择 CPU 和 Memory 空闲资源最多的 Node

- BalancedResourceAllocation

    计算 Pod 请求的资源 / 节点上的可用资源，理解为资源间的 “距离”，选择资源距离最近的 Node，目的在于选出使 Node 资源分配均衡

- ImageLocalityPriority

    如果待调度 Pod 需要使用的镜像很大，并且已经存在于某些 Node上，那么这些 Node 的得分就会比较高

## 优先级和抢占机制

### PriorityClass

PriorityClass 是一种资源类型，可以通过类似如下的 yaml 进行定义

```yaml
kind: PriorityClass
metadata:
 name: high-priority
value: 1000000
globalDefault: false
description: "This priority class should be used for high priority service pods only."
```

上面的 yaml 声明了一个叫做 `high-priority` 优先级为 `1000000` 的 PriorityClass；需要注意字段 `globalDefault` 用于标志该值是否为默认的优先级，没有声明 PriorityClass 的 Pod 优先级为 0

声明完 PriorityClass 后，只需要在 Pod 中添加 `spec.priorityClassName` 就可以为该 Pod 设置优先级

优先级主要体现在调度的 Scheduling Path 阶段，优先级高的 Pod 会比优先级低的 Pod 提前出队

### 抢占机制

调度器的抢占能力主要体现在高优先级 Pod（记作 抢占者 Preemptor）调度失败的时候，这个时候调度器会找一个或多个低优先级 Pod（记作 牺牲者 Victims），在该 Pod 被删除后，将这个高优先级的 Pod 调度到该节点上

需要注意的是 “抢占” 并不会立刻发生，而是把抢占者留到下一个调度周期处理；调度器会把抢占者的 `spec.nominatedNodeName` 修改为牺牲者所在节点的名称，等到下一个调度周期决定是否真的要运行在该节点上

如果在抢占者等待调度过程中出现了优先级更高的抢占者，则会清空优先级较低的那个抢占者的 `spec.nominatedNodeName`，让优先级高的抢占者抢占，优先级低的抢占者抢占其他节点

---

# 参考资料

- [Kubernetes Handbook——Kubernetes 中文指南/云原生应用架构实践手册](https://jimmysong.io/kubernetes-handbook/)
- [CNCF x Alibaba 云原生技术公开课 - 云原生教程](https://edu.aliyun.com/roadmap/cloudnative)
- [深入剖析Kubernetes - Kubernetes原来可以如此简单](https://time.geekbang.org/column/intro/116)
- [iptables详解（1）：iptables概念](https://www.zsythink.net/archives/1199)