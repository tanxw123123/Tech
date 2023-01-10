## 1.k8s组件

`i. master节点组件：`

- *API Server*：提供Kubernetes API接口，主要处理 Rest操作以及更新Etcd中的对象。是所有资源增删改查的唯一入口。
- *Scheduler*：资源调度，绑定Pod到Node上；
- *Controller*: 负责运行控制器进程。负责维护群集的状态，比如故障检测、自动扩展、滚动更新等；
  - `deployment`：管理pod多副本
  - `replicaSet`：pod的多副本，创建deployment会自动创建replicaSet，通常都用deployment
  - `daemonSet`：每个节点运行一个pod
  - `statefuleSet`：保证pod在整个生命周期中名称不变，其他conntroller在pod发生故障需要删除并重启时，pod的名称会发生改变。
  - `Job`：运行结束就删除的应用，其他controller中的pod会长期持续的运行。
- *etcd*：Kubernetes 提供的一个高可用的键值数据库，用于保存集群所有的网络配置和资源对象的状态信息，也就是保存了整个集群的状态。



`ii. node节点组件：`

- *kubelet*：负责pod的生命周期管理，维护和管理该Node上面的所有容器；
- *kube-proxy*：是实现service的通信与负载均衡机制的重要组件，将到service的请求转发到后端的pod上；
- *docker*：是负责容器的创建和管理工作；
- *pod*：k8s的最小工作单元，一个pod包含一个或多个容器。pod作为整体被master调度到某个node上运行，pod中所有容器使用相同的ip和端口。



##  2.pod健康检查

`1.livenessProbe探针（存活检查）`

根据用户自定义规则来判定pod是否健康，kubelet会根据其重启策略来决定是否重启

**pod重启策略：**

- Always: 当容器终止退出后，总是重启容器，默认策略。

- onFailure: 当容器异常退出（退出状态码非0）时，才重启容器
- Never：当容器终止退出，从不重启容器

`2.ReadinessProbe探针（就绪检查）`

根据用户自定义规则来判断pod是否健康，如果探测失败，控制器会将此pod从对应service的endpoint列表中移除，从此不再将任何请求调度到此Pod上，直到下次探测成功。



**上面两种探针都支持3种探测方法：**

- Exec： 通过执行命令的方式来检查服务是否正常
- Httpget： 通过发送http/htps请求检查服务是否正常，返回的状态码为200-399则表示容器健康
- tcpSocket： 通过容器的IP和Port执行TCP检查，如果能够建立TCP连接，则表明容器健康，这种方式与HTTPget的探测机制有些类似，tcpsocket健康检查适用于TCP业务。



## 3.pod调度方式

```
https://cloud.tencent.com/developer/article/1644857

Pod调度策略除了系统默认的kube-scheduler调度器外还有以下几种实现方式：

1.nodeName（直接指定Node主机名）

2.nodeSelector （节点选择器，为Node打上标签，然后Pod中通过nodeSelector选择打上标签的Node）

3.污点taint与容忍度tolerations

4.NodeAffinity 节点亲和性

5.PodAffinity Pod亲和性

6.PodAntAffinity Pod反亲和性
```



```shell
一、 kube-scheduler调度介绍
默认调度器。
满足一个Pod调度请求的所有Node称之为可调度节点
如果没有任何一个Node能满足Pod的资源请求，那么这个Pod将一直停留在未调度状态直到调度器能够找到合适的Node

两个步骤：
1.过滤阶段： 过滤出可调度节点
2.打分阶段： 对可调度节点进行打分

二、Pod调度之nodeName
nodeNamed这种调度方式比较简单，我们可以指定Pod在哪台Node上进行运行，通过spec.nodeName参数来指定Node主机名称即可。

spec:
#指定该Pod运行在k8s-node02节点上
  nodeName: k8s-node02
  
三、Pod调度之nodeSelector
nodeSelector用于将Pod调度到匹配Label的Node上,所以要先给node打上标签


https://www.jianshu.com/p/2d59e83162a7  给节点打标签
- 添加标签
- 查看标签
spec:
  nodeSelector:                      #指定标签选择器
    team: development                #label指定开发团队的label
    

四、Pod调度之污点与容忍
污点（taint）： 定义在Node之上的键值型的数据。用于让节点拒绝将Pod调度运行于其上，除非该Pod对象具有接纳Node污点的容忍度
容忍度（tolerations）： 定义在Pod对象的键值型属性数据，用于·配置其可容忍的Node污点，而且调度器仅能将Pod对象调度至其能够容忍该Node污点的Node之上。

污点类型:
• NoSchedule ：为Node添加污点等级为NoSchedule,除容忍此污点的Pod以外的其它Pod将不再被调度到本机。
•PreferNoSchedule：为Node添加污点等级为PreferNoSchedule,不能容忍此污点的Pod对象尽量不要调度到当前节点，如果没有其它节点可以供Pod选择时，也会接受没有容忍此污点的Pod对象。
•NoExecute：为Node添加污点等级为NoExecute，能容忍此污点的Pod将被调度到此节点，而且节点上现存的Pod对象因节点使用了NoExceute等级的污点，则现存的Pod将被驱赶至其它满足条件的Node

# 添加污点： 为k8s-node02添加污点，污点程度为NoSchedule，type=calculate为标签
$ kubectl taint node k8s-node02 type=calculate:NoSchedule

# 查看污点：
$ kubectl describe nodes k8s-node02 | grep Taints

# 删除污点：
$ kubectl taint node k8s-node02 type:NoSchedule-


容忍度介绍及定义

```



## 4.k8s资源限制

```yaml
https://juejin.cn/post/6974203884369608734
requests：相对限制，是容器的最低申请资源，这个限制是相对的，无法做到绝对严格。
limits：绝对限制，这个是限制的绝对的，不可能超越。


resources:
      limits:
        cpu: "2"
        memory: 1000Mi
      requests:
        cpu: "1"
        memory: 500Mi
        
```



## 5.k8s node节点停机维护，pod如何迁移？

```shell
# 1. 默认迁移
当node节点关机后，k8s集群并没有立刻发生任何自动迁移动作，如果该node节点上的副本数为1，则会出现服务中断的情况。其实事实并非如此，k8s在等待5分钟后，会自动将停机node节点上的pod自动迁移到其他node节点上。

模拟node节点停机，停止kubelet
$ systemctl stop kubelet

$ kubectl get node
... NotReady

监控pod状态，大约5分钟
$ kubectl get pod -n test -o wide -n test -w

5分钟后，pod终止并重建
从以上过程看出，停机node节点上的pod在5分钟后先终止再重建，直到pod在新节点启动并由readiness探针检测正常后并处于1\1 Running状态才可以正式对外提供服务。因此服务中断时间=停机等待5分钟时间+重建时间+服务启动时间+readiness探针检测正常时间

kubelet停止后，node节点自动添加了污点（Taints）
$ kubectl describe node k8s-3-218

可以看到，此时pod的Tolerations 默认对于具有相应Taint的node节点容忍时间为300s

# 2. 手动迁移
为避免等待默认的5分钟，我们还可以使用cordon、drain、uncordor三个命令实现节点的主动维护。此时需要用到以下三个命令：

cordon：标记节点不可调度，后续新的pod不会被调度到此节点，但是该节点上的pod可以正常对外服务；
drain：驱逐节点上的pod至其他可调度节点；
uncordon：标记节点可调度；

a. 标记节点不可调度：
$ kubectl cordon k8s-3-219
$ kubectl get node

b. 驱逐pod
$ kubectl drain k8s-3-219 --delete-local-data --ignore-daemonsets --force

此时与默认迁移不同的是，pod会先重建再终止，此时的服务中断时间=重建时间+服务启动时间+readiness探针检测正常时间，必须等到1/1 Running服务才会正常

# 3. 如何能够做到平滑迁移呢
论是默认迁移和手动迁移，都会导致服务中断，而pdb能可以实现节点维护期间不低于一定数量的pod正常运行，从而保证服务的可用性
要做到平滑迁移就需要用的pdb(PodDisruptionBudget)，即主动驱逐保护

从218驱逐到219
1. 标记节点不可调度
2. 新建pdb

此时由于只有一个副本，最小可用为1，则allow就为0，因此在一个副本通过pdb是不会发生驱逐的，我们需要先扩容，将副本数调整为大于1.
调整副本数replicas为2
$ kubectl edit deploy helloworld -n test
再次查看pdb
$ kubectl get pdb
ALLOWED为1

3. 驱逐
4. 维护完毕，将node调整为可调度
$ kubectl uncordon k8s-3-218


```



## 6.k8s监控方案

https://cloud.tencent.com/developer/article/2047144



## 7.pod间通信

https://www.51cto.com/article/702401.html  网络模型

### 1.Docker容器间通信

我们安装Docker时，它会自动创建三个网络，bridge（创建容器默认连接到此网络）、 none 、host；[网址](https://cloud.tencent.com/developer/article/1674259)

```shell
- host：容器将不会虚拟出自己的网卡，配置自己的IP等，而是使用宿主机的IP和端口。相当于Vmware中的桥接模式
- None：该模式关闭了容器的网络功能，相当于一个回环网络。
- Bridge：此模式会为每一个容器分配、设置IP等，并将容器连接到一个叫docker0的虚拟网桥，通过docker0网桥以及Iptables nat表配置与宿主机通信。相当于Vmware中的NAT模式

- overlay：顾名思义：覆盖，但它又不是覆盖，它的作用就是在容器原有的网络基础之上，再添加一块网卡，并为其分配一个IP地址，可以将所有的docker容器关联到同一个局域网中，适用于容器与容器是跨主机进行通信的场景。
```

```shell
$ docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
35ac75169514   bridge    bridge    local
20310e4d27d0   host      host      local
7376d0c6efe5   none      null      local
```



**一、Bridge模式**

启动docker--》创建docker0虚拟网桥（类似交换机）

容器连接到docker0虚拟网桥--》然后，分配ip给容器



**单机环境下的网络拓扑如下（主机地址是10.10.0.186/24）：**

![image-20230109192801436](C:\Users\ronnie\AppData\Roaming\Typora\typora-user-images\image-20230109192801436.png)



```shell
实现的效果如下：
 
1. 基于docker0网络创建2个容器，分别是test1、test2。
2. 创建自定义网络，网络类型为bridge，名称为my_net1.基于此网络创建两个容器test3,test4（不指定网段）
3. 创建自定义网络，网络类型为bridge，名称为my_net2，指定网段为172.20.18.0/24，基于此网络创建两个容器test5(ip为172.20.18.5)，test6（IP为172.20.18.6）。

配置实现test2能够和test3相互通信，test4和test5可以相互通信。
```

配置如下：

```shell
$ docker run -itd --name test1 busybox  #创建一个容器test1
$ docker run -itd --name test2 busybox  # 同上

$ docker network create -d bridge my_net1             #创建一个桥接网络，名称为my_net1
$ docker run -itd --name test3 --network my_net1 busybox       #基于my_net1创建容器test3
$ docker run -itd --name test4 --network my_net1 busybox       # 同上

$ docker network create -d bridge --subnet 172.20.18.0/24 my_net2    # 创建一个桥接网络my_net2，并指定其网段
$ docker run -itd --name test5 --network my_net2 --ip 172.20.18.5 busybox    #基于my_net2网络，创建一个容器test5，并且指定其IP地址
$ docker run -itd --name test6 --network my_net2 --ip 172.20.18.6 busybox    # 同上

$ docker network connect my_net1 test2          #将test2连接到my_net1这个网络，相当于将test2这个容器添加了一块网卡，然后连接到了my_net1这个交换机 
$ docker exec test2 ping test3           #进行ping测试，可以发现test2 可以ping通test3了。
$ docker exec test2 ip a s            # 查看此时容器ip

$ docker network connect my_net2 test4    # 将test4连接到my_net2网络
$ docker exec test4 ping test5 
```

---



### 2.Docker容器跨主机通信

1.直接路由方式

```
https://cloud.tencent.com/developer/article/1587094
```



## 7.镜像下载策略

主要分为三种：

- Always：总是从指定的仓库中获取镜像。
- Never：使用本地镜像，不从仓库中下载。
- IfNotPresent：当本地镜像不存在时，才从仓库拉取。

## 8.pod状态

Pending：pod 正在等待 kube-scheduler 选择合适的节点创建。

Running：pod 已正常创建，并且至少有一个容器正在运行。

Succeeded：所有容器已成功启动运行。

Failed：pod 的容器非正常退出。

Unknown：无法获取 pod 状态，可能节点间通信出现问题。

## 9.创建pod流程

1） 客户端提交 Pod 的配置信息（可以是 yaml 文件定义好的信息）到 kube-apiserver；
 2） Apiserver 收到指令后，通知给 controller-manager 创建一个资源对象；
 3） Controller-manager 通过 api-server 将 pod 的配置信息存储到 ETCD 数据中心中；
 4） Kube-scheduler 检测到 pod 信息会开始调度预选，会先过滤掉不符合 Pod 资源配置要求的节点，然后开始调度调优，主要是挑选出更适合运行 pod 的节点，然后将 pod 的资源配置单发送到 node 节点上的 kubelet 组件上。
 5） Kubelet 根据 scheduler 发来的资源配置单运行 pod，运行成功后，将 pod 的运行信息返回给 scheduler，scheduler 将返回的 pod 运行状况的信息存储到 etcd 数据中心。

## 10.持久化

1）EmptyDir（空目录）：没有指定要挂载宿主机上的某个目录，直接由 Pod 内保部映射到宿主机上。类似于 docker 中的 manager volume。

2）Hostpath：将宿主机上已存在的目录或文件挂载到容器内部。类似于 docker 中的 bind mount 挂载方式。

3）PersistentVolume（简称 PV）： 基于 NFS 服务的 PV，也可以基于 GFS 的 PV。它的作用是统一数据持久化目录，方便管理。

